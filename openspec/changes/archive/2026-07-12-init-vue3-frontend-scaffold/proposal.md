# Proposal: init-vue3-frontend-scaffold

## 为什么做这个变更？

后端脚手架已搭建完成（`backend/` Spring Boot 3.4），前端部分为空。需要搭建 Vue 3 前端骨架，实现前后端分离开发的基础。

## 变更了什么？

在项目根目录下创建 `frontend/` 模块，使用 **Vue 3 + TypeScript + Vite 6** 搭建最小可运行骨架，通过 Vite 代理与后端联调。

## 影响范围

- 新增 `frontend/` 目录（含 package.json、源码、配置）
- 更新 `openspec/project.md`（前端技术栈）
- 更新 `.gitignore`（Node.js 忽略项）

## 风险评估

| 风险 | 影响 | 缓解措施 |
|------|------|----------|
| 组件库版本未来可能不兼容 | 升级时需回归测试 | 锁定版本，SemVer 约束 |

## 验收标准

- [x] `cd frontend && corepack enable && pnpm install` 安装成功
- [x] `pnpm dev` 启动开发服务器
- [x] `http://localhost:5173` 可访问首页
- [x] Vite 代理 `/api` → `localhost:8080`，页面请求 `/api/health` 不跨域
- [x] `pnpm build` 构建成功，产物在 `dist/`
- [x] `openspec/project.md` 前端技术栈已更新
