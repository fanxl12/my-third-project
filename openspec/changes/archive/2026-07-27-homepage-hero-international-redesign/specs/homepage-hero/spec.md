# Spec Delta: Homepage — Hero 区国际化重设计

> 本 delta 对应 [homepage-hero spec](../../../specs/homepage-hero/spec.md)。此变更将 Hero 区从 placeholder 状态升级为符合国际化产品定位的品牌首屏，包括：真实摄影背景、新文案体系、毛玻璃搜索栏、入场动画、城市快捷标签、双 CTA 按钮和滚动指示器。

---

## MODIFIED Requirements

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

### Requirement: 搜索框交互（样式升级）

The search bar SHALL use glassmorphism styling (`backdrop-blur-xl bg-white/10 rounded-full`) with placeholder **"Where are you headed?"**. The submit button SHALL be an ArrowRight icon-only button. Submission behavior SHALL remain unchanged: pop Sonner toast "Search is coming soon!" and clear input. Empty/whitespace-only input SHALL not trigger submission.

#### Scenario: 搜索框毛玻璃效果

- **WHEN** Hero 区渲染
- **THEN** 搜索框 Input 有 `backdrop-blur-xl bg-white/10 rounded-full` 样式
- **AND** placeholder 为 "Where are you headed?"
- **AND** 提交按钮为 ArrowRight 图标（`lucide-react`），无文字

#### Scenario: 搜索行为不变

- **WHEN** 用户输入 "Great Wall" 并提交
- **THEN** 弹出 toast "Search is coming soon!"，300ms 防抖，input 清空
- **AND** 空/空格输入不触发（与旧 spec 行为一致）
- **AND** maxLength=200 截断（与旧 spec 行为一致）

---

## ADDED Requirements

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

A row of 6 city quick-tag pills SHALL appear below the search bar: Beijing, Shanghai, Xi'an, Chengdu, Guilin, Hangzhou. Each tag SHALL be an `<a>` link to `/city/{id}` with glassmorphism styling. On mobile (<768px), tags SHALL display in a 3×2 grid; on tablet/desktop, in a single flex-wrap row.

#### Scenario: 城市标签渲染

- **WHEN** Hero 区渲染
- **THEN** 搜索栏下方显示 6 个城市 pill 标签
- **AND** 每个标签样式：`backdrop-blur-sm bg-white/10 rounded-full px-4 py-1.5 text-sm text-white/80 hover:bg-white/20 hover:text-white`
- **AND** 标签顺序：Beijing → Shanghai → Xi'an → Chengdu → Guilin → Hangzhou
- **AND** 每个标签 `<a href="/city/{id}">` 可点击

#### Scenario: 移动端 3×2 网格

- **WHEN** 视口宽度 < 768px
- **THEN** 6 个标签以 3 列 × 2 行 CSS Grid 排列
- **AND** 标签间距 8px（`gap-2`）

#### Scenario: 桌面端单行

- **WHEN** 视口宽度 ≥ 768px
- **THEN** 6 个标签在单行 flex-wrap 中水平排列
- **AND** 间距 12px（`gap-3`）

### Requirement: 双 CTA 按钮

Two CTA buttons SHALL appear below the city tags: **"Explore Destinations"** (outline style, links to `/attractions`) and **"Ask AI"** (solid orange, triggers AI assistant panel). On mobile, buttons SHALL stack vertically; on tablet/desktop, display side-by-side.

#### Scenario: CTA 按钮渲染

- **WHEN** Hero 区渲染
- **THEN** 城市标签下方显示两个 CTA 按钮
- **AND** "Explore Destinations" 为 outline 样式：`variant="outline" border-white/30 text-white hover:bg-white/10`，`<Link href="/attractions">`
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

---

## REMOVED Requirements

_无。所有旧 spec 中关于搜索行为、可访问性、响应式布局、防抖策略的需求均保留，仅升级视觉表现层。_

---

## 与主 spec 的关系

本 delta 为 `homepage-hero` 的增量升级。实现完成后，需将本 delta 合并至 `openspec/specs/homepage-hero/spec.md` 主规范，替换原有「品牌主视觉与文字展示」和「搜索框交互」的视觉描述部分，并新增入场动画、城市标签、双 CTA 和滚动指示器四个 requirement。搜索交互的行为逻辑（toast、防抖、空输入忽略）保持不变。
