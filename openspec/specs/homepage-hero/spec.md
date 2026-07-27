# Spec: Homepage — Hero 区

## Purpose

本规范定义 WanderChina 首页首屏品牌展示区。Hero 区为 Client Component，负责真实城市摄影背景、品牌标语、副标题、毛玻璃搜索框、城市快捷标签、双 CTA 按钮、入场动画和滚动指示器。搜索框本期仅弹出 "Coming Soon" toast，真实搜索能力由后续 spec 承接。

技术基线：React 19 + Next.js 16 App Router + Tailwind CSS 4 + shadcn/ui + lucide-react。HeroSection 为 Client Component（含搜索交互状态），动画使用纯 CSS @keyframes（零依赖）。

---

## Requirements

### Requirement: 品牌主视觉与文字展示

The Hero section SHALL display a full-viewport real city photograph as background (via `next/image` with `fill` + `priority`), overlaid with a dark gradient mask ensuring text readability. The background SHALL slowly scale between 100%–105% over 20s (CSS "breathing" animation). On image load failure, it SHALL fall back to `bg-gradient-to-b from-slate-900 to-slate-700`.

The headline SHALL be **"China, Simplified."** in bold white, responsive: `text-3xl` (mobile), `text-5xl` (tablet), `text-6xl` (desktop).

The subtitle SHALL read: **"Verified guides, real traveler tips, and instant AI help — so you always know what to do next."** in `text-white/80`.

All text SHALL maintain WCAG AA 4.5:1 contrast ratio against the gradient overlay.

#### Scenario: Hero 区首次渲染

- **WHEN** 用户访问首页 `/`
- **THEN** Hero 区占据 `min-h-screen`，宽度 100%
- **AND** 背景使用 `next/image`（`fill` + `priority`），LCP < 2.5s
- **AND** 背景加载中显示 `placeholder="blur"` 模糊占位
- **AND** 背景上方覆盖渐变遮罩（`from-black/70 via-black/40 to-black/20`）
- **AND** 背景缓慢缩放（CSS breathingScale 动画，20s 循环）
- **AND** 主标题 `<h1>` 为 "China, Simplified."，页面内唯一顶级标题
- **AND** 副标题传达三支柱价值主张

#### Scenario: 背景图加载失败

- **WHEN** 背景图 URL 返回 404 或网络超时
- **THEN** `onError` 触发，fallback 为 `bg-gradient-to-b from-slate-900 to-slate-700`
- **AND** 主标题和副标题仍然正常显示
- **AND** 控制台不出现未捕获异常

#### Scenario: 响应式标语字号

- **WHEN** 视口宽度 < 768px（移动端）
- **THEN** 标题 `text-3xl`，副标题 `text-base`
- **WHEN** 视口宽度 768–1023px（平板端）
- **THEN** 标题 `text-5xl`，副标题 `text-lg`
- **WHEN** 视口宽度 ≥ 1024px（桌面端）
- **THEN** 标题 `text-6xl`，副标题 `text-xl`

### Requirement: 搜索框交互

The search bar SHALL use glassmorphism styling (`backdrop-blur-xl bg-white/10 rounded-full`) with placeholder **"Where are you headed?"**. The submit button SHALL be an ArrowRight icon-only button. Submission behavior SHALL remain unchanged: pop Sonner toast "Search is coming soon!" and clear input. Empty or whitespace-only input SHALL not trigger submission.

#### Scenario: 搜索框毛玻璃效果

- **WHEN** Hero 区渲染
- **THEN** 搜索框 Input 有 `backdrop-blur-xl bg-white/10 rounded-full border-0` 样式
- **AND** placeholder 为 "Where are you headed?"
- **AND** 提交按钮为 ArrowRight 图标（`lucide-react`），无文字

#### Scenario: 搜索框正常提交

- **WHEN** 用户在搜索框中输入 "Great Wall" 并按 Enter 或点击搜索按钮
- **THEN** 弹出 Sonner toast "Search is coming soon!"，位置 `top-center`，持续 3 秒后自动消失
- **AND** 搜索框内容清空，焦点回到搜索框
- **AND** toast 可见期间再次提交时，新 toast 替换旧 toast（不堆叠）

#### Scenario: 空输入不触发

- **WHEN** 搜索框内容为空字符串
- **THEN** 按 Enter 或点击搜索按钮不弹出 toast，不触发回调，无视觉反馈变化

#### Scenario: 全空格输入不触发

- **WHEN** 搜索框内容为 `"   "`（全空格）
- **THEN** 不触发回调，不弹出 toast，内容保持不清除（允许用户继续编辑）

#### Scenario: 超长文本截断

- **WHEN** 用户粘贴长度 > 200 字符的文本
- **THEN** Input 截断至 200 字符（`maxLength` 约束）
- **AND** 不触发 toast 或错误提示

#### Scenario: 快速连续点击防抖

- **WHEN** 用户在 300ms 内连续点击搜索按钮 5 次
- **THEN** 仅弹出 1 个 toast（300ms debounce 生效）
- **AND** 搜索框内容在第 1 次点击后清空，后续 4 次因内容为空而忽略

### Requirement: 入场动画

All foreground content elements SHALL animate in with a fade-up stagger effect on mount: headline (0ms delay) → subtitle (100ms) → search bar (200ms) → city tags (300ms) → CTA buttons (400ms). Each element SHALL use CSS `@keyframes fadeInUp` (opacity 0→1, translateY 24px→0, 600ms ease-out). All animations SHALL be disabled when `prefers-reduced-motion: reduce`.

#### Scenario: 入场动画依次播放

- **WHEN** HeroSection 组件挂载
- **THEN** 标题 fade-up 出现（0ms delay）
- **AND** 副标题 fade-up 出现（100ms delay）
- **AND** 搜索栏 fade-up 出现（200ms delay）
- **AND** 城市标签 fade-up 出现（300ms delay）
- **AND** CTA 按钮 fade-up 出现（400ms delay）
- **AND** 所有动画 `duration: 600ms ease-out`

#### Scenario: 无障碍动画偏好

- **WHEN** 用户系统设置了 `prefers-reduced-motion: reduce`
- **THEN** 所有入场动画 immediate 渲染（`animation: none`）
- **AND** 背景呼吸动画停止（`animation: none`）
- **AND** 滚动指示器跳动停止

### Requirement: 城市快捷标签

A row of 6 city quick-tag pills SHALL appear below the search bar: Beijing, Shanghai, Xi'an, Chengdu, Guilin, Hangzhou. Each tag SHALL be an `<a>` link to `/city/{id}` with glassmorphism styling. On mobile (<768px), tags SHALL display in a flex-wrap row; on tablet/desktop, in a single centered row.

#### Scenario: 城市标签渲染

- **WHEN** Hero 区渲染
- **THEN** 搜索栏下方显示 6 个城市 pill 标签
- **AND** 每个标签样式：`backdrop-blur-sm bg-white/10 rounded-full px-4 py-1.5 text-sm text-white/80 hover:bg-white/20 hover:text-white`
- **AND** 标签顺序：Beijing → Shanghai → Xi'an → Chengdu → Guilin → Hangzhou
- **AND** 每个标签 `<a href="/city/{id}">` 可点击

#### Scenario: 响应式排列

- **WHEN** 视口宽度 < 768px
- **THEN** 6 个标签以 flex-wrap 排列，间距 8px（`gap-2`）
- **WHEN** 视口宽度 ≥ 768px
- **THEN** 6 个标签在单行水平排列，间距 12px（`gap-3`）

### Requirement: 双 CTA 按钮

Two CTA buttons SHALL appear below the city tags: **"Explore Destinations"** (outline style, links to `/attractions`) and **"Ask AI"** (solid orange). On mobile, buttons SHALL stack vertically; on tablet/desktop, display side-by-side.

#### Scenario: CTA 按钮渲染

- **WHEN** Hero 区渲染
- **THEN** 城市标签下方显示两个 CTA 按钮
- **AND** "Explore Destinations" 为 outline 样式：border-white/30 文字 white，`<Link href="/attractions">`
- **AND** "Ask AI" 为 solid 样式：`bg-orange-500 hover:bg-orange-600 text-white`
- **AND** "Ask AI" 点击触发 toast "AI Assistant is coming soon!"（占位行为，后续对接真实面板）

#### Scenario: 移动端堆叠

- **WHEN** 视口宽度 < 768px
- **THEN** 两个按钮竖向堆叠，各占 100% 宽度
- **AND** 间距 12px（`gap-3`）

#### Scenario: 桌面端并排

- **WHEN** 视口宽度 ≥ 768px
- **THEN** 两个按钮水平并排，间距 16px（`gap-4`）

### Requirement: 滚动指示器

A scroll-down indicator SHALL appear at the bottom-center of the Hero section: a ChevronDown icon with CSS bounce animation (2s loop, translateY 0→8px→0) and semi-transparent white color. The indicator SHALL have `aria-hidden="true"` (purely decorative).

#### Scenario: 滚动指示器渲染

- **WHEN** Hero 区渲染
- **THEN** 底部居中显示 ChevronDown 图标（lucide-react）
- **AND** 图标样式：`text-white/50 h-8 w-8`
- **AND** CSS bounce 动画持续跳动（2s ease-in-out infinite）
- **AND** `aria-hidden="true"`（装饰性）

#### Scenario: 无障碍动画偏好

- **WHEN** 用户系统设置了 `prefers-reduced-motion: reduce`
- **THEN** 跳动动画停止，箭头静态显示

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
