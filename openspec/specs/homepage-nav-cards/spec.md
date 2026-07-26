# Spec: Homepage — 功能导航栏

## Purpose

本规范定义 WanderChina 首页三大平台入口卡片：Travel Community、Attraction Guides、AI Assistant。纯前端静态组件，无 API 依赖，卡片内容为编译期常量。NavCards 容器为 Server Component，单卡片 NavCard 为 Client Component（hover 动效需客户端交互）。

技术基线：React 19 + Next.js 16 App Router + Tailwind CSS 4 + shadcn/ui + lucide-react。

---

## Requirements

### Requirement: 三张导航卡片渲染

The NavCards section SHALL render three equal-width cards in a responsive grid: "Travel Community" (MessageCircle icon), "Attraction Guides" (MapPin icon), "AI Assistant" (Bot icon). Each card SHALL display a 32×32 blue icon, English title, and one-line description.

#### Scenario: 导航栏正常渲染

- **WHEN** 用户滚动到 Hero 区下方，功能导航栏进入视口
- **THEN** 展示三张等宽卡片：桌面端水平排列（`grid-cols-3`，间距 `gap-6`），移动端竖向堆叠（`gap-4`）
- **AND** 卡片顺序固定：Travel Community → Attraction Guides → AI Assistant
- **AND** 每张卡片包含：32×32 品牌蓝色图标（`text-blue-500`）、英文标题（`text-lg font-semibold`）、一句话描述（`text-sm text-gray-500`）
- **AND** 卡片外观统一：白色背景、圆角 `rounded-xl`、默认阴影 `shadow-md`、内边距 `p-8`

#### Scenario: 卡片 hover 交互（桌面端）

- **WHEN** 桌面端（≥768px）鼠标悬停在任意卡片上
- **THEN** 卡片向上浮起 4px（`translateY(-4px)`），阴影从 `shadow-md` 加深至 `shadow-lg`
- **AND** 过渡动画 `duration-200 ease-out`，鼠标离开后 200ms 内恢复原位

### Requirement: 卡片点击导航

Each card SHALL be wrapped in a Next.js `<Link>`, navigating to `/community`, `/attractions`, or `/ai-assistant` respectively via client-side routing (no full page reload).

#### Scenario: 点击卡片跳转

- **WHEN** 用户点击 "Travel Community" 卡片
- **THEN** Next.js `<Link>` 客户端路由跳转到 `/community`，不触发全页刷新
- **WHEN** 用户点击 "Attraction Guides" 卡片
- **THEN** 导航到 `/attractions`
- **WHEN** 用户点击 "AI Assistant" 卡片
- **THEN** 导航到 `/ai-assistant`

#### Scenario: 目标页面不存在

- **WHEN** 用户点击跳转至尚未实现的路由（如 `/community` 返回 404）
- **THEN** Next.js 显示默认 404 页面
- **AND** 浏览器后退按钮可返回首页，导航栏卡片不受影响

#### Scenario: JavaScript 禁用

- **WHEN** 浏览器禁用 JavaScript
- **THEN** 卡片以 `<a>` 标签渲染（Link 降级为原生链接），点击触发全页刷新导航（MPA 行为）

### Requirement: 响应式布局

Cards SHALL stack vertically on mobile (<768px) with full width and 16px gap, and display as 3 equal columns on tablet/desktop (≥768px) with 24px gap.

#### Scenario: 移动端

- **WHEN** 视口宽度 < 768px
- **THEN** 三张卡片竖向堆叠，每张宽度 100%，间距 16px（`gap-4`）
- **AND** 图标尺寸缩小至 28×28

#### Scenario: 平板/桌面端

- **WHEN** 视口宽度 ≥ 768px
- **THEN** 三张卡片水平等宽排列（`grid-cols-3`），间距 24px（`gap-6`）

### Requirement: 可访问性

Each card link SHALL have descriptive `aria-label`, icons SHALL be marked `aria-hidden="true"`, and cards SHALL be keyboard-navigable with visible focus ring.

#### Scenario: 辅助技术兼容

- **WHEN** 屏幕阅读器解析导航栏
- **THEN** 卡片 `<Link>` 有 `aria-label="Navigate to Travel Community"` 等描述
- **AND** 图标有 `aria-hidden="true"`（纯装饰性）
- **AND** 键盘 Tab 键可在三张卡片间切换焦点，焦点样式 `ring-2 ring-blue-500`
