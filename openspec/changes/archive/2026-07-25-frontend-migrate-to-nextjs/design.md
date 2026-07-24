# Design: frontend-migrate-to-nextjs

## 概述

将 `frontend/` 子模块从 Vue 3 SPA 架构完整迁移到 Next.js 15 App Router 架构。保持与后端 Spring Boot REST API 的通信方式不变（HTTP/JSON + `/api` 代理），仅变更前端框架层。

### 核心设计原则

1. **一次性替换，不留混用期**：删除所有 Vue 文件后再创建 Next.js 文件，避免 `frontend/` 同时存在两套框架
2. **功能等价迁移**：现有 3 个功能点（首页展示、健康检测、计数器）在 Next.js 中行为完全一致
3. **Server Components 优先**：默认使用 React Server Components，仅在需要交互时添加 `'use client'`
4. **YAGNI**：不引入状态管理库（Zustand），计数器用 `useState` 足够

---

## 组件映射方案

### 现有 Vue 组件 → Next.js React 组件

| Vue 组件（旧） | React 组件（新） | 映射说明 |
|---|---|---|
| `App.vue` | `app/layout.tsx` | 根布局：`<html>` + `<body>` + Tailwind 全局样式引入 |
| `views/HomeView.vue` | `app/page.tsx` | 首页：Server Component 静态内容 + Client Component 交互部分 |
| `components/HelloWorld.vue` | `components/Counter.tsx` | 计数器组件：`'use client'`，使用 `useState` 替代 Pinia store |

### 组件拆解：HomeView.vue → app/page.tsx

原 `HomeView.vue` 包含 4 个区域：

```
┌─────────────────────────────────┐
│  标题 "My Third Project"         │  ← 纯静态，Server Component
│  副标题 "全栈开发骨架"            │
├─────────────────────────────────┤
│  [检测后端连接] [显示/隐藏计数器] │  ← 交互按钮，Client Component
│  后端状态: UP / DOWN             │  ← 动态数据，Client Component
├─────────────────────────────────┤
│  计数器: 0  [+1]                 │  ← 交互 + 状态，Client Component
└─────────────────────────────────┘
```

**拆分策略**：

```tsx
// app/page.tsx — Server Component（静态内容）
export default function HomePage() {
  return (
    <main className="min-h-screen bg-gray-50 flex items-center justify-center">
      <div className="text-center space-y-6">
        <h1 className="text-3xl font-bold text-gray-800">My Third Project</h1>
        <p className="text-gray-500">全栈开发骨架</p>
        <HealthChecker />   {/* Client Component：健康检测 */}
        <Counter />          {/* Client Component：计数器 */}
      </div>
    </main>
  )
}
```

```tsx
// components/HealthChecker.tsx — Client Component
'use client'
// 交互逻辑：loading、healthStatus、showCounter
```

```tsx
// components/Counter.tsx — Client Component
'use client'
// 状态：count，操作：increment
```

### Vue 语法 → React 语法对照

| Vue 3 (`<script setup>`) | React (TSX) |
|---|---|
| `ref(false)` | `useState(false)` |
| `ref('')` | `useState('')` |
| `loading.value = true` | `setLoading(true)` |
| `v-if="condition"` | `{condition && <Component />}` |
| `v-for="item in list"` | `{list.map(item => <Component key={item.id} />)}` |
| `@click="handler"` | `onClick={handler}` |
| `:class="{ 'text-green': ok }"` | `className={ok ? 'text-green-600' : 'text-red-600'}` |
| `:loading="loading"` | `disabled={loading}` + 手动渲染 spinner |
| `const { count } = storeToRefs(store)` | `const [count, setCount] = useState(0)` |
| `watch(showCounter, ...)` | 不需要 watch，直接条件渲染 |
| `onMounted(() => { ... })` | `useEffect(() => { ... }, [])` |

### Element Plus 组件替代方案

| Element Plus 组件 | React 替代方案 | 实现方式 |
|---|---|---|
| `<el-button type="primary">` | `<button>` + Tailwind | `className="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 disabled:opacity-50"` |
| `<el-button type="success" size="small">` | `<button>` + Tailwind | `className="px-3 py-1 bg-green-500 text-white rounded text-sm"` |

> 不再引入 Headless UI。当前页面仅用到按钮元素，Tailwind 手写足够。后续 todo-module 涉及复杂表单/对话框时再按需引入。

---

## 路由重构策略

### Vue Router → Next.js App Router

| Vue Router 概念 | Next.js App Router 等价物 |
|---|---|
| `createRouter({ routes: [...] })` | `app/` 目录下的文件即路由 |
| `{ path: '/', component: HomeView }` | `app/page.tsx` |
| `{ path: '/todos', component: TodoView }` | `app/todos/page.tsx` |
| `{ path: '/todos/:id', component: TodoDetail }` | `app/todos/[id]/page.tsx` |
| `<router-view />` | `{children}` in `layout.tsx` |
| 导航守卫 `beforeEach` | Next.js Middleware `middleware.ts` |
| 动态导入 `() => import(...)` | `next/dynamic` 或自动代码分割 |
| `createWebHistory()` | 内置 History API |

### 路由映射

```
旧（Vue Router）                    新（Next.js App Router）
─────────────────────────────────────────────────────────────
/                   → HomeView      app/page.tsx
/todos              → TodoView      app/todos/page.tsx
/todos/:id          → TodoDetail    app/todos/[id]/page.tsx
```

### 根布局设计

```tsx
// app/layout.tsx — 所有页面的根布局
import type { Metadata } from 'next'
import './globals.css'

export const metadata: Metadata = {
  title: 'My Third Project',
  description: '全栈开发骨架',
}

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="zh-CN">
      <body>{children}</body>
    </html>
  )
}
```

### API 代理配置

Next.js 通过 `next.config.ts` 的 `rewrites` 实现 API 代理，替代 Vite `server.proxy`：

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: 'http://localhost:8080/api/:path*',
      },
    ]
  },
}

export default nextConfig
```

---

## 状态管理迁移

### Pinia Store → React 原生状态

**现有 Pinia Store**（`stores/counter.ts`）：
```ts
defineStore('counter', () => {
  const count = ref(0)
  function increment() { count.value++ }
  return { count, increment }
})
```

**迁移后**：计数器是局部状态，不需要全局状态管理。直接在 `Counter.tsx` 中用 `useState`：

```tsx
'use client'
import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)
  return (
    <div className="p-4 bg-white rounded-lg shadow-sm border border-gray-200">
      <p className="text-gray-700 mb-2">
        计数器: <span className="font-mono font-bold text-lg">{count}</span>
      </p>
      <button
        onClick={() => setCount(c => c + 1)}
        className="px-3 py-1 bg-green-500 text-white rounded text-sm"
      >
        +1
      </button>
    </div>
  )
}
```

> **YAGNI 原则**：当前不需要全局状态。后续 todo-module 如涉及跨组件共享，再评估 `React Context + useReducer` 或引入 Zustand。

---

## 样式适配

### Tailwind CSS 4 在 Next.js 中的配置

Next.js 15 默认支持 Tailwind CSS，创建项目时选择 Tailwind 即可。对于 Vite 项目中的 `@tailwindcss/vite` 插件，Next.js 使用 `@tailwindcss/postcss` 插件：

**旧（Vite）**：
```ts
// vite.config.ts
import tailwindcss from '@tailwindcss/vite'
plugins: [vue(), tailwindcss()]
```

**新（Next.js）**：
```css
/* app/globals.css */
@import "tailwindcss";
```

> 不再需要 `postcss.config.mjs`，Tailwind CSS 4 通过 CSS `@import` 引入，Next.js 内置支持。

### 样式文件迁移

| 旧文件 | 新文件 | 说明 |
|---|---|---|
| `src/style.css` | `app/globals.css` | `@import "tailwindcss"` + 全局样式，在 `layout.tsx` 中引入 |
| 组件内 `<style scoped>` | Tailwind className | Vue scoped style 全部转换为 Tailwind 工具类 |

---

## API 层适配

### Axios 实例保持不变，仅调整引入方式

```ts
// lib/api.ts（原 src/api/index.ts）
import axios from 'axios'

const http = axios.create({
  baseURL: '/api',
})

export interface HealthResponse {
  code: number
  message: string
  data: {
    status: string
    timestamp: string
  }
}

export function getHealth() {
  return http.get<HealthResponse>('/health')
}
```

> 路径从 `@/api`（alias）改为 `@/lib/api`（Next.js 默认无 `@` alias，保持一致性）。

---

## 构建与工具链变更

### package.json 依赖对比

| 类别 | 旧（Vue 3） | 新（Next.js） |
|---|---|---|
| **框架** | `vue`、`vue-router`、`pinia` | `next`、`react`、`react-dom` |
| **UI 库** | `element-plus` | —（Tailwind 手写） |
| **HTTP** | `axios` | `axios`（保留） |
| **构建** | `vite`、`@vitejs/plugin-vue` | Next.js 内置 |
| **类型检查** | `vue-tsc` | `typescript` + `tsc --noEmit` |
| **代码检查** | `eslint`、`eslint-plugin-vue` | `eslint`、`eslint-config-next` |
| **样式** | `tailwindcss`、`@tailwindcss/vite` | `tailwindcss`、`@tailwindcss/postcss` |
| **格式化** | `prettier` | `prettier`（保留） |
| **类型** | — | `@types/react`、`@types/react-dom`、`@types/node` |

### scripts 变更

| 命令 | 旧 | 新 |
|---|---|---|
| 开发 | `vite` | `next dev` |
| 构建 | `vue-tsc -b && vite build` | `next build` |
| 启动 | `vite preview` | `next start` |
| 代码检查 | `eslint .` | `next lint` |
| 类型检查 | （合并到 build） | `tsc --noEmit` |

### tsconfig 关键配置

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

---

## 目录结构对比

### 旧结构（Vue 3）

```
frontend/
├── src/
│   ├── api/index.ts              # Axios 实例
│   ├── components/HelloWorld.vue # 计数器
│   ├── router/index.ts           # Vue Router
│   ├── stores/counter.ts         # Pinia Store
│   ├── views/HomeView.vue        # 首页
│   ├── App.vue                   # 根组件
│   ├── main.ts                   # 入口
│   ├── style.css                 # 全局样式
│   └── vite-env.d.ts             # Vite 类型
├── index.html                    # HTML 入口
├── vite.config.ts                # Vite 配置
├── package.json
└── tsconfig.json
```

### 新结构（Next.js App Router）

```
frontend/
├── app/
│   ├── layout.tsx                # 根布局（替代 App.vue + index.html）
│   ├── page.tsx                  # 首页（替代 HomeView.vue）
│   ├── globals.css               # 全局样式（替代 style.css）
│   └── favicon.ico               # 图标
├── components/
│   ├── HealthChecker.tsx         # 健康检测（Client Component）
│   └── Counter.tsx               # 计数器（Client Component）
├── lib/
│   └── api.ts                    # Axios 实例（替代 api/index.ts）
├── public/                       # 静态资源
├── next.config.ts                # Next.js 配置（替代 vite.config.ts）
├── package.json
├── tsconfig.json
├── eslint.config.mjs             # ESLint 配置（Next.js 默认）
└── postcss.config.mjs            # PostCSS 配置
```

### 关键变化

| 变化 | 说明 |
|------|------|
| `src/` → `app/` | Next.js App Router 约定目录，不再嵌套 `src/` |
| 无 `main.ts` | Next.js 自动通过 `layout.tsx` + `page.tsx` 拼装 |
| 无 `index.html` | `layout.tsx` 中的 `<html>` 标签替代 |
| 无 `router/`、`stores/` | 路由由文件系统定义，状态用 React 原生 API |
| 无 `.vue` 文件 | 全部使用 `.tsx` |

---

## 部署配置

### Vercel 部署（推荐）

Next.js 在 Vercel 上**零配置部署**，无需编写 `vercel.json`：

1. 在 Vercel 中导入 Git 仓库
2. 设置 Root Directory 为 `frontend`
3. 框架自动检测为 Next.js
4. 环境变量通过 Vercel Dashboard 配置
5. `git push` 自动触发部署

### Docker 部署（备选）

参考 Next.js 官方 Dockerfile（`standalone` 模式），镜像体积约 150MB。

---

## 不包含（YAGNI）

- ❌ 不引入 Radix UI / Headless UI / shadcn/ui（当前页面简单，Tailwind 手写足够）
- ❌ 不引入 Zustand / Jotai（无跨组件状态共享需求）
- ❌ 不配置国际化（i18n）
- ❌ 不引入 React Query / SWR（当前仅有简单 GET 请求，axios 直接调用即可）
- ❌ 不配置 PWA
- ❌ 不迁移测试（`HealthControllerTest.ts` 测试的是后端，前端暂无测试）
