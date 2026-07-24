# Spec: 前端子模块（Next.js + React）

> 本规范定义 `frontend/` 子模块的技术边界、目录约定、组件编码规范。
> 此规范为新版本，**完全替代**旧的 Vue 3 前端子模块规范。

---

## 1. 技术边界

| 属性 | 值 |
|------|-----|
| 框架 | Next.js 15（App Router） |
| UI 库 | React 19 |
| 语言 | TypeScript 5.x（strict 模式） |
| 样式方案 | Tailwind CSS 4 |
| HTTP 客户端 | Axios（仅客户端组件中使用） |
| 包管理器 | pnpm（Corepack 管理） |
| 部署平台 | Vercel（零配置） |
| 开发端口 | 3000 |
| API 代理 | Next.js rewrites → `http://localhost:8080` |

---

## 2. 目录约定

```
frontend/
├── app/                          # App Router 页面目录（文件即路由）
│   ├── layout.tsx                # 根布局（必须存在）
│   ├── page.tsx                  # 首页 /（必须存在）
│   ├── globals.css               # 全局样式 + Tailwind 入口
│   └── favicon.ico               # 网站图标
├── components/                   # 可复用组件
│   └── *.tsx                     # 组件文件（PascalCase）
├── lib/                          # 工具函数 / API 封装
│   └── api.ts                    # Axios 实例 + 后端接口函数
├── public/                       # 静态资源（图片、字体等）
├── next.config.ts                # Next.js 配置
├── tsconfig.json                 # TypeScript 配置
├── eslint.config.mjs             # ESLint 配置（eslint-config-next）
├── postcss.config.mjs            # PostCSS 配置
├── package.json
└── pnpm-lock.yaml
```

### 禁止项

- ❌ 无 `src/` 嵌套目录 — Next.js App Router 使用顶层 `app/` 作为路由根
- ❌ 无 `pages/` 目录 — 本项目使用 App Router，不使用 Pages Router
- ❌ 无 `.vue` 文件 — 所有 UI 使用 `.tsx`
- ❌ 无 `router/`、`stores/` 目录 — 路由由文件系统定义，状态用 React 原生 API

---

## 3. 组件编码规范

### 3.1 Server Component 优先

默认创建 **Server Component**（不加 `'use client'` 指令），仅在以下场景添加 `'use client'`：

- 使用 `useState`、`useEffect`、`useReducer` 等 React Hooks
- 监听浏览器事件（`onClick`、`onChange` 等）
- 使用浏览器 API（`localStorage`、`window`、`document`）

```tsx
// ✅ Server Component（默认）
// app/page.tsx
export default function HomePage() {
  // 可直接 async/await 获取数据
  return <main>...</main>
}
```

```tsx
// ✅ Client Component（按需标记）
// components/Counter.tsx
'use client'
import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>
}
```

### 3.2 组件拆分原则

- 交互部分提取为独立 Client Component，静态父组件保持为 Server Component
- 一个组件文件 ≤ 100 行，超过则拆分
- 组件文件命名使用 **PascalCase**（如 `HealthChecker.tsx`）

### 3.3 Props 类型定义

```tsx
// ✅ 内联类型（简单 Props）
export default function Counter({ initialCount = 0 }: { initialCount?: number }) {
  // ...
}

// ✅ interface（多 Props 或复用）
interface TodoItemProps {
  todo: TodoItem
  onToggle: (id: number) => void
  onDelete: (id: number) => void
}

export default function TodoCard({ todo, onToggle, onDelete }: TodoItemProps) {
  // ...
}
```

---

## 4. 路由约定

### 4.1 文件系统路由

| 文件路径 | URL | 说明 |
|----------|-----|------|
| `app/page.tsx` | `/` | 首页 |
| `app/todos/page.tsx` | `/todos` | 待办列表页 |
| `app/todos/[id]/page.tsx` | `/todos/123` | 待办详情页 |

### 4.2 新路由创建流程

1. 在 `app/` 下创建对应目录（如 `app/todos/`）
2. 创建 `page.tsx` 文件，默认导出 React 组件
3. 需要多个共享布局时，创建 `layout.tsx`

---

## 5. API 调用规范

### 5.1 Axios 实例（`lib/api.ts`）

```ts
// lib/api.ts
import axios from 'axios'

const http = axios.create({
  baseURL: '/api',
})

// 所有后端接口函数在此文件中导出
export function getHealth() {
  return http.get<ApiResponse<HealthData>>('/health')
}
```

### 5.2 调用位置

- **Server Component**：可直接 `fetch()` 或调用 axios（数据在服务端获取）
- **Client Component**：在 `useEffect` 或事件处理器中调用 axios

```tsx
// Client Component 中调用 API
'use client'
import { useState, useEffect } from 'react'
import { getHealth } from '@/lib/api'

export default function HealthChecker() {
  const [status, setStatus] = useState('')

  useEffect(() => {
    getHealth().then(res => setStatus(res.data.data.status))
  }, [])

  return <div>Status: {status}</div>
}
```

### 5.3 统一响应类型

```ts
// 与后端 ApiResponse<T> 对应
export interface ApiResponse<T> {
  code: number
  message: string
  data: T
}
```

---

## 6. 样式规范

### 6.1 Tailwind CSS 优先

- 全部样式使用 Tailwind 工具类，**不创建** `.css` / `.module.css` 文件（`globals.css` 除外）
- 复杂样式用 `@apply` 在 `globals.css` 中定义工具类
- 颜色使用 Tailwind 内置色板，不自定义颜色值

### 6.2 按钮样式约定

```tsx
// 主按钮
<button className="px-4 py-2 bg-blue-500 text-white rounded-lg
                   hover:bg-blue-600 disabled:opacity-50 transition-colors">
  提交
</button>

// 成功按钮
<button className="px-3 py-1 bg-green-500 text-white rounded text-sm
                   hover:bg-green-600 transition-colors">
  +1
</button>
```

---

## 7. 配置规范

### 7.1 Next.js 配置（`next.config.ts`）

```ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  // API 代理到后端 Spring Boot
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: 'http://localhost:8080/api/:path*',
      },
    ]
  },
  // Vercel 部署时忽略 ESLint/TypeScript 错误（已在 CI 中检查）
  eslint: { ignoreDuringBuilds: false },
  typescript: { ignoreBuildErrors: false },
}

export default nextConfig
```

### 7.2 TypeScript 配置约束

- `strict: true` — 严格模式必须开启
- `noEmit: true` — Next.js 自行处理编译，tsc 仅做类型检查
- `paths: { "@/*": ["./*"] }` — `@/` 别名指向 `frontend/` 根目录

---

## 8. 构建与部署

### 8.1 开发阶段

| 命令 | 用途 |
|------|------|
| `pnpm dev` | 启动开发服务器（端口 3000，热更新） |
| `pnpm type-check` | TypeScript 类型检查 |
| `pnpm lint` | ESLint 代码检查 |
| `pnpm format` | Prettier 格式化 |

### 8.2 生产阶段

| 命令 | 用途 |
|------|------|
| `pnpm build` | 生产构建（含类型检查 + ESLint） |
| `pnpm start` | 启动生产服务器 |

### 8.3 Vercel 部署

- 零配置：Vercel 自动识别 Next.js 框架
- Root Directory 设置为 `frontend`
- 环境变量通过 Vercel Dashboard 配置
- `git push` 到主分支自动触发部署

---

## 9. 与旧版（Vue 3）的差异摘要

| 层面 | 旧规范（Vue 3） | 新规范（Next.js） |
|------|-----------------|-------------------|
| 入口文件 | `src/main.ts` + `index.html` | `app/layout.tsx` + `app/page.tsx` |
| 路由定义 | `src/router/index.ts` 手动配置 | `app/` 文件系统自动路由 |
| 状态管理 | Pinia `defineStore` | React `useState` / `useReducer` / Context |
| 组件格式 | `.vue` SFC（template + script + style） | `.tsx`（JSX + Tailwind） |
| API 代理 | `vite.config.ts` server.proxy | `next.config.ts` rewrites |
| UI 库 | Element Plus | Tailwind CSS 4 手写 |
| 类型检查 | `vue-tsc` | `tsc --noEmit` |
| 部署 | Vite SPA + `vercel.json` | Vercel 原生零配置 |

---

## 10. 验收标准

- [ ] `pnpm dev` 启动成功，端口 3000 可访问
- [ ] `pnpm build` 构建成功，无 TypeScript 错误
- [ ] `pnpm lint` ESLint 通过
- [ ] `pnpm type-check` 类型检查通过
- [ ] Vercel 部署成功，线上页面可访问
- [ ] 通过 `/api/*` 路径可代理到后端 `localhost:8080`
- [ ] 目录结构符合 §2 约定
- [ ] 所有组件遵循 §3 编码规范（Server/Client Component 分界正确）
