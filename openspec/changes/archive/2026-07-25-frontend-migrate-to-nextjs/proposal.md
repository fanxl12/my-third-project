# Proposal: frontend-migrate-to-nextjs

## 为什么做这个变更？

当前前端子模块使用 **Vue 3 + Element Plus + Vite 6** 技术栈，与团队核心约束存在三重不匹配：

### 1. 团队技能不匹配
- 团队 3 人**全员熟悉 React**，无 Vue 3 生产经验
- Vue 3 Composition API、SFC 语法、响应式系统需要从零学习，预估 **1~2 周**才能达到同等生产力
- 项目周期仅 **8 周**，不允许在框架切换上浪费 25% 的工期

### 2. 部署体验不匹配
- Vue 3 SPA 部署到 Vercel 非一等公民，需额外配置 `vercel.json` 处理 SPA fallback
- **Next.js 是 Vercel 原生框架**，`git push` 即自动部署，零配置 SSR/SSG/ISR 渲染策略

### 3. AI 生成质量不匹配
- AI 编程模型（GPT/Claude）对 React/Next.js 代码生成质量显著高于 Vue 3
- React 训练数据量约为 Vue 的 **3~5 倍**，生成的组件更符合最佳实践、幻觉更少
- 项目以 AI 辅助开发为主，选择 AI 生成质量最高的方案是核心约束

### 迁移目标

将 `frontend/` 子模块从 Vue 3 + Element Plus 迁移到 **Next.js + React + Tailwind CSS 4**，保持与后端 Spring Boot 的 REST API 通信不变。

## 变更了什么？

| 层面 | 旧 | 新 |
|------|-----|-----|
| **框架** | Vue 3 + Vite 6 | Next.js 15（App Router） |
| **UI 语言** | Vue SFC（template + script setup） | React TSX（Server Components 优先） |
| **组件库** | Element Plus（Vue 专属） | 移除，使用 Tailwind CSS 4 + Headless UI 手写 |
| **路由** | Vue Router 4（SPA） | Next.js App Router（文件路由 + SSR） |
| **状态管理** | Pinia | React Context + useReducer（简单场景）/ Zustand（复杂场景） |
| **HTTP 客户端** | Axios | Axios（保留，服务端 fetch + 客户端 axios） |
| **样式** | Tailwind CSS 4 | Tailwind CSS 4（保留，配置方式调整） |
| **构建工具** | Vite 6 | Next.js 内置（Turbopack） |
| **类型检查** | vue-tsc | TypeScript 编译器（tsc） |
| **代码检查** | ESLint + eslint-plugin-vue | ESLint + eslint-plugin-react-hooks |
| **部署** | Vite SPA + vercel.json | Vercel 原生部署（零配置） |

## 影响范围

- **`frontend/` 子模块**：整个源码目录重写，package.json 依赖完全替换
- **`AGENTS.md`**：已更新前端技术栈描述（下次 Git commit 时由子模块提交）
- **`openspec/project.md`**：已更新前端技术栈及架构决策（已完成）
- **`openspec/specs/todo-module.md`**：前端验收标准中的 Vue 组件名（`TodoView.vue`、`TodoForm.vue` 等）需更新为 React 组件名
- **`openspec/changes/todo-module-crud/`**：现有 todo-module 变更中的前端部分需基于新框架重新实现
- **后端 `backend/`**：**无影响**，REST API 接口不变

## 风险评估

| 风险 | 影响 | 缓解措施 |
|------|------|----------|
| 迁移期间前端不可用 | 中期 Demo 受影响 | 整个迁移在 1 个 Phase 内完成（约 2~3 天），期间后端独立可用 |
| todo-module-crud 前端代码需重写 | 已有前端工作量作废 | todo-module-crud 前端部分尚未开始实现（仍在 proposal 阶段），无沉没成本 |
| Tailwind CSS 4 在 Next.js 中配置方式不同 | 样式异常 | 使用 `@tailwindcss/postcss` 插件适配 Next.js |
| Element Plus 组件交互需手写替代 | 开发效率下降 | 使用 Headless UI 提供无样式的可访问原语 + Tailwind 样式，代码量可控 |
| SSR 模式下浏览器 API（localStorage 等）报错 | 运行时错误 | 浏览器 API 调用包裹在 `useEffect` / `'use client'` 中 |

## 验收标准

- [ ] `cd frontend && pnpm dev` 启动 Next.js 开发服务器，端口 3000
- [ ] 访问 `http://localhost:3000` 渲染首页（Home 页面），布局正常
- [ ] 首页「检测后端连接」按钮可正常请求 `/api/health`（通过 Next.js rewrites 代理）
- [ ] `pnpm build` 构建成功，无 TypeScript 错误
- [ ] `pnpm lint` ESLint 通过
- [ ] Vercel 部署成功（`git push` 后自动触发），线上页面可访问
- [ ] 所有现有功能（Home 页面、健康检测、计数器）在 Next.js 中行为一致
- [ ] `openspec/specs/todo-module.md` 前端验收标准中的组件引用已更新为 React 名称
- [ ] 删除所有 Vue 专属文件（`.vue`、`vue-router`、`pinia`、`element-plus` 引用）
