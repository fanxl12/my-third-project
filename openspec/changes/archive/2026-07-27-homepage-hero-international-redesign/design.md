# Design: homepage-hero-international-redesign

## Context

基于产品理念（面向外国游客的在华实时任务执行助手）重设计 Hero 区，从 placeholder 状态升级为具备国际化产品感的品牌首屏。WanderChina 的设计原则：**内容体现中国（照片），UI 保持国际（极简现代）**。

## Decisions

| 决策 | 选择 | 理由 |
|------|------|------|
| 背景图 | Unsplash 高质量中国城市照片（上海外滩/北京故宫/桂林山水），`next/image` `fill` + `priority` | 无需版权采购即可获得高质量素材；`priority` 确保 LCP 最优 |
| 背景图 fallback | `onError` 时回退到 `bg-gradient-to-b from-slate-900 to-slate-700` | 保留品牌色渐变作为安全网 |
| 动画方案 | 纯 CSS `@keyframes`（fadeInUp、breathingScale、bounce），不使用 framer-motion | 零额外依赖；CSS 动画在 Next.js SSR 场景天然兼容 |
| 入场动画触发 | `animation-delay` stagger（100ms 间隔），组件挂载即播放 | 无需 JS 监听滚动；首屏动画即刻触发合理 |
| 主标题 | **"China, Simplified."** | 极简有力，外国用户秒懂；呼应"复杂中国变简单"的产品理念 |
| 副标题 | "Verified guides, real traveler tips, and instant AI help — so you always know what to do next." | 传达三支柱（攻略+社区+AI）和价值主张（always know what to do next） |
| 搜索栏样式 | `backdrop-blur-xl bg-white/10` 毛玻璃 pill 输入框 | 现代国际化 UI 标配（Apple/Stripe 风格），与背景图自然融合 |
| 城市快捷标签 | 6 个城市 pill 标签行（Beijing, Shanghai, Xi'an, Chengdu, Guilin, Hangzhou），点击跳转 `/city/{id}` | 降低搜索门槛，一眼展示覆盖城市；当前目标页 404 可接受（后续实现） |
| CTA 按钮 | 双按钮：Outline "Explore Destinations" → `/attractions` + Solid "Ask AI" → 触发 AI 面板 | 对应两个核心用户路径：自助浏览 vs 即时帮助 |
| 滚动指示器 | ChevronDown 图标 + CSS bounce 动画 | 告诉用户下面有内容，提升首页浏览深度 |
| 配色 | 主色调从 `blue-900→blue-700` 改为 `slate-900→slate-800` 渐变遮罩 + 暖橙 `orange-500` 点缀（Upvote 同色系） | slate 比 blue 更国际化、更沉稳；暖橙作为行动色有活力 |
| 字体 | 保持 `font-sans`（Geist），主标题 48-64px bold | Geist 是 Vercel 现代无衬线体，国际用户阅读体验好 |
| `prefers-reduced-motion` | 全部动画在 `@media (prefers-reduced-motion: reduce)` 下禁用 | WCAG 无障碍要求 |

## 组件结构

```
components/homepage/hero.tsx
- HeroSection() — Client Component
  - 背景层（absolute）：
    - next/image（真实摄影，fill + priority + onError fallback）
    - 渐变遮罩（bg-gradient-to-t from-black/70 via-black/40 to-black/20）
    - CSS breathingScale 动画（transform: scale(1.05) 缓慢缩放）
  - 前景内容层（relative z-10 + flex col + 居中）：
    - <h1> "China, Simplified."（fadeInUp, delay 0ms）
    - <p> 副标题（fadeInUp, delay 100ms）
    - 搜索栏区域（fadeInUp, delay 200ms）：
      - glassmorphism Input + Search Button
    - 城市快捷标签行（fadeInUp, delay 300ms）：
      - 6 个 <a> pill 标签
    - CTA 按钮组（fadeInUp, delay 400ms）：
      - Outline Button "Explore Destinations"
      - Solid Button "Ask AI"
  - 滚动指示器（absolute bottom-8 + bounce 动画）
```

## 文案策略

| 元素 | 英文文案 | 设计意图 |
|------|---------|---------|
| 主标题 | China, Simplified. | 两个字传达核心价值：把复杂的中国旅行变简单 |
| 副标题 | Verified guides, real traveler tips, and instant AI help — so you always know what to do next. | "verified"建立信任，"real"区分于 AI 生成内容，"always know what to do next"直击焦虑 |
| 搜索 placeholder | Where are you headed? | 口语化，轻松友好，不强硬 |
| CTA 1 | Explore Destinations | 明确行动，对应攻略浏览路径 |
| CTA 2 | Ask AI | 极简，强调即时性 |
| 城市标签 | Beijing · Shanghai · Xi'an · Chengdu · Guilin · Hangzhou | 直接展示覆盖范围 |
| SEO title | WanderChina — China, Simplified. | 保持一致 |
| SEO description | WanderChina helps foreign travelers navigate China with verified attraction guides, a community of real travelers, and instant AI assistance. | 具体描述三支柱 + 目标用户 |

## CSS 动画关键值

```css
/* globals.css 新增 */

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(24px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes breathingScale {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(8px); }
}

/* 入场动画基类 */
.animate-fade-in-up {
  animation: fadeInUp 0.6s ease-out both;
}

/* 背景呼吸 */
.animate-breathing {
  animation: breathingScale 20s ease-in-out infinite;
}

/* 跳动指示器 */
.animate-bounce-slow {
  animation: bounce 2s ease-in-out infinite;
}

/* 无障碍：减少动画偏好 */
@media (prefers-reduced-motion: reduce) {
  .animate-fade-in-up,
  .animate-breathing,
  .animate-bounce-slow {
    animation: none;
  }
}
```

## 搜索行为逻辑（不变）

```
用户输入 → 按 Enter / 点击 Button
  ├─ 内容空/全空格 → 不触发
  ├─ 内容非空 → toast("Search is coming soon!")
  │   ├─ 清空 input
  │   └─ 按钮 disabled 300ms（防抖）
  └─ 快速点击 → 首次清空后因空值忽略
```

## 响应式断点

| 断点 | 主标题 | 副标题 | 搜索栏宽度 | 城市标签 | CTA 排列 |
|------|--------|--------|-----------|---------|---------|
| <768px（移动） | text-3xl | text-base | 90vw max 420px | 3×2 grid | 竖向堆叠 |
| 768–1023px（平板） | text-5xl | text-lg | max-w-xl | 6 个一行 | 水平排列 |
| ≥1024px（桌面） | text-6xl | text-xl | max-w-2xl | 6 个一行 | 水平排列 |

## 文件清单

```
frontend/
├── app/
│   ├── page.tsx                     # 修改：更新 SEO metadata 文案
│   └── globals.css                  # 修改：新增 @keyframes 动画
├── components/homepage/
│   └── hero.tsx                     # 重写：全面升级 HeroSection
└── public/images/
    └── hero-bg.jpg                  # 新增：默认 Hero 背景图（Unsplash 中国城市摄影）
```

无新增 npm 依赖。背景图使用 `next/image`（已内置）。动画使用纯 CSS。
