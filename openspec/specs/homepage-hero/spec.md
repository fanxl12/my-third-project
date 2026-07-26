# Spec: Homepage — Hero 区

## Purpose

本规范定义 WanderChina 首页首屏品牌展示区。Hero 区为纯前端静态组件，无 API 依赖，负责品牌主视觉背景图、品牌标语、副标题和搜索框交互。搜索框本期仅弹出 "Coming Soon" toast，真实搜索能力由后续 spec 承接。

技术基线：React 19 + Next.js 16 App Router + Tailwind CSS 4 + shadcn/ui + lucide-react。HeroSection 为 Client Component（含搜索交互状态），其余子元素通过 props 和常量驱动。

---

## Requirements

### Requirement: 品牌主视觉与文字展示

The Hero section SHALL display a full-viewport brand background image with gradient overlay, centered brand slogan, and subtitle. Text SHALL maintain WCAG AA 4.5:1 contrast ratio against the overlay.

#### Scenario: Hero 区首次渲染

- **WHEN** 用户访问首页 `/`
- **THEN** Hero 区占据 `min-h-screen`（100% 视口高度），宽度 100%
- **AND** 背景图使用 `next/image`（`fill` + `priority` 属性），LCP < 2.5s（本地开发环境）
- **AND** 背景图加载中展示 `placeholder="blur"` 模糊占位，加载完成后 300ms `opacity` 淡入过渡
- **AND** 背景图上方覆盖 bottom-to-top 渐变遮罩（`from-black/60 via-black/30 to-transparent`）
- **AND** 背景图 `alt=""`（装饰性图片，对屏幕阅读器隐藏）
- **AND** 品牌标语 `<h1>` 为页面内唯一的顶级标题

#### Scenario: 响应式标语字号

- **WHEN** 视口宽度 < 768px（移动端）
- **THEN** 标语字号为 `text-3xl`，副标题为 `text-lg`
- **WHEN** 视口宽度 768–1023px（平板端）
- **THEN** 标语字号为 `text-5xl`
- **WHEN** 视口宽度 ≥ 1024px（桌面端）
- **THEN** 标语字号为 `text-6xl`，副标题为 `text-xl`
- **AND** 标语颜色 `text-white`、font-weight bold；副标题 `text-white/80`

#### Scenario: 背景图加载失败

- **WHEN** 背景图 URL 返回 404 或网络超时
- **THEN** 显示 fallback 纯色背景 `bg-gradient-to-b from-blue-900 to-blue-700`
- **AND** 品牌标语和副标题仍然正常显示
- **AND** 控制台不出现未捕获异常

### Requirement: 搜索框交互

The Hero section SHALL include a pill-style search input (shadcn/ui `Input`) with a submit button (shadcn/ui `Button` + lucide-react `Search` icon). Submission SHALL pop a Sonner toast "Search is coming soon!" and clear the input. Empty or whitespace-only input SHALL not trigger submission.

#### Scenario: 搜索框正常提交

- **WHEN** 用户在搜索框中输入 "Great Wall" 并按 Enter 或点击搜索按钮
- **THEN** 弹出 Sonner toast "Search is coming soon!"，位置 `top-center`，持续 3 秒后自动消失
- **AND** 搜索框内容清空，焦点回到搜索框
- **AND** toast 可见期间再次提交时，新 toast 替换旧 toast（不堆叠）

#### Scenario: 空输入不触发

- **WHEN** 搜索框内容为空字符串
- **THEN** 按 Enter 或点击搜索按钮不弹出 toast，不触发 `onSearch` 回调，无视觉反馈变化

#### Scenario: 全空格输入不触发

- **WHEN** 搜索框内容为 `"   "`（全空格）
- **THEN** 不触发 `onSearch` 回调，不弹出 toast，内容保持不清除（允许用户继续编辑）

#### Scenario: 超长文本截断

- **WHEN** 用户粘贴长度 > 200 字符的文本
- **THEN** Input 截断至 200 字符（`maxLength` 约束）
- **AND** 不触发 toast 或错误提示

#### Scenario: 快速连续点击防抖

- **WHEN** 用户在 300ms 内连续点击搜索按钮 5 次
- **THEN** 仅弹出 1 个 toast（300ms debounce 生效）
- **AND** 搜索框内容在第 1 次点击后清空，后续 4 次因内容为空而忽略

#### Scenario: JavaScript 被禁用

- **WHEN** 浏览器禁用 JavaScript，Hero 区以 SSR HTML 渲染
- **THEN** 背景图以 `<img>` 标签渲染（next/image fallback）
- **AND** 搜索框显示为 `disabled` 态原生 `<input>`，按钮不可点击

### Requirement: 响应式布局

The search bar SHALL adapt to viewport width: mobile 90vw max 420px, tablet max-w-xl (576px), desktop max-w-2xl (672px). Content area SHALL be horizontally padded and centered across all breakpoints.

#### Scenario: 移动端布局

- **WHEN** 视口宽度 < 768px
- **THEN** 搜索框宽度 90vw、最大 420px、`px-4` padding
- **AND** 搜索框距 Hero 底部 32px（`pb-8`）
- **AND** 内容区水平 padding 16px（`px-4`）

#### Scenario: 平板端布局

- **WHEN** 视口宽度 768–1024px
- **THEN** 搜索框宽度 `max-w-xl`（576px）居中
- **AND** 内容区水平 padding 32px（`px-8`）

#### Scenario: 桌面端布局

- **WHEN** 视口宽度 ≥ 1024px
- **THEN** 搜索框宽度 `max-w-2xl`（672px）居中
- **AND** 内容区 `max-w-4xl` 居中

### Requirement: 可访问性

The Hero section SHALL meet WCAG AA accessibility baseline: decorative image with empty alt, search input with aria-label, keyboard-navigable focus states, and 4.5:1 contrast ratio on all text.

#### Scenario: 辅助技术兼容

- **WHEN** 屏幕阅读器解析 Hero 区
- **THEN** 背景图 `alt=""` 被忽略（装饰性）
- **AND** 搜索框有 `aria-label="Search destinations and guides"`
- **AND** 搜索框支持 Tab 键聚焦，Enter 键提交，焦点样式 `ring-2 ring-blue-500`
- **AND** 白色文字在渐变遮罩上对比度 ≥ 4.5:1（WCAG AA）
