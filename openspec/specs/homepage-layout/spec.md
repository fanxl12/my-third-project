# Spec: Homepage — 路由与布局容器

## Purpose

本规范定义 WanderChina 首页的路由注册 (`/`)、布局容器和各区块插槽编排，为其余 6 个单元提供挂载点。Layout 作为纯 Server Component，负责 SEO metadata 输出、6 个 `<section>` 插槽按序编排、统一间距和响应式容器约束。

技术基线：React 19 + Next.js 16 App Router（Server Component）+ Tailwind CSS 4 + shadcn/ui。

---

## Requirements

### Requirement: 首页路由注册

The app SHALL register `app/page.tsx` as the root route (`/`), exporting a Server Component (no `'use client'` directive) and a static `metadata` object for SEO.

#### Scenario: 用户访问根路径

- **WHEN** 用户访问 `/`
- **THEN** 页面返回 HTTP 200，渲染 HomePage 组件
- **AND** `<title>` 标签为 "WanderChina — Explore China, One City at a Time"
- **AND** `<meta name="description">` 存在且非空
- **AND** Open Graph 标签（`og:title`、`og:description`、`og:image`、`og:type`）均存在
- **AND** `next build` 编译成功，无 TypeScript 错误

#### Scenario: 搜索引擎爬虫抓取首页

- **WHEN** 搜索引擎爬虫访问 `/`
- **THEN** `<head>` 包含完整结构化 SEO metadata
- **AND** `og:image` 指向 `/images/og-homepage.jpg`（1200×630px）

### Requirement: 6 个区块插槽按序编排

The HomePage component SHALL render six content slots in fixed order: ① Hero (full-width) → ② NavCards → ③ HotPosts → ④ HotAttractions → ⑤ CityGrid → ⑥ AiAssistantEntry (fixed positioning). Slots ②–⑤ SHALL be wrapped in a centered container with `max-w-7xl mx-auto px-4`.

#### Scenario: 首页完整渲染

- **WHEN** 所有子组件已就绪，用户访问 `/`
- **THEN** 按顺序渲染 6 个 `<section>` 区块
- **AND** 插槽 ②–⑤ 在 `<div className="mx-auto max-w-7xl px-4">` 内水平居中
- **AND** 插槽 ①（Hero）无 `max-w-7xl` 容器约束，全宽渲染
- **AND** 插槽 ⑥（AiAssistantEntry）在 `<main>` 末尾，使用 `fixed` 定位
- **AND** 每个 `<section>` 的 `id` 属性与契约表一致：`hero`、`nav-cards`、`hot-posts`、`hot-attractions`、`city-grid`

#### Scenario: 缺失的子组件应触发编译期错误

- **WHEN** 任一子组件文件不存在（如 `@/components/homepage/hot-attractions` 不可解析）
- **THEN** `next build` 在编译阶段报错（TypeScript/import resolution 错误）
- **AND** 不会在运行时静默降级为空白区块

### Requirement: 统一区块间距

Content slots ②–⑤ SHALL use unified vertical spacing controlled by a shared constant `SECTION_GAP = 'py-20 md:py-12'`. Slot ① (Hero) SHALL manage its own height via `min-h-screen`.

#### Scenario: 桌面端间距

- **WHEN** 视口宽度 ≥ 1024px
- **THEN** 插槽 ②–⑤ 的 `<section>` 应用 `py-20`（上下各 80px padding）
- **AND** 插槽 ①（Hero）无间距，自身控制 `min-h-screen`

#### Scenario: 移动端间距

- **WHEN** 视口宽度 < 768px
- **THEN** 插槽 ②–⑤ 的 `<section>` 应用 `py-12`（上下各 48px padding）
- **AND** 页面无横向溢出滚动条
- **AND** 水平 padding 为 `px-4`（16px）

#### Scenario: 平板端过渡

- **WHEN** 视口宽度 768–1023px
- **THEN** 插槽 ②–⑤ 间距通过 `md:py-12` 过渡为移动端间距

### Requirement: 子组件独立加载不相互阻塞

Each content slot SHALL render independently. A slow-loading or errored child component SHALL NOT block rendering of other slots. The Hero section SHALL trigger First Contentful Paint first since it has no network dependency.

#### Scenario: HotPosts 延迟加载不阻塞其他区块

- **WHEN** HotPosts 内部 mock API 延迟 800ms 仍在 loading 状态
- **THEN** HeroSection 已完整渲染
- **AND** NavCards 已完整渲染
- **AND** HotPosts 显示自身的 Skeleton 占位
- **AND** 页面整体不白屏
- **AND** 浏览器 FCP 由 HeroSection 最先触发

#### Scenario: 任一子组件运行时抛出异常

- **WHEN** HeroSection 渲染时抛出未捕获异常
- **THEN** 建议为每个插槽包裹 `React.Suspense` + ErrorBoundary
- **AND** 若无 ErrorBoundary，整个页面显示 Next.js 默认错误页
- **AND** 下一阶段为 layout 引入 ErrorBoundary 缓解此问题

### Requirement: 子组件占位容错（开发阶段）

During development, when a child component is intentionally replaced with a placeholder, the layout SHALL gracefully render a dashed-border mock div without affecting other slots.

#### Scenario: 暂未实现的组件使用占位替代

- **WHEN** 开发阶段将 HotAttractions 替换为占位 div
- **THEN** 对应插槽渲染 `className="rounded-lg border-2 border-dashed border-gray-300 p-12 text-center text-gray-400"` 的占位容器
- **AND** 占位文本为 "Mock: HotAttractions"
- **AND** 其余 5 个插槽正常渲染
- **AND** 页面无编译或运行时错误

### Requirement: 极窄视口兼容

The layout SHALL support viewport widths down to 320px without horizontal overflow or text truncation.

#### Scenario: 320px 视口宽度

- **WHEN** 视口宽度为 320px
- **THEN** `px-4`（16px 水平 padding）确保内容不贴边
- **AND** 无横向溢出滚动条
- **AND** 所有文字可读，不被截断

---

## 插槽契约表（非规范内容，实现参考）

| 编号 | 组件 | section id | 容器 | 桌面间距 | 移动间距 | 备注 |
|------|------|------------|------|----------|----------|------|
| 1 | `HeroSection` | `hero` | 无（全宽） | — | — | 自身控制 min-h-screen |
| 2 | `NavCards` | `nav-cards` | max-w-7xl | `py-20` | `py-12` | — |
| 3 | `HotPosts` | `hot-posts` | max-w-7xl | `py-20` | `py-12` | — |
| 4 | `HotAttractions` | `hot-attractions` | max-w-7xl | `py-20` | `py-12` | — |
| 5 | `CityGrid` | `city-grid` | max-w-7xl | `py-20` | `py-12` | — |
| 6 | `AiAssistantEntry` | — | fixed | — | — | `bottom-6 right-6` |

## 代码骨架（非规范内容，实现参考）

```typescript
// ============================================================
// frontend/app/page.tsx
// ============================================================
import type { Metadata } from 'next'
import { HeroSection } from '@/components/homepage/hero'
import { NavCards } from '@/components/homepage/nav-cards'
import { HotPosts } from '@/components/homepage/hot-posts'
import { HotAttractions } from '@/components/homepage/hot-attractions'
import { CityGrid } from '@/components/homepage/city-grid'
import { AiAssistantEntry } from '@/components/homepage/ai-entry'

export const metadata: Metadata = {
  title: 'WanderChina — Explore China, One City at a Time',
  description:
    'WanderChina helps foreign travelers explore China with curated attraction guides, real traveler tips, and instant AI assistance.',
  openGraph: {
    title: 'WanderChina — Explore China, One City at a Time',
    description:
      'Curated attraction guides, real traveler tips, and instant AI assistance for your China adventure.',
    images: ['/images/og-homepage.jpg'],
    type: 'website',
  },
}

const SECTION_GAP = 'py-20 md:py-12'

export default function HomePage() {
  return (
    <main>
      <section id="hero">
        <HeroSection />
      </section>
      <div className="mx-auto max-w-7xl px-4">
        <section id="nav-cards" className={SECTION_GAP}>
          <NavCards />
        </section>
        <section id="hot-posts" className={SECTION_GAP}>
          <HotPosts />
        </section>
        <section id="hot-attractions" className={SECTION_GAP}>
          <HotAttractions />
        </section>
        <section id="city-grid" className={SECTION_GAP}>
          <CityGrid />
        </section>
      </div>
      <AiAssistantEntry />
    </main>
  )
}
```

## 实现文件清单（非规范内容，实现参考）

```
frontend/app/
├── page.tsx                     # 首页路由（原文件替换）
├── layout.tsx                   # 全局 RootLayout（已有，更新 metadata title）
└── globals.css                  # 全局样式（无需改动）

frontend/public/images/
└── og-homepage.jpg              # Open Graph 社交分享图（1200×630px）
```

无新增依赖。


