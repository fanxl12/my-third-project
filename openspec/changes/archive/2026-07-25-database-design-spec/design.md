# Design: database-design-spec

## 概述

制定 AI 对话应用的数据库设计规范，涵盖主键策略、公共字段、NULL/NOT NULL 约束、索引策略四大主题，基于 **Spring Boot + JPA + MySQL + Flyway** 技术栈。

所有规范结合 AI 对话业务场景（conversations / messages 等核心表）给出具体示例。

---

## 1. 主键策略

### 1.1 选型：Snowflake ID（雪花算法）

| 策略 | 优点 | 缺点 | 结论 |
|------|------|------|:----:|
| **自增 ID** `AUTO_INCREMENT` | 简单、性能好、索引紧凑 | 暴露记录数、分库分表冲突、不安全 | ❌ 不采用 |
| **UUID v4** `CHAR(36)` | 全局唯一、不暴露信息 | 无序、索引碎片严重、存储大(36字节)、查询性能差 | ❌ 不采用 |
| **Snowflake ID** `BIGINT(19)` | 全局唯一、趋势递增、有时间语义、防暴露 | 需引入 ID 生成器 | ✅ **采用** |

**理由**：AI 对话应用天然需要分布式 ID（未来可能多实例/分库），Snowflake 64-bit BIGINT 与 MySQL InnoDB 聚簇索引高度兼容，趋势递增减少页分裂。

### 1.2 Snowflake ID 结构

```
 0         41          51           64
├─┼───────────┼──────────┼────────────┤
│0│ timestamp │ worker   │  sequence  │
│ │  41bits   │ 10bits   │  12bits    │
```

- **1-bit**: 符号位（始终 0）
- **41-bit**: 毫秒时间戳（69 年跨度）
- **10-bit**: 机器/工作节点 ID
- **12-bit**: 毫秒内序列号（4096/ms）

### 1.3 JPA 实现

引入雪花 ID 生成器：

```java
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "snowflake")
@GenericGenerator(name = "snowflake", type = SnowflakeIdGenerator.class)
private Long id;
```

> `SnowflakeIdGenerator` 实现 `IdentifierGenerator` 接口。

### 1.4 MySQL DDL

```sql
CREATE TABLE conversations (
    id BIGINT NOT NULL PRIMARY KEY COMMENT 'Snowflake ID',
    -- ...
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 1.5 TIP：何时可以例外用自增？

**仅在以下情况**可考虑使用 `AUTO_INCREMENT`：
- 纯内部配置表（字典表、枚举表）
- 生命周期短、不会分表的临时表

常规业务表（包括关联表）一律使用 Snowflake ID。

---

## 2. 每张表必须有的公共字段

### 2.1 公共字段清单

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| `id` | `BIGINT` | PK, NOT NULL | 主键，Snowflake ID |
| `created_at` | `DATETIME(3)` | NOT NULL, 默认值 `CURRENT_TIMESTAMP(3)` | 创建时间（毫秒精度） |
| `updated_at` | `DATETIME(3)` | NOT NULL, ON UPDATE CURRENT_TIMESTAMP(3) | 最后更新时间（毫秒精度） |
| `version` | `INT` | NOT NULL DEFAULT 0 | 乐观锁版本号（JPA @Version） |
| `deleted` | `TINYINT(1)` | NOT NULL DEFAULT 0 | 逻辑删除标志（0=正常，1=删除） |

### 2.2 JPA 基类（@MappedSuperclass）

```java
@Getter
@Setter
@MappedSuperclass
public abstract class BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "snowflake")
    @GenericGenerator(name = "snowflake", type = SnowflakeIdGenerator.class)
    private Long id;

    @Column(name = "created_at", nullable = false, updatable = false)
    @CreatedDate
    private LocalDateTime createdAt;

    @Column(name = "updated_at", nullable = false)
    @LastModifiedDate
    private LocalDateTime updatedAt;

    @Version
    @Column(name = "version", nullable = false)
    private Integer version;

    @Column(name = "deleted", nullable = false)
    private Boolean deleted = false;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
        if (deleted == null) deleted = false;
        if (version == null) version = 0;
    }

    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
}
```

### 2.3 启用 JPA Auditing

```java
@Configuration
@EnableJpaAuditing
public class JpaConfig {
}
```

### 2.4 公共索引

每张表（包括关联表）必须包含以下索引：

```sql
INDEX idx_created_at (created_at),
INDEX idx_deleted (deleted)
```

---

## 3. 字段约束原则（NULL vs NOT NULL）

### 3.1 核心原则

```
NOT NULL — 业务上「必须有值」且「现在就知道值」的字段
NULL     — 业务上「可选」或「现在还不知道值」的字段
```

### 3.2 具体原则

| 原则 | 说明 | 示例 |
|------|------|------|
| ① 主键必 NOT NULL | 自解释 | `id BIGINT NOT NULL` |
| ② 公共字段必 NOT NULL | created_at / updated_at / deleted / version | 见上 |
| ③ 业务必填字段 NOT NULL | 缺少该字段业务无法运行 | `conversations.title` |
| ④ 可选/后期填充字段 NULL | 业务不依赖该字段 | `messages.model_response_time_ms` |
| ⑤ 数值型默认 0 而非 NULL | 聚合/计算避免 NULL 污染 | `message.token_count INT NOT NULL DEFAULT 0` |
| ⑥ 布尔型默认 false 而非 NULL | 三态布尔是设计坏味道 | `message.is_from_user TINYINT(1) NOT NULL DEFAULT 1` |
| ⑦ 文本字段 NULL 与空串分清 | NULL=未提供，''=主动置空 | 见 3.4 |
| ⑧ 外键/关联字段 NOT NULL | 除非关系可选 | `message.conversation_id BIGINT NOT NULL` |
| ⑨ 时间字段可选用 NULL | 未来事件尚未发生 | `conversations.scheduled_at DATETIME(3) NULL` |

### 3.3 AI 对话场景示例

```sql
CREATE TABLE conversations (
    id          BIGINT       NOT NULL PRIMARY KEY COMMENT 'Snowflake ID',
    title       VARCHAR(200) NOT NULL        COMMENT '对话标题：用户创建时必填',
    summary     TEXT         NULL            COMMENT 'AI 生成的对话摘要，异步填充',
    status      VARCHAR(20)  NOT NULL DEFAULT 'ACTIVE'
                                        COMMENT '会话状态：ACTIVE/ARCHIVED/DELETED',
    -- 公共字段
    created_at  DATETIME(3)  NOT NULL DEFAULT CURRENT_TIMESTAMP(3),
    updated_at  DATETIME(3)  NOT NULL DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3),
    version     INT          NOT NULL DEFAULT 0,
    deleted     TINYINT(1)   NOT NULL DEFAULT 0
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE messages (
    id                   BIGINT       NOT NULL PRIMARY KEY COMMENT 'Snowflake ID',
    conversation_id      BIGINT       NOT NULL        COMMENT '关联会话（必填）',
    role                 VARCHAR(20)  NOT NULL        COMMENT '消息角色：user/assistant/system',
    content              TEXT         NOT NULL        COMMENT '消息内容（必填，LLM 响应/用户输入）',
    content_type         VARCHAR(20)  NOT NULL DEFAULT 'text'
                                                     COMMENT '内容类型：text/markdown/tool_call',
    token_count          INT          NOT NULL DEFAULT 0  COMMENT 'Token 数（聚合用，默认 0）',
    model_name           VARCHAR(100) NULL            COMMENT 'AI 模型名：仅 assistant 消息有值',
    model_response_time_ms INT        NULL            COMMENT '响应耗时 ms：仅 assistant 消息',
    metadata_json        JSON         NULL            COMMENT '扩展元数据（可选）',
    parent_message_id    BIGINT       NULL            COMMENT '父消息 ID（分支对话可选）',
    -- 公共字段
    created_at  DATETIME(3)  NOT NULL DEFAULT CURRENT_TIMESTAMP(3),
    updated_at  DATETIME(3)  NOT NULL DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3),
    version     INT          NOT NULL DEFAULT 0,
    deleted     TINYINT(1)   NOT NULL DEFAULT 0,

    FOREIGN KEY (conversation_id) REFERENCES conversations(id),
    FOREIGN KEY (parent_message_id) REFERENCES messages(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 3.4 文本字段：NULL vs 空串

| 场景 | 用 | 原因 |
|------|----|------|
| 用户未输入、模型未生成 | `NULL` | 表示「尚未产生」 |
| 用户输入了空内容、手动清空 | `''` | 表示「主动设为空」 |
| JPA String 字段 | `@Column(nullable = false)` + 空串校验 | 业务必填时用 NOT NULL 加 @NotBlank 校验 |
| DDL 默认 | `DEFAULT ''` 或 `DEFAULT NULL` 按上表原则 | — |

---

## 4. 索引策略

### 4.1 核心原则

```
① 每个 WHERE 条件涉及的字段都要评估索引
② 复合索引遵循"最左前缀"规则
③ 覆盖索引优于回表查询
④ 索引不是越多越好——控制单表索引数 ≤ 5
```

### 4.2 什么情况必须加索引

| 场景 | 索引类型 | 示例 |
|------|----------|------|
| **主键** | PRIMARY KEY（聚簇索引） | `id` |
| **外键 / 关联查询** | 单列索引 | `conversation_id` |
| **唯一约束** | UNIQUE INDEX | `users.email` |
| **排序字段（ORDER BY）** | 单列或复合索引 | `created_at` |
| **范围查询（> / < / BETWEEN）** | 复合索引的右侧字段 | `created_at` |
| **等值查询（WHERE =）** | 单列或复合索引左侧 | `status` |
| **逻辑删除过滤** | 部分索引（MySQL 8+）或单列索引 | `deleted` |
| **状态筛选 + 时间排序** | 复合索引：`(status, created_at)` | 业务高频查询 |

### 4.3 AI 对话场景索引设计

```sql
-- conversations 表
ALTER TABLE conversations ADD INDEX idx_status_created (status, created_at) COMMENT '按状态+时间排序查询会话列表';
ALTER TABLE conversations ADD INDEX idx_user_id_created (user_id, created_at) COMMENT '按用户查询会话列表（如有用户模块）';

-- messages 表
ALTER TABLE messages ADD INDEX idx_conversation_created (conversation_id, created_at) COMMENT '按会话查询消息列表，按时间排序';
ALTER TABLE messages ADD UNIQUE INDEX idx_uniq_parent_message (parent_message_id) COMMENT '分支消息唯一约束（可选）';

-- 公共索引（每张表）
ALTER TABLE conversations ADD INDEX idx_created_at (created_at);
ALTER TABLE conversations ADD INDEX idx_deleted (deleted);
ALTER TABLE messages ADD INDEX idx_created_at (created_at);
ALTER TABLE messages ADD INDEX idx_deleted (deleted);
```

### 4.4 索引优化口诀

```
等值查询左开头，范围查询放右边
排序字段跟上走，覆盖索引最省事
小表不建大表建，索引维护有成本
EXPLAIN 看 type，ref/range 是及格
rows 过大要警惕，extra filesort 要优化
```

### 4.5 JPA @Table 索引注解

```java
@Entity
@Table(name = "messages", indexes = {
    @Index(name = "idx_conversation_created", columnList = "conversation_id,created_at"),
    @Index(name = "idx_created_at", columnList = "created_at"),
    @Index(name = "idx_deleted", columnList = "deleted")
})
public class Message extends BaseEntity {
    // ...
}
```

---

## 5. Flyway 迁移规范

### 5.1 命名规则

```
V{主版本}.{次版本}__{描述}.sql
```

示例：
```
V1.0__init_conversations.sql
V1.1__init_messages.sql
V2.0__add_user_id_to_conversations.sql
```

### 5.2 Flyway 配置 (application.yml)

```yaml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true
    baseline-version: 0
```

### 5.3 DDL 原则（Flyway + JPA 配合）

- **Flyway 负责 DDL**：所有建表/改表通过 `V*.sql` 脚本
- **JPA 负责 DML 查询**：Entity 映射已有表结构
- **DDL-auto**：设置为 `validate`（启动时校验 Entity 与表结构一致，不一致报错）
- **禁止** `ddl-auto: update`（会导致不可控的隐式 DDL）

```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
```

---

## 6. AI 对话核心表 Schema

> 结合以上所有规范，给出首版本两张核心表的完整 Flyway 迁移脚本。

**V1.0__init_conversations.sql**

```sql
CREATE TABLE conversations (
    id          BIGINT       NOT NULL PRIMARY KEY COMMENT 'Snowflake ID',
    user_id     BIGINT       NOT NULL        COMMENT '所属用户 ID',
    title       VARCHAR(200) NOT NULL        COMMENT '对话标题',
    summary     TEXT         NULL            COMMENT 'AI 对话摘要',
    status      VARCHAR(20)  NOT NULL DEFAULT 'ACTIVE'
                                           COMMENT 'ACTIVE/ARCHIVED',
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

**V1.1__init_messages.sql**

```sql
CREATE TABLE messages (
    id                   BIGINT       NOT NULL PRIMARY KEY COMMENT 'Snowflake ID',
    conversation_id      BIGINT       NOT NULL        COMMENT '所属会话',
    role                 VARCHAR(20)  NOT NULL        COMMENT 'user/assistant/system',
    content              MEDIUMTEXT   NOT NULL        COMMENT '消息内容',
    content_type         VARCHAR(20)  NOT NULL DEFAULT 'text',
    token_count          INT          NOT NULL DEFAULT 0,
    model_name           VARCHAR(100) NULL,
    model_response_time_ms INT        NULL,
    metadata_json        JSON         NULL,
    parent_message_id    BIGINT       NULL,
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

## 7. YAGNI 排除清单

以下内容本版本**不引入**，未来按需加入：

- ❌ 分库分表中间件（ShardingSphere）— 当前单库足够
- ❌ 多主键类型支持— 全部用 Snowflake ID
- ❌ 数据库加密/脱敏层 — 后续安全需求再引入
- ❌ 软删除自动过滤插件 — JPA 层手动加 `WHERE deleted = 0`
- ❌ 全局 ID 中心/发号器微服务 — 本地雪花算法即可
