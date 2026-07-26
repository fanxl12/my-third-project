# Spec Delta: Homepage — 路由与布局容器

> 本 delta 对应 [homepage-layout spec](../../../specs/homepage-layout/spec.md) 的实现。此变更交付布局骨架、SEO metadata 和 6 个占位组件。

---

## ADDED Requirements

### Requirement: HomePage Server Component 实现

The system SHALL implement `app/page.tsx` as a Server Component (no `'use client'` directive) rendering six content slots in fixed order: Hero → NavCards → HotPosts → HotAttractions → CityGrid → AiAssistantEntry. Slots ②–⑤ SHALL be wrapped in `<div className="mx-auto max-w-7xl px-4">`. A shared constant `SECTION_GAP = 'py-20 md:py-12'` SHALL control vertical spacing.

#### Scenario: 首页编译通过

- **WHEN** 运行 `pnpm build`
- **THEN** 编译成功，无 TypeScript 错误
- **AND** page.tsx 导出 HomePage 默认函数组件和 metadata 对象

#### Scenario: 6 个区块按序渲染

- **WHEN** 访问 `http://localhost:3000`
- **THEN** 页面按序渲染 6 个 `<section>` 区块：`hero`、`nav-cards`、`hot-posts`、`hot-attractions`、`city-grid`
- **AND** AiAssistantEntry 在 `<main>` 末尾渲染
- **AND** Hero 区全宽（无 `max-w-7xl` 容器）
- **AND** 插槽 ②–⑤ 在 `max-w-7xl mx-auto px-4` 内水平居中

### Requirement: SEO Metadata 更新

The system SHALL update `app/layout.tsx` metadata template and `app/page.tsx` metadata to output WanderChina brand title, description, and Open Graph tags.

#### Scenario: 搜索引擎爬虫抓取

- **WHEN** 搜索引擎爬虫访问 `/`
- **THEN** `<title>` 为 "WanderChina — Explore China, One City at a Time"
- **AND** `<meta name="description">` 存在且非空
- **AND** Open Graph 标签（`og:title`、`og:description`、`og:image`、`og:type`）均存在

### Requirement: 6 个占位组件

The system SHALL create six placeholder components under `components/homepage/` (hero.tsx, nav-cards.tsx, hot-posts.tsx, hot-attractions.tsx, city-grid.tsx, ai-entry.tsx), each rendering a dashed-border mock div with the component name. Placeholders SHALL not block rendering of other slots.

#### Scenario: 占位组件渲染

- **WHEN** 开发阶段访问首页
- **THEN** 每个占位组件渲染 `className="rounded-lg border-2 border-dashed border-gray-300 p-12 text-center text-gray-400"` 的容器
- **AND** 占位文本为 "Mock: <ComponentName>"
- **AND** 全部 6 个占位块可见，无编译或运行时错误

#### Scenario: 占位组件独立不相互阻塞

- **WHEN** 任一占位组件渲染失败
- **THEN** 其余 5 个占位组件正常渲染
- **AND** 失败组件对应的 `<section>` 插槽不显示内容（React 不渲染抛出异常的组件）
