# Tasks: frontend-migrate-to-nextjs

## Phase 1: 清理旧框架

- [ ] **Task 1.1** 删除所有 Vue 源码文件并验证
  - 删除 `src/api/`、`src/components/`、`src/router/`、`src/stores/`、`src/views/` 目录
  - 删除 `src/App.vue`、`src/main.ts`、`src/style.css`、`src/vite-env.d.ts`
  - 删除 `index.html`
  - 删除 `vite.config.ts`
  - **验证**: `frontend/src/` 目录为空（仅保留可能的 `.gitkeep`），`git status` 显示所有 Vue 文件已删除

- [ ] **Task 1.2** 清空 package.json 依赖
  - 移除 `dependencies`: `vue`、`vue-router`、`pinia`、`element-plus`
  - 移除 `devDependencies`: `@vitejs/plugin-vue`、`vue-tsc`、`eslint-plugin-vue`、`@tailwindcss/vite`
  - 保留 `axios`、`tailwindcss`、`@typescript-eslint/*`、`prettier`、`typescript`（将在 Next.js 中复用）
  - 移除 `pnpm-lock.yaml`
  - **验证**: `package.json` 不包含任何 Vue 相关依赖

## Phase 2: 初始化 Next.js 项目

- [ ] **Task 2.1** 安装 Next.js + React 核心依赖
  - `pnpm add next@latest react@latest react-dom@latest`
  - `pnpm add -D @types/react @types/react-dom @types/node`
  - **验证**: `pnpm list next react react-dom` 显示已安装

- [ ] **Task 2.2** 安装 Next.js 配套工具链
  - `pnpm add -D eslint-config-next @tailwindcss/postcss`
  - 为 `tailwindcss` 从 devDependencies 移至 dependencies（Next.js 生产构建需要）
  - **验证**: `pnpm list @tailwindcss/postcss eslint-config-next` 显示已安装

- [ ] **Task 2.3** 更新 `package.json` 的 scripts
  ```json
  {
    "scripts": {
      "dev": "next dev",
      "build": "next build",
      "start": "next start",
      "lint": "next lint",
      "type-check": "tsc --noEmit",
      "format": "prettier --write \"**/*.{ts,tsx,css,json}\""
    }
  }
  ```
  - **验证**: `pnpm dev` 启动成功（Next.js 自动创建 `app/` 骨架）

## Phase 3: 核心文件创建（RED — 文件结构验证）

- [ ] **Task 3.1** 创建 Next.js 项目骨架文件
  - `app/layout.tsx` — 根布局（含 `<html lang="zh-CN">`、metadata、全局样式引入）
  - `app/globals.css` — Tailwind CSS 4 入口（`@import "tailwindcss"`）
  - `app/page.tsx` — 首页基础骨架（标题 + 副标题，纯静态）
  - `tsconfig.json` — 更新为 Next.js 标准配置（含 `paths: { "@/*": ["./*"] }`）
  - `next.config.ts` — API rewrites 代理配置
  - `eslint.config.mjs` — 替换为 `eslint-config-next`
  - `postcss.config.mjs` — PostCSS 配置
  - 删除 `src/` 空目录
  - **验证**: `pnpm dev` 访问 `http://localhost:3000`，显示标题和副标题

- [ ] **Task 3.2** 验证 Vercel 部署链路
  - 在 frontend 子模块中 `git add && git commit`
  - 推送至远程仓库
  - 在 Vercel Dashboard 中导入项目，设置 Root Directory 为 `frontend`
  - **验证**: Vercel 自动部署成功，线上 URL 可访问，首页正常渲染

## Phase 4: 功能组件迁移（GREEN）

- [ ] **Task 4.1** 创建 API 层
  - 创建 `lib/api.ts`（从旧 `src/api/index.ts` 迁移 Axios 实例 + `getHealth` 函数）
  - **验证**: `pnpm type-check` 通过

- [ ] **Task 4.2** 创建 HealthChecker 组件
  - 创建 `components/HealthChecker.tsx`（`'use client'`）
  - 实现「检测后端连接」按钮（loading 状态 + 手动 spinner）
  - 实现后端状态 UP/DOWN 展示（用 Tailwind 替代 Element Plus 颜色指示）
  - **验证**: `pnpm dev` 点击按钮，后端在线时显示绿色 UP，离线时显示红色 DOWN

- [ ] **Task 4.3** 创建 Counter 组件
  - 创建 `components/Counter.tsx`（`'use client'`）
  - 实现计数器展示 + `+1` 按钮（用 Tailwind 按钮替代 `<el-button type="success">`）
  - **验证**: `pnpm dev` 点击 `+1`，计数递增，UI 实时更新

- [ ] **Task 4.4** 集成到首页
  - 更新 `app/page.tsx`，引入 `HealthChecker` 和 `Counter`
  - 实现「显示/隐藏计数器」切换（`showCounter` 状态在 HealthChecker 中管理）
  - **验证**: 首页功能与旧 Vue 版本行为完全一致

## Phase 5: 代码质量验证

- [ ] **Task 5.1** TypeScript 类型检查
  - `pnpm type-check` 无错误
  - **验证**: 控制台无类型错误输出

- [ ] **Task 5.2** ESLint 检查
  - `pnpm lint` 无错误
  - **验证**: 控制台无 lint 错误输出

- [ ] **Task 5.3** 构建产物验证
  - `pnpm build` 构建成功
  - 检查 `.next/` 目录包含构建产物
  - **验证**: 构建无错误，无警告

- [ ] **Task 5.4** 清理残留
  - 删除 `.npmrc` 中 Vue 相关配置（如有）
  - 删除 `.prettierrc` 中 Vue 相关格式化规则（如有）
  - 更新 `.gitignore`：移除 `dist/`（Vite 产物），保留 `node_modules/`、`.next/`
  - **验证**: `git status` 无 Vue 相关残留文件

## Phase 6: 规范同步

- [ ] **Task 6.1** 更新 `openspec/specs/todo-module.md`
  - 将前端验收标准中的 Vue 组件名更新为 React 组件名：
    - `TodoView.vue` → `app/todos/page.tsx`
    - `TodoForm.vue` → `components/todo/TodoForm.tsx`
    - `TodoList.vue` → `components/todo/TodoList.tsx`
    - `TodoItem.vue` → `components/todo/TodoItem.tsx`
  - 将「Element Plus 消息提示」改为「自定义 Toast 或 Tailwind 消息提示」
  - **验证**: spec 文件中无 Vue/Element Plus/Pinia/Vite 引用

- [ ] **Task 6.2** 更新 `openspec/changes/todo-module-crud/design.md`
  - 前端架构部分改为 Next.js App Router 方案
  - 更新组件树图（Vue SFC → React TSX）
  - 更新数据流图（ref/reactive → useState/useEffect）
  - **验证**: design.md 中无 Vue 专属 API 引用

## Phase 7: 最终验证

- [ ] **Task 7.1** 全链路联调
  - 同时启动 `backend:8080` + `frontend:3000`
  - 访问首页 → 检测后端连接 → 状态显示 UP
  - 计数器 +1 正常，显示/隐藏切换正常
  - **验证**: 前后端数据一致，浏览器控制台无错误

- [ ] **Task 7.2** 验收标准逐项确认
  - 对照 proposal.md 验收标准 checklist 逐项打勾
  - **验证**: 所有 checklist 项通过
