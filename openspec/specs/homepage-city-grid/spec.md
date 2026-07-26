# Spec: Homepage — 城市快速入口

## Purpose

本规范定义 WanderChina 首页 8 个 MVP 城市的图标网格入口。纯前端静态组件，无 API 依赖，城市列表为编译期常量。CityGrid 容器为 Server Component，CityItem 单项为 Client Component（hover + SVG onError fallback）。

技术基线：React 19 + Next.js 16 App Router + Tailwind CSS 4 + lucide-react。

---

## Requirements

### Requirement: 8 城网格渲染

The CityGrid SHALL render 8 MVP cities in a responsive CSS Grid: each cell displays a 32×32 SVG landmark icon in a circular container and the English city name below. Order is fixed: Beijing, Shanghai, Xi'an, Chengdu, Guangzhou, Guilin, Lijiang, Hangzhou.

#### Scenario: 城市网格完整渲染

- **WHEN** CityGrid 组件挂载
- **THEN** 展示标题 "🌆 Explore by City"（`text-2xl font-bold`）
- **AND** 8 个城市按 CSS Grid 排列：桌面端/平板端 4 列（`grid-cols-4`），移动端 3 列（`grid-cols-3`）
- **AND** 每个城市项包含：圆形背景容器（`bg-gray-100`，`w-16 h-16`）、SVG 图标居中（32×32，`text-gray-600`）、英文名称（`text-sm font-medium`，居中）
- **AND** 城市顺序固定：Beijing → Shanghai → Xi'an → Chengdu → Guangzhou → Guilin → Lijiang → Hangzhou

#### Scenario: SVG 图标缺失 fallback

- **WHEN** 城市 SVG 图标文件不存在（如图标 `<img>` onError 触发）
- **THEN** 显示 fallback 首字母圆形占位图：`bg-blue-500` 圆形 + 白色首字母文字（如 "B" for Beijing）
- **AND** 城市名称仍正常显示，控制台无未捕获异常

### Requirement: 城市点击导航

Each city item SHALL be a Next.js `<Link>` navigating to `/city/{id}` via client-side routing. Clicking the same city repeatedly SHALL work without errors.

#### Scenario: 城市点击跳转

- **WHEN** 用户点击 Beijing 图标
- **THEN** 导航到 `/city/beijing`（客户端路由），其余城市同理映射到对应 `/city/{id}`

#### Scenario: 目标页面 404

- **WHEN** 用户跳转到尚未实现的城市路由（如 `/city/beijing` 返回 404）
- **THEN** Next.js 显示默认 404 页面，浏览器后退按钮正常返回首页

#### Scenario: 重复点击同一城市

- **WHEN** 用户已在 `/city/beijing` 页面，通过浏览器返回首页后再次点击 Beijing
- **THEN** 正常导航到 `/city/beijing`，无异常

### Requirement: hover 交互

On desktop, hovering a city item SHALL scale the icon to 1.1×, change background to `bg-blue-50`, and change city name color to `text-blue-600` (200ms ease-out transition).

#### Scenario: 桌面端 hover

- **WHEN** 桌面端鼠标悬停在城市项上
- **THEN** 图标 `scale(1.1)`，背景从 `bg-gray-100` 变为 `bg-blue-50`
- **AND** 名称颜色从 `text-gray-700` 变为 `text-blue-600`
- **AND** 过渡 `duration-200 ease-out`，鼠标离开后 200ms 内恢复

### Requirement: 可访问性

City links SHALL have descriptive `aria-label`, SVGs SHALL have `role="img"` and `aria-hidden="true"`, and all items SHALL be keyboard-focusable with visible ring.

#### Scenario: 辅助技术兼容

- **WHEN** 屏幕阅读器解析城市网格
- **THEN** 每项 `<Link>` 有 `aria-label="Explore {nameEn} city guide"`
- **AND** SVG 图标有 `role="img"` 和 `aria-hidden="true"`（名称已说明）
- **AND** 首字母 fallback 有 `aria-label="{nameEn} icon"`
- **AND** 8 个城市链接均可键盘 Tab 聚焦，焦点显示 `ring-2 ring-blue-500`
