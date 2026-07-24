# Tasks: init-vue3-frontend-scaffold

## Phase 1: 环境准备

- [ ] **Task 1.1** Corepack 启用 + pnpm 可用性验证
  - 执行 `corepack enable` 成功
  - `pnpm --version` 正常输出版本号

- [ ] **Task 1.2** Vite + Vue 3 + TypeScript 项目脚手架初始化
  - 执行 `pnpm create vite frontend --template vue-ts` 生成项目骨架
  - 确认 `package.json`、`tsconfig.json`、`vite.config.ts` 存在
  - **验证**: `pnpm install` 成功，`pnpm dev` 可看到 Vite 默认首页

## Phase 2: 依赖 + 配置

- [ ] **Task 2.1** 安装核心依赖
  - 安装 `vue-router`、`pinia`、`axios`
  - 安装 `element-plus`、`tailwindcss` + `@tailwindcss/vite`
  - 安装 `eslint`、`prettier`、`eslint-plugin-vue`、`@typescript-eslint/*`
  - **验证**: `pnpm install` 全部成功，`pnpm list --depth=0` 列出所有包

- [ ] **Task 2.2** Vite 配置
  - `vite.config.ts`: 添加代理 `/api` → `localhost:8080`
  - `vite.config.ts`: 添加 Element Plus 按需导入插件（可选，先全量导入以降低复杂度）
  - **验证**: `pnpm dev` 启动，`curl localhost:5173/api/health` 代理到后端返回 JSON

- [ ] **Task 2.3** Tailwind CSS 配置
  - `style.css`: 添加 `@tailwind base/components/utilities`
  - 导入 Element Plus 样式
  - **验证**: 页面中 tailwind class（如 `text-red-500`）生效

- [ ] **Task 2.4** ESLint + Prettier 配置
  - `eslint.config.js`: Vue + TS 规则
  - `.prettierrc`: 统一格式（单引号、无分号、2空格缩进）
  - `package.json` 添加 `format` / `lint` scripts
  - **验证**: `pnpm lint` 通过（无报错）

## Phase 3: 源码实现

- [ ] **Task 3.1** `src/main.ts` 应用入口
  - 创建 Vue app，挂载 Router + Pinia
  - 导入 Element Plus + Tailwind 样式
  - **验证**: 应用启动无报错

- [ ] **Task 3.2** `src/router/index.ts` 路由
  - 配置 `/` → HomeView
  - **验证**: 访问 `/` 渲染 HomeView

- [ ] **Task 3.3** `src/api/index.ts` Axios 实例
  - `baseURL: '/api'`，封装 `getHealth()` 方法
  - **验证**: 调用返回 `{ code: 200, data: { status: "UP" } }`

- [ ] **Task 3.4** `src/views/HomeView.vue` 首页
  - 展示后端健康状态（调用 `/api/health`）
  - 使用 Element Plus 按钮 + Tailwind 布局
  - **验证**: 页面显示 "后端状态: UP"

- [ ] **Task 3.5** `src/App.vue` 根组件
  - `<router-view>` 路由出口
  - **验证**: 路由切换正常

- [ ] **Task 3.6** `src/components/HelloWorld.vue` 示例组件
  - 简单计数器或问候组件，演示 Pinia store
  - **验证**: 计数器可交互

## Phase 4: 最终验证

- [ ] **Task 4.1** 全链路联调
  - 同时启动 `backend:8080` + `frontend:5173`
  - 访问 `localhost:5173`，页面正确展示后端健康状态
  - **验证**: 控制台无 CORS/网络错误

- [ ] **Task 4.2** 生产构建
  - `pnpm build` 成功，`dist/` 目录有产物
  - **验证**: `pnpm preview` 预览构建产物，首页正常
