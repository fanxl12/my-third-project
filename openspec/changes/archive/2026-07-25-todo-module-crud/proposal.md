# Proposal: todo-module-crud

## 为什么做这个变更？

前后端骨架已搭建完成，需要第一个实际功能模块来验证全栈链路。ToDo 待办清单作为入门 CRUD 模块，覆盖前后端联调、数据库持久化、校验、异常处理等核心工程实践。

## 变更了什么？

在现有项目中新增 ToDo 待办清单完整 CRUD 功能：
- 后端：Spring Boot + JPA + PostgreSQL 持久化，6 个 REST 接口
- 前端：Vue 3 待办管理页面，列表/创建/切换完成/删除

## 影响范围

- `backend/pom.xml` — 新增 JPA、PostgreSQL、Validation 依赖
- `backend/src/main/resources/application.yml` — 新增 PostgreSQL 数据源配置
- `backend/src/main/java/com/fanxl/demo/` — 新增 entity/repository/service/controller/dto/exception 包
- `frontend/src/` — 新增 types/todo.ts, api/todo.ts, TodoView.vue, todo/ 组件集
- `frontend/src/router/index.ts` — 追加 `/todos` 路由
- `openspec/project.md` — 更新数据库选型

## 风险评估

| 风险 | 影响 | 缓解措施 |
|------|------|----------|
| PostgreSQL 连接失败 | 应用无法启动 | application.yml 配置连接池超时，启动时自动检测 |
| 数据库表变更 | 需手动重建 | 使用 JPA `ddl-auto: update` 自动建表 |
| 前端与后端字段不匹配 | 数据展示错误 | 统一基于 Spec 的 TodoItem 定义，camelCase 约束 |

## 验收标准

- [ ] `mvn test` 后端测试全部通过
- [ ] `pnpm build` 前端构建通过
- [ ] `GET /api/todos` 返回空列表 `[]`
- [ ] `POST /api/todos` 创建成功后列表返回新项
- [ ] `PUT /api/todos/{id}` 更新标题和完成状态
- [ ] `DELETE /api/todos/{id}` 删除后列表不再包含
- [ ] 异常路径：空标题 400、不存在 ID 404
- [ ] 前端 `/todos` 页面：展示列表 + 创建表单 + 切换完成 + 删除
