# Spec: Homepage — 热门景点

## Purpose

本规范定义 WanderChina 首页热门景点卡片列表组件。展示 6–8 个按浏览量降序排列的景点，通过 mock API 获取数据，封面图使用 `next/image` 优化。HotAttractions 为 Client Component，使用 `useHotAttractions` Hook 管理四态。

技术基线：React 19 + Next.js 16 App Router + Tailwind CSS 4 + shadcn/ui + lucide-react + next/image。

---

## Requirements

### Requirement: 景点列表加载与 grid 渲染

The HotAttractions component SHALL fetch mock data on mount, displaying 4 Skeleton cards during loading, up to 8 attraction cards in responsive grid on success, and supporting empty/error states.

#### Scenario: 景点列表正常加载

- **WHEN** HotAttractions 组件挂载，mock API 在 600ms 后返回 8 条景点数据
- **THEN** 展示 4 个 Skeleton 卡片（2 行 × 2 列桌面端 grid）
- **AND** mock API resolve 后 Skeleton 消失，8 张景点卡片按 grid 展示
- **AND** 桌面端 4 列（`grid-cols-4`），平板 2 列，移动端 1 列
- **AND** 每张卡片包含：封面图（`next/image`，aspect-ratio 4/3，`placeholder="blur"`）、城市 Badge（叠加在封面左下角）、景点英文名称（`line-clamp-1`）、摘要（`text-sm text-gray-500`，`line-clamp-1`）
- **AND** 卡片按 `viewCount` 降序排列

#### Scenario: 景点列表为空

- **WHEN** mock API 返回空数组 `[]`
- **THEN** 展示空状态：lucide-react `ImageOff` 图标 + "No attractions available"

#### Scenario: 景点列表加载失败

- **WHEN** mock API 模拟 500 错误
- **THEN** 展示错误状态：lucide-react `AlertCircle` 图标 + "Failed to load attractions" + Retry 按钮
- **AND** 用户点击 Retry → 重新进入 loading → 重新请求

#### Scenario: 数据条目不足 6 条

- **WHEN** mock API 返回 3 条数据（少于 6 条）
- **THEN** 正常展示 3 张卡片，不强制补足，grid 列数不受影响

### Requirement: 景点卡片交互与 hover 效果

On desktop, hovering an attraction card SHALL lift it 4px with shadow increase and scale the cover image to 1.05× (200ms ease-out). Each card SHALL be a clickable `<Link>` to `/attractions/{id}`.

#### Scenario: 桌面端 hover

- **WHEN** 桌面端鼠标悬停在景点卡片上
- **THEN** 卡片 `translateY(-4px)` + `shadow-md → shadow-lg`，200ms ease-out
- **AND** 封面图 `scale(1.05)`（`overflow-hidden` 裁剪），200ms ease-out
- **AND** 鼠标离开后 200ms 内恢复

#### Scenario: 景点卡片点击

- **WHEN** 用户点击景点卡片
- **THEN** Next.js `<Link>` 导航到 `/attractions/{attractionId}`

#### Scenario: 封面图加载失败

- **WHEN** 封面图 URL 返回 404
- **THEN** 显示 fallback 占位图（灰色背景 + lucide-react `ImageOff` 图标居中）
- **AND** 卡片其余信息（名称、城市、摘要）正常展示

#### Scenario: 景点名称为空

- **WHEN** mock 数据中某景点 `nameEn` 为空字符串
- **THEN** 名称显示为 "Unknown Attraction"（fallback）

### Requirement: 封面图优化

Cover images SHALL use `next/image` with `fill` + `sizes` attribute, `placeholder="blur"`, and 4:3 aspect ratio. Images SHALL be JPEG ≤150KB with WebP alternative.

#### Scenario: 图片优化属性

- **WHEN** 景点卡片渲染封面图
- **THEN** 使用 `next/image`（`fill` + `sizes="(max-width: 768px) 100vw, (max-width: 1024px) 50vw, 25vw"`）
- **AND** 加载中显示 blur placeholder，加载完成后 fade-in 过渡
- **AND** 封面图在所有断点保持 4:3 宽高比（`aspect-[4/3]`）

### Requirement: 数据获取 Hook

The `useHotAttractions` hook SHALL manage four states, expose `retry()`, and cancel pending requests on unmount via AbortController.

#### Scenario: 快速重试

- **WHEN** mock API 持续返回错误，用户连续点击 Retry 按钮 5 次
- **THEN** 每次点击都重新触发请求（不防抖），组件状态在 loading/error 间正确切换
- **AND** 不出现内存泄漏或状态残留

#### Scenario: 组件卸载

- **WHEN** HotAttractions 组件在请求未完成前卸载
- **THEN** AbortController 取消请求，控制台无 "setState on unmounted" 警告

### Requirement: 可访问性

Card links SHALL have descriptive `aria-label`, cover images SHALL have meaningful `alt` text (not decorative), Skeleton SHALL have `aria-busy="true"`.

#### Scenario: 辅助技术兼容

- **WHEN** 屏幕阅读器解析热门景点区
- **THEN** 卡片 `<Link>` 有 `aria-label="View attraction: {nameEn}"`
- **AND** 封面图 `alt` 设置为景点名称（非装饰性，携带信息）
- **AND** Skeleton 有 `aria-busy="true"` 和 `aria-label="Loading attractions"`
