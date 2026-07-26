# Design: introduce-shadcn-ui

## 概述

为 WanderChina 前端工程引入 shadcn/ui + lucide-react 作为 UI 组件层，替代手写原生 HTML 表单控件，满足首页 7 个 spec 单元的组件契约。

## 现状分析

| 资产 | 状态 | 版本 |
|------|------|------|
| Tailwind CSS | ✅ 已安装 | v4.1.4 |
| `@tailwindcss/postcss` | ✅ 已安装 | v4.3.3 |
| `postcss.config.mjs` | ✅ 已配置 | — |
| `globals.css` | ✅ `@import "tailwindcss"` | — |
| shadcn/ui | ❌ 未初始化 | — |
| lucide-react | ❌ 未安装 | — |

## 技术方案

### 1. 安装顺序

```
Step 1: pnpm add lucide-react           # 图标库（与 shadcn 解耦）
Step 2: npx shadcn@latest init          # 初始化（生成 components.json + lib/utils.ts）
Step 3: npx shadcn@latest add input button sonner card skeleton badge
```

### 2. shadcn/ui 初始化参数

```bash
npx shadcn@latest init --defaults
```

预期生成的 `components.json`：

```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "default",
  "rsc": true,
  "tsx": true,
  "tailwind": {
    "config": "",
    "css": "app/globals.css",
    "baseColor": "slate",
    "cssVariables": true
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "ui": "@/components/ui",
    "lib": "@/lib",
    "hooks": "@/hooks"
  }
}
```

### 3. globals.css 更新

shadcn/ui 初始化会在 `globals.css` 中追加 CSS 变量声明。需确保不覆盖已有的 `@import "tailwindcss"` 语句。最终文件结构：

```css
@import "tailwindcss";

@theme inline {
  /* shadcn/ui 注入的 CSS 变量（--background, --foreground, --primary 等） */
}

/* base 层样式 */
```

### 4. lib/utils.ts

```typescript
import { clsx, type ClassValue } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

shadcn 依赖 `clsx` + `tailwind-merge`，`init` 时会自动安装。

### 5. 组件目录结构

```
frontend/components/ui/
├── button.tsx      # shadcn/ui Button
├── input.tsx       # shadcn/ui Input
├── sonner.tsx      # shadcn/ui Sonner (toast)
├── card.tsx        # shadcn/ui Card
├── skeleton.tsx    # shadcn/ui Skeleton
└── badge.tsx       # shadcn/ui Badge
```

每个组件文件 ≈ 30-80 行，可复制粘贴到项目中使用。

### 6. AGENTS.md 变更

#### 6.1 选型原则（第 100 行）

```diff
- 状态管理优先用 React Context/useReducer，避免引入第三方 UI 库（除非 spec 明确要求）
+ 状态管理优先用 React Context/useReducer；UI 组件使用 shadcn/ui + lucide-react 图标库
```

#### 6.2 项目结构表格（第 125 行）

```diff
- | `frontend` | `./frontend` | Next.js + React + TypeScript + Tailwind CSS 4 | SSR/SSG 前端应用 |
+ | `frontend` | `./frontend` | Next.js + React + TypeScript + Tailwind CSS 4 + shadcn/ui + lucide-react | SSR/SSG 前端应用 |
```

#### 6.3 禁止事项（第 143-150 行）

新增 3 条：

```markdown
- ❌ 不引入 shadcn/ui 和 lucide-react 之外的第三方 UI 库（如 MUI、Ant Design、Chakra UI）
- ❌ 不使用内联 `style={{}}` 对象（统一使用 Tailwind 类名或 shadcn/ui 组件 Props）
- ❌ 不自行实现已有的 shadcn/ui 组件功能（如 toast 应使用 Sonner 而非手写）
```

## 关键决策

| 决策 | 选择 | 理由 |
|------|------|------|
| shadcn/ui 版本 | `latest`（canary for Tailwind v4） | 与已安装的 Tailwind v4 兼容 |
| CSS 变量方案 | 使用 shadcn 默认 CSS variables（`@theme inline`） | Tailwind v4 原生支持，零配置覆盖 |
| 组件安装方式 | `npx shadcn@latest add` 逐组件安装 | YAGNI — 仅安装 spec 明确需要的 6 个组件 |
| Sonner 位置 | `sonner.tsx` → 使用 `next/dynamic` 动态导入 | 避免 SSR 时的 hydration mismatch |

## 备选方案评估

| 方案 | 优点 | 缺点 | 结论 |
|------|------|------|------|
| **shadcn/ui（推荐）** | 无包体积开销、完全控制源码、与 Tailwind 深度集成 | 需手动初始化 | ✅ 采用 |
| Radix UI 原语 | 更底层控制 | 无预置样式，需大量手写 CSS | ❌ 工作量过大 |
| Headless UI | Tailwind 团队维护 | 组件种类少（无 Skeleton/Badge/Sonner） | ❌ 不满足 spec 需求 |
| MUI / Ant Design | 组件齐全 | 庞大依赖、样式与 Tailwind 冲突、违背 AGENTS.md 轻量原则 | ❌ 违背选型约束 |
