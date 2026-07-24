# Proposal: database-design-spec

## 为什么做这个变更？

现有项目中 `project.md` 记录的后端技术栈为 **MyBatis-Plus + PostgreSQL**，但实际项目方向为 **AI 对话应用**，需要选择合适的后端栈来支撑业务。

### 变更原因

| 维度 | 当前（MyBatis-Plus + PostgreSQL） | 新方案（JPA + MySQL + Flyway） |
|------|-----------------------------------|-------------------------------|
| ORM 选型 | MyBatis-Plus（半自动化 SQL） | JPA（全自动 ORM + 实体驱动建模） |
| 数据库 | PostgreSQL | MySQL |
| 迁移管理 | schema.sql / DDL-auto | Flyway 版本化迁移 |
| 实体设计 | Mapper + XML 映射 | Entity + Repository（规范驱动建模） |
| 团队匹配 | （据用户反馈）需调整 | JPA 在 AI/数据模型场景下更适合快速迭代 |

### AI 对话场景的特殊要求

- 对话/消息实体的复杂关联（Conversation ⇉ Message 一对多）
- 长文本存储（LLM 响应的 message content）
- 高写入量（消息流式写入）、低延迟查询（按会话查询）
- 规范的字段约束和数据完整性

## 变更了什么？

1. 数据库设计规范（本提案核心）：主键策略、公共字段、NULL/NOT NULL 原则、索引策略
2. 后端依赖切换：JPA + MySQL Driver + Flyway，去除 MyBatis-Plus
3. `application.yml` 配置切换：MySQL 数据源 + Flyway + JPA
4. `openspec/project.md` 更新：技术栈记录

## 影响范围

- `backend/pom.xml` — 移除 MyBatis-Plus & PostgreSQL，新增 JPA、MySQL Connector、Flyway
- `backend/src/main/resources/application.yml` — 替换为 MySQL 数据源 + Flyway 配置
- `backend/src/main/resources/db/migration/` — Flyway 迁移脚本目录
- `openspec/specs/database-design.md` — 新增数据库设计规范
- `openspec/project.md` — 更新技术栈

## 风险评估

| 风险 | 影响 | 缓解措施 |
|------|------|----------|
| 实体设计不规范导致后期改表困难 | 频繁 DDL 变更 | 通过 Flyway 版本化管理所有 DDL，不依赖 DDL-auto |
| NULL/NOT NULL 策略不统一 | 数据质量下降 | 规范明确每类字段的约束原则 |
| 索引缺失导致查询慢 | 对话列表/搜索响应缓慢 | 明确索引策略，每张表首版本就涵盖查询索引 |

## 验收标准

- [ ] `openspec/specs/database-design.md` 发布，涵盖主键策略、公共字段、NULL/NOT NULL 约束、索引策略
- [ ] 主键策略给出 JPA 下的具体实现方案（含 @GeneratedValue 配置）
- [ ] 规范结合 AI 对话场景给出示例（conversations / messages 表）
