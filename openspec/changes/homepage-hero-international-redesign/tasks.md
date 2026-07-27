# Tasks: homepage-hero-international-redesign

## Phase 1: CSS 动画基础设施

- [x] 1.1 在 `globals.css` 新增 `@keyframes fadeInUp`、`@keyframes breathingScale`、`@keyframes bounce`
- [x] 1.2 新增 `.animate-fade-in-up`、`.animate-breathing`、`.animate-bounce-slow` 工具类
- [x] 1.3 新增 `@media (prefers-reduced-motion: reduce)` 规则禁用动画

## Phase 2: 背景图片准备

- [x] 2.1 从 Unsplash 下载一张高质量中国城市摄影（建议上海外滩夜景），保存为 `public/images/hero-bg.jpg`
- [x] 2.2 确认图片尺寸 ≥ 1920×1080，文件大小 ≤ 300KB（WebP 压缩）

## Phase 3: HeroSection 重写

- [x] 3.1 重写 `components/homepage/hero.tsx`：
  - 替换渐变背景为 `next/image` 背景图（`fill` + `priority` + `placeholder="blur"`）
  - `onError` fallback 回退到 `bg-gradient-to-b from-slate-900 to-slate-700`
  - 渐变遮罩 `bg-gradient-to-t from-black/70 via-black/40 to-black/20`
  - 背景层添加 CSS breathingScale 动画
  - 主标题：`<h1>` "China, Simplified."（响应式字号：text-3xl / md:text-5xl / lg:text-6xl）
  - 副标题：`<p>` 传达三支柱价值主张（响应式：text-base / md:text-lg / lg:text-xl）
  - 入场动画：标题 delay-0、副标题 delay-100、搜索栏 delay-200、标签 delay-300、CTA delay-400
- [x] 3.2 搜索栏区域：
  - 毛玻璃 Input：`backdrop-blur-xl bg-white/10 rounded-full`
  - placeholder "Where are you headed?"
  - Button 改为仅图标 `→`（ArrowRight），保留 Search 行为
  - 保留空/空格忽略 + toast "Search is coming soon!" + 300ms 防抖
- [x] 3.3 城市快捷标签行：
  - 6 个 `<a>` pill 标签：Beijing, Shanghai, Xi'an, Chengdu, Guilin, Hangzhou
  - 样式：`backdrop-blur-sm bg-white/10 rounded-full px-4 py-1.5 text-sm text-white/80 hover:bg-white/20 hover:text-white`
  - 移动端 3×2 grid，桌面端 6 个一行 flex-wrap
- [x] 3.4 双 CTA 按钮：
  - "Explore Destinations"：shadcn Button `variant="outline"`，`border-white/30 text-white hover:bg-white/10`，Link → `/attractions`
  - "Ask AI"：shadcn Button `variant="default"`，`bg-orange-500 hover:bg-orange-600`，点击行为 → 触发 AiAssistantEntry 面板（或 toast 占位）
- [x] 3.5 滚动指示器：
  - 底部居中 ChevronDown 图标（lucide-react）
  - CSS bounce 动画，`text-white/50`
- [x] 3.6 可访问性：
  - 背景图 `alt=""`（装饰性）
  - 搜索框 `aria-label="Search destinations"`
  - 城市标签 `aria-label="Explore {city}"`
  - 所有交互元素键盘可聚焦，focus:ring 可见
  - 文字在渐变遮罩上对比度 ≥ 4.5:1

## Phase 4: SEO Metadata 更新

- [x] 4.1 更新 `app/page.tsx` metadata：
  - title: `"WanderChina — China, Simplified."`
  - description: `"WanderChina helps foreign travelers navigate China with verified attraction guides, a community of real travelers, and instant AI assistance."`
  - og:title、og:description 同步更新

## Phase 5: 验证

- [x] 5.1 运行 `pnpm build`：确认编译成功，无 TypeScript 错误
- [x] 5.2 运行 `pnpm dev`：确认 Hero 区完整渲染
  - 背景图正常加载（或 fallback 渐变）
  - 入场动画依次播放（fade-up stagger）
  - 背景缓慢呼吸缩放
  - 搜索框毛玻璃效果 + toast 功能正常
  - 城市标签 hover 交互正常
  - CTA 按钮功能正常
  - 底部箭头持续跳动
  - 移动端/平板/桌面布局正确
- [x] 5.3 Chrome DevTools 检查：
  - LCP < 2.5s（背景图 priority 预加载）
  - 无布局偏移（CLS ≈ 0）
  - prefers-reduced-motion 时动画静止
- [x] 5.4 可访问性检查：
  - Tab 键可遍历所有交互元素
  - 屏幕阅读器正确读出 aria-label
  - 对比度满足 WCAG AA

## Phase 6: 提交

- [ ] 6.1 frontend 子模块提交：`feat: redesign hero section with international style`
- [ ] 6.2 根仓库提交：`feat: add homepage-hero-international-redesign change`
