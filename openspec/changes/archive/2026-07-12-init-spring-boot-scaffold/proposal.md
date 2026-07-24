# Proposal: init-spring-boot-scaffold

## 为什么做这个变更？

项目当前只有 OpenSpec 规范和 Agent 配置，缺少可运行的代码框架。需要一个 Spring Boot 后端脚手架作为全栈项目的起点。

## 变更了什么？

在项目根目录下创建 `backend/` 模块，搭建 Spring Boot 3.4 + JDK 21 + Maven 的最小可运行骨架。

## 影响范围

- 新增 `backend/` 目录（含 pom.xml、源码、测试）
- 更新 `openspec/project.md`（技术栈）
- 根目录结构由当前的规范文件扩展为 `backend/` + 待定 `frontend/` + `openspec/` + `.qoder/`

## 风险评估

| 风险 | 影响 | 缓解措施 |
|------|------|----------|
| 后续前端框架可能与后端目录约定不一致 | 需要调整目录约定 | 使用业界常见的 `backend/` + `frontend/` 平级约定，主流框架兼容 |

## 验收标准

- [x] `cd backend && ./mvnw spring-boot:run` 能成功启动
- [x] `GET http://localhost:8080/api/health` 返回 `{"status":"UP"}`
- [x] `./mvnw test` 全部测试通过
- [x] `openspec/project.md` 技术栈已更新
