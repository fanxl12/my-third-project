# Design: homepage-hero

## Context

替换 hero.tsx 占位组件为 HeroSection Client Component。无 API 依赖，纯前端静态组件 + 搜索交互。

## Decisions

| 决策 | 选择 | 理由 |
|------|------|------|
| 组件类型 | Client Component（`'use client'`） | 搜索交互需要 useState + useRef |
| 背景图策略 | 使用 `bg-gradient-to-b from-blue-900 to-blue-700` 作为默认背景 | 尚无正式品牌背景图；该渐变满足 spec fallback 要求，后续可替换为 next/image |
| 搜索状态 | `useRef` 管理 input value + `useState` 管理提交禁用锁 | useRef 避免受控组件重渲染；禁用锁实现 debounce |
| toast | `sonner` 的 `toast` 函数直接调用 | shadcn/ui Sonner 组件已安装，`toast.message()` 即可 |
| 防抖策略 | 提交后 300ms 禁用按钮（disabled + opacity-50） | 比 setTimeout 防抖更直观，视觉反馈明确 |
| 字体 | 系统字体 `font-sans` | 已通过 layout.tsx 注入 Geist |

## 组件结构

```
components/homepage/hero.tsx
- HeroSection() — Client Component
  - Hero 容器：relative + min-h-screen + flex + overflow-hidden
  - 渐变背景层：absolute inset-0 + bg-gradient
  - 前景内容层：relative + flex flex-col + 居中
  - <h1> 品牌标语（响应式字号）
  - <p> 副标题
  - 搜索框区域（flex row，桌面端居中）
    - shadcn Input（ref 非受控）
    - shadcn Button + Search 图标
    - Sonner toast 调用
```

## 搜索行为逻辑

```
用户输入 → 按 Enter / 点击 Button
  ├─ 内容空/全空格 → 不触发，按钮保持 disabled
  ├─ 内容非空 → toast("Search is coming soon!")
  │   ├─ inputRef.current.value = ''（清空）
  │   └─ 按钮 disabled 300ms（防抖）
  └─ 快速点击 → 首次已清空 input，后续因空值忽略
```

## CSS 关键值

```tsx
// Hero 容器
className="relative min-h-screen flex flex-col items-center justify-center overflow-hidden bg-gradient-to-b from-blue-900 to-blue-700"

// 渐变遮罩（bottom-to-top）
<div className="absolute inset-0 bg-gradient-to-t from-black/60 via-black/30 to-transparent" />

// 前景内容（相对定位在上层）
className="relative z-10 flex flex-col items-center px-4"

// 标语
className="text-3xl md:text-5xl lg:text-6xl font-bold text-white text-center"

// 副标题
className="mt-4 text-lg lg:text-xl text-white/80 text-center max-w-2xl"

// 搜索框容器
className="mt-8 flex w-full max-w-[90vw] items-center gap-2 sm:max-w-xl lg:max-w-2xl"

// Input
<Input ref={inputRef} maxLength={200} aria-label="Search destinations and guides" />

// Button
<Button disabled={isSubmitting} onClick={handleSearch}>
  <Search className="h-4 w-4" aria-hidden="true" />
  Search
</Button>
```

## 关键样式

- 搜索框容器：移动端 90vw max 420px，平板 `max-w-xl`，桌面 `max-w-2xl`
- Hero 底部 padding：`pb-8 px-4`（搜索框距底部 32px）→ 通过 flex justify-center + pb-8 实现
- 背景渐变 bottom-to-top：`bg-gradient-to-t from-black/60 via-black/30 to-transparent`
