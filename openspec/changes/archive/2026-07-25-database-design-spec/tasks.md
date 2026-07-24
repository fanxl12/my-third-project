# Tasks: database-design-spec

## Phase 1: 规范文档

- [ ] **Task 1.1** 编写 `openspec/specs/database-design.md` 规范文档
  - 从 design.md 提炼为正式团队规范（保留 all sections）
  - 使用 Markdown 格式，加入目录（TOC）
  - **验证**: 文档可通过，内容完整（主键策略 / 公共字段 / NULL NOT NULL / 索引 / Flyway）

- [ ] **Task 1.2** 发布规范后团队 Review
  - 确认 Snowflake ID 实现方案（Java 生成器代码）
  - 确认每张表公共字段是否齐全
  - 确认索引策略是否覆盖所有高频查询场景
  - **验证**: 反馈无争议

## Phase 2: 后端依赖切换

- [ ] **Task 2.1** `pom.xml` 替换依赖
  - 移除：`mybatis-plus-spring-boot3-starter`、`postgresql`
  - 新增：`spring-boot-starter-data-jpa`、`mysql-connector-j`、`flyway-core`、`flyway-mysql`
  - **验证**: `mvn dependency:tree` 列出新增依赖

- [ ] **Task 2.2** `application.yml` 替换配置
  - 移除 PostgreSQL + MyBatis-Plus 配置
  - 新增 MySQL 数据源 + JPA + Flyway 配置（参考 design.md §5.2 / §5.3）
  - **验证**: `mvn spring-boot:run` 启动成功（MySQL 连接正常）

- [ ] **Task 2.3** 创建 Flyway 迁移目录
  - `src/main/resources/db/migration/` 目录
  - 准备 V1.0__init_conversations.sql / V1.1__init_messages.sql
  - **验证**: Flyway 启动时自动执行迁移

## Phase 3: 清理与验证

- [ ] **Task 3.1** 删除 MyBatis-Plus 相关代码
  - 删除旧 Mapper / XML / Entity 中的 MyBatis-Plus 注解
  - 统一改为 JPA Entity + Repository
  - **验证**: `mvn compile` 通过

- [ ] **Task 3.2** 更新 `openspec/project.md`
  - 技术栈：MyBatis-Plus → JPA, PostgreSQL → MySQL
  - 新增 Flyway 条目
  - **验证**: 文档与代码一致

- [ ] **Task 3.3** 全量验证
  - `mvn test` 后端测试全部通过
  - 数据库连接正常，Flyway 版本记录正确
  - **验证**: 无错误
