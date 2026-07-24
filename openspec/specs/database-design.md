# 数据库设计规范

> 适用技术栈：Spring Boot 3.4 + JPA + MySQL 8.0+ + Flyway
> 业务领域：AI 对话应用

---

- [1. 主键策略](#1-主键策略)
- [2. 公共字段](#2-公共字段)
- [3. NULL / NOT NULL 约束原则](#3-null--not-null-约束原则)
- [4. 索引策略](#4-索引策略)
- [5. Flyway 迁移规范](#5-flyway-迁移规范)
- [6. AI 对话核心表示例](#6-ai-对话核心表示例)

---

## 1. 主键策略

### 选型：Snowflake ID（雪花算法）

所有业务表主键统一使用 Snowflake ID，不采用自增 ID 或 UUID。

| 策略 | 结论 | 原因 |
|------|:----:|------|
| 自增 ID `AUTO_INCREMENT` | ❌ 不采用 | 暴露记录数、分库分表冲突、不安全 |
| UUID v4 `CHAR(36)` | ❌ 不采用 | 无序导致索引碎片、存储大(36B)、查询性能差 |
| Snowflake ID `BIGINT(19)` | ✅ 采用 | 全局唯一、趋势递增、有时间语义、防暴露 |

MySQL InnoDB 聚簇索引按主键顺序组织，Snowflake 趋势递增特性可最大限度减少页分裂。

### JPA 实现

```java
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "snowflake")
@GenericGenerator(name = "snowflake", type = SnowflakeIdGenerator.class)
private Long id;
```

> `SnowflakeIdGenerator` 需实现 `IdentifierGenerator` 接口。

### 例外规则

仅以下场景可用 `AUTO_INCREMENT`：
- 纯内部配置表、字典表、枚举表
- 生命周期短、明确不分表的临时表

**常规业务表（含关联表）一律用 Snowflake ID。**

---

## 2. 公共字段

### 字段清单

每张数据库表**必须包含**以下 5 个公共字段：

| 字段名 | MySQL 类型 | 约束 | JPA 注解 | 说明 |
|--------|-----------|------|----------|------|
| `id` | `BIGINT` | PK, NOT NULL | `@Id` | Snowflake ID 主键 |
| `created_at` | `DATETIME(3)` | NOT NULL, `DEFAULT CURRENT_TIMESTAMP(3)` | `@CreatedDate` | 创建时间（毫秒精度） |
| `updated_at` | `DATETIME(3)` | NOT NULL, `DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3)` | `@LastModifiedDate` | 最后更新时间 |
| `version` | `INT` | NOT NULL DEFAULT 0 | `@Version` | 乐观锁版本号 |
| `deleted` | `TINYINT(1)` | NOT NULL DEFAULT 0 | — | 逻辑删除：0=正常，1=删除 |

### JPA 基类

```java
@Getter
@Setter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "snowflake")
    @GenericGenerator(name = "snowflake", type = SnowflakeIdGenerator.class)
    private Long id;

    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(name = "updated_at", nullable = false)
    private LocalDateTime updatedAt;

    @Version
    @Column(name = "version", nullable = false)
    private Integer version;

    @Column(name = "deleted", nullable = false)
    private Boolean deleted = false;
}
```

### 启用审计

```java
@Configuration
@EnableJpaAuditing
public class JpaConfig {}
```

### 公共索引

```sql
INDEX idx_created_at (created_at),
INDEX idx_deleted (deleted)
```

---

## 3. NULL / NOT NULL 约束原则

### 核心判断

```
NOT NULL — 业务上「必须有值」且「现在就知道值」的字段
NULL     — 业务上「可选」或「现在还不知道值」的字段
```

### 逐条原则

| # | 原则 | 说明 | 示例 |
|---|------|------|------|
| 1 | **主键必 NOT NULL** | 自解释 | `id BIGINT NOT NULL` |
| 2 | **公共字段必 NOT NULL** | created_at / updated_at / deleted / version | 见 §2 |
| 3 | **业务必填字段 NOT NULL** | 缺了它业务跑不动 | `conversations.title` / `messages.content` |
| 4 | **可选/延时字段用 NULL** | 该信息在创建时尚未产生 | `messages.model_name`（仅 assistant 有值） |
| 5 | **数值型默认 0 而非 NULL** | 避免 SUM/AVG 等聚合函数被 NULL 污染 | `token_count INT NOT NULL DEFAULT 0` |
| 6 | **布尔型默认 false** | 三态布尔是设计坏味道 | `deleted TINYINT(1) NOT NULL DEFAULT 0` |
| 7 | **外键必 NOT NULL** | 除非关系本身可选 | `messages.conversation_id BIGINT NOT NULL` |
| 8 | **时间字段用 NULL 表示未发生** | 未来事件尚未触发 | `conversations.archived_at DATETIME NULL` |

### 文本字段：NULL vs 空串

| 场景 | 用 | 原因 |
|------|----|------|
| 用户未输入、模型尚未生成 | `NULL` | 表示「尚未产生」 |
| 用户输入后手动清空 | `''` | 表示「主动置空」 |
| JPA 层必填校验 | `@Column(nullable = false)` + `@NotBlank` | 双重保障 |
| DDL 默认值 | `DEFAULT ''` 或 `DEFAULT NULL` 按上表原则 | — |

---

## 4. 索引策略

### 必须加索引的场景

| 场景 | 索引类型 | 示例 |
|------|----------|------|
| 主键 | `PRIMARY KEY`（天然聚簇索引） | `id` |
| 外键 / 关联字段 | 单列索引 | `conversation_id` |
| 唯一约束 | `UNIQUE INDEX` | `users.email` |
| ORDER BY 排序字段 | 单列或复合索引 | `created_at` |
| WHERE 等值查询 | 单列或复合索引左侧 | `status = 'ACTIVE'` |
| WHERE 范围查询 | 复合索引右侧 | `created_at > ?` |
| WHERE 状态+时间组合 | 复合索引 `(status, created_at)` | 高频会话列表查询 |
| 逻辑删除过滤 | 单列索引 | `deleted` |
| JOIN 连接字段 | 单列索引 | 关联查询的外键 |

### 复合索引的"最左前缀"规则

```sql
INDEX idx_a_b_c (a, b, c)
-- ✅ 可以用: WHERE a=1, WHERE a=1 AND b=2, WHERE a=1 AND b=2 AND c=3
-- ❌ 不可用: WHERE b=2, WHERE c=3, WHERE b=2 AND c=3
```

### 索引控制原则

- 单表索引数 ≤ 5
- 每一版表结构**必须**包含基于业务查询场景的索引
- 用 `EXPLAIN` 检查查询：`type` 应为 `ref` 或 `range`，避免 `ALL`（全表扫描）
- 避免 `filesort`（ORDER BY 无索引覆盖时触发）

### 索引设计口诀

```
等值查询左开头，范围查询放右边
排序字段跟上走，覆盖索引最省事
小表不建大表建，索引维护有成本
```

---

## 5. Flyway 迁移规范

### 命名规则

```
V{主版本}.{次版本}__{描述}.sql
```

| 示例 | 说明 |
|------|------|
| `V1.0__init_conversations.sql` | 首版本建表 |
| `V1.1__init_messages.sql` | 首版本建表 |
| `V2.0__add_user_id_to_conversations.sql` | 后续变更：加字段 |

### JPA DDL 配合

```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: validate    # 启动时校验 Entity 与表一致
    show-sql: false
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true
```

| 原则 | 说明 |
|------|------|
| ✅ Flyway 负责 DDL | 所有建表/改表通过 `V*.sql` 脚本 |
| ✅ JPA 负责 ORM | Entity 注解映射已有表结构 |
| ❌ 禁止 `ddl-auto: update` | 避免不可控的隐式 DDL |

---

## 6. AI 对话核心表示例

### conversations（对话表）

```sql
CREATE TABLE conversations (
    id          BIGINT       NOT NULL PRIMARY KEY COMMENT 'Snowflake ID',
    user_id     BIGINT       NOT NULL        COMMENT '所属用户 ID',
    title       VARCHAR(200) NOT NULL        COMMENT '对话标题',
    summary     TEXT         NULL            COMMENT 'AI 生成的对话摘要（异步填充）',
    status      VARCHAR(20)  NOT NULL DEFAULT 'ACTIVE'
                                           COMMENT 'ACTIVE / ARCHIVED',
    created_at  DATETIME(3)  NOT NULL DEFAULT CURRENT_TIMESTAMP(3),
    updated_at  DATETIME(3)  NOT NULL DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3),
    version     INT          NOT NULL DEFAULT 0,
    deleted     TINYINT(1)   NOT NULL DEFAULT 0,

    INDEX idx_user_id_created (user_id, created_at),
    INDEX idx_status_created (status, created_at),
    INDEX idx_created_at (created_at),
    INDEX idx_deleted (deleted)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### messages（消息表）

```sql
CREATE TABLE messages (
    id                   BIGINT       NOT NULL PRIMARY KEY COMMENT 'Snowflake ID',
    conversation_id      BIGINT       NOT NULL        COMMENT '所属会话',
    role                 VARCHAR(20)  NOT NULL        COMMENT 'user / assistant / system',
    content              MEDIUMTEXT   NOT NULL        COMMENT '消息内容',
    content_type         VARCHAR(20)  NOT NULL DEFAULT 'text' COMMENT 'text / markdown / tool_call',
    token_count          INT          NOT NULL DEFAULT 0  COMMENT 'Token 数',
    model_name           VARCHAR(100) NULL            COMMENT 'AI 模型名（仅 assistant）',
    model_response_time_ms INT        NULL            COMMENT '响应耗时（仅 assistant）',
    metadata_json        JSON         NULL            COMMENT '扩展元数据',
    parent_message_id    BIGINT       NULL            COMMENT '分支对话父消息',
    created_at  DATETIME(3)  NOT NULL DEFAULT CURRENT_TIMESTAMP(3),
    updated_at  DATETIME(3)  NOT NULL DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3),
    version     INT          NOT NULL DEFAULT 0,
    deleted     TINYINT(1)   NOT NULL DEFAULT 0,

    INDEX idx_conversation_created (conversation_id, created_at),
    INDEX idx_created_at (created_at),
    INDEX idx_deleted (deleted),
    CONSTRAINT fk_conversation FOREIGN KEY (conversation_id) REFERENCES conversations(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

> 本文档为正式数据库设计规范，所有新表、新字段、新索引的开发须遵循以上规则。
> 规范随项目演进持续更新，变更历史记录在 `openspec/changes/` 中。
