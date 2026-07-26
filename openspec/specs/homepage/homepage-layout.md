# Spec: Homepage — 路由与布局容器

## 1. 模块边界

### 包含的功能

| 功能 | 说明 |
|------|------|
| 首页路由注册 | `/` 路由指向首页，使用 Next.js App Router `app/page.tsx`（Server Component） |
| 布局容器 | 全宽页面容器，内部各区块垂直堆叠，统一最大宽度 `max-w-7xl` 居中 |
| 区块插槽 | 定义 6 个区块的渲染顺序和挂载位置，每个插槽接收对应子组件 |
| 区块间距 | 各区块之间统一垂直间距（桌面端 `py-20`，移动端 `py-12`） |
| SEO Metadata | `title`: "WanderChina — Explore China, One City at a Time"；`description` 含关键词 |
| 响应式容器 | 桌面端 max-width 1280px 居中，移动端全宽 + 水平 padding |

### 不包含的功能

- ❌ 全局 Header / Footer 导航栏（本期首页为独立落地页，后续统一定义布局）
- ❌ 页面级数据获取 / SSR（所有数据由子组件自行处理）
- ❌ 页面过渡动画 / 滚动视差效果
- ❌ A/B 测试 / 特性开关
- ❌ 页面级错误边界（各子组件自行处理错误状态）

---

## 2. 核心场景

### SC-01: 首页完整渲染

```
GIVEN 用户访问 "/"
WHEN 页面加载完成
THEN 系统按以下顺序渲染 6 个区块：
      ① Hero 区 (HeroSection)
      ② 功能导航栏 (NavCards)
      ③ 热门帖子 (HotPosts)
      ④ 热门景点 (HotAttractions)
      ⑤ 城市快速入口 (CityGrid)
      ⑥ AI 助手悬浮入口 (AiAssistantEntry，fixed 定位，不参与文档流)
  AND 区块 ①-⑤ 包裹在 max-w-7xl 容器内水平居中
  AND 区块 ⑥ 固定在视口右下角，不随区块顺序影响
  AND 各区块（①-⑤）之间按统一间距分隔
```

### SC-02: 各区块独立加载（不阻塞）

```
GIVEN 首页路由已注册
WHEN 某个子组件数据加载中（如 HotPosts skeleton）
THEN 其余已就绪区块正常展示
  AND 加载中的区块展示自身的 skeleton/loading 状态
  AND 页面不会因单个区块加载慢而白屏
```

### SC-03: 移动端布局适配

```
GIVEN 视口宽度 < 768px
WHEN 首页渲染
THEN 容器水平 padding 调整为 16px（px-4）
  AND 区块间距缩小至 py-12
  AND AI 悬浮按钮缩小至 48px
  AND 所有区块内容在移动端正常可读，无横向溢出
```

### SC-04: SEO Metadata 生效

```
GIVEN 首页已部署
WHEN 搜索引擎爬虫抓取 "/"
THEN <title> 标签为 "WanderChina — Explore China, One City at a Time"
  AND <meta name="description"> 包含：
      "WanderChina helps foreign travelers explore China with curated attraction guides,
       real traveler tips, and instant AI assistance."
  AND Open Graph 标签包含 title、description、og:image
```

---

## 3. 组件定义

### 3.1 路由文件

```
frontend/app/page.tsx          # Server Component，首页路由入口
```

### 3.2 组件结构

```
HomePage (page.tsx)
├── <main>                     # 页面主体，bg-white
│   └── <div class="max-w-7xl mx-auto px-4">
│       ├── <section id="hero">             # 插槽 1: HeroSection
│       ├── <section id="nav-cards">        # 插槽 2: NavCards
│       ├── <section id="hot-posts">        # 插槽 3: HotPosts
│       ├── <section id="hot-attractions">  # 插槽 4: HotAttractions
│       └── <section id="city-grid">        # 插槽 5: CityGrid
└── AiAssistantEntry           # 插槽 6: fixed 定位，不在此容器内
```

### 3.3 插槽契约

每个插槽为 `<section>` 标签包裹，提供以下统一属性：

| 插槽编号 | 组件 | section id | 桌面端间距 | 移动端间距 |
|----------|------|------------|-----------|-----------|
| 1 | `HeroSection` | `hero` | `py-0`（Hero 自带高度） | `py-0` |
| 2 | `NavCards` | `nav-cards` | `py-20` | `py-12` |
| 3 | `HotPosts` | `hot-posts` | `py-20` | `py-12` |
| 4 | `HotAttractions` | `hot-attractions` | `py-20` | `py-12` |
| 5 | `CityGrid` | `city-grid` | `py-20` | `py-12` |
| 6 | `AiAssistantEntry` | — | `fixed bottom-6 right-6` | `fixed bottom-4 right-4` |

### 3.4 插槽基础模板

```tsx
// 每个插槽遵循统一的模式：
<section id="{section-id}" className="py-20 md:py-12">
  {children}
</section>
```

Hero 区例外 — 不设 py，由其自身控制高度。

---

## 4. 代码骨架（page.tsx 轮廓）

```typescript
// frontend/app/page.tsx
import type { Metadata } from "next"
import { HeroSection } from "@/components/homepage/hero/HeroSection"
import { NavCards } from "@/components/homepage/nav-cards/NavCards"
import { HotPosts } from "@/components/homepage/hot-posts/HotPosts"
import { HotAttractions } from "@/components/homepage/hot-attractions/HotAttractions"
import { CityGrid } from "@/components/homepage/city-grid/CityGrid"
import { AiAssistantEntry } from "@/components/homepage/ai-entry/AiAssistantEntry"

export const metadata: Metadata = {
  title: "WanderChina — Explore China, One City at a Time",
  description:
    "WanderChina helps foreign travelers explore China with curated attraction guides, real traveler tips, and instant AI assistance.",
  openGraph: {
    title: "WanderChina — Explore China, One City at a Time",
    description:
      "Curated attraction guides, real traveler tips, and instant AI assistance for your China adventure.",
    images: ["/images/og-homepage.jpg"],
  },
}

const SECTION_GAP = "py-20 md:py-12"

export default function HomePage() {
  return (
    <main>
      {/* 插槽 1: Hero — 全宽，无容器约束 */}
      <section id="hero">
        <HeroSection />
      </section>

      {/* 插槽 2-5: max-w-7xl 居中容器 */}
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

      {/* 插槽 6: AI 入口 — fixed 定位 */}
      <AiAssistantEntry />
    </main>
  )
}
```

> **注意**：以上为 layout spec 定义的代码骨架。各子组件的具体实现由其自身 spec 定义，layout 层只负责导入和挂载。子组件未就绪时，可用 `<div>Mock: {ComponentName}</div>` 占位。

---

## 5. Mock 数据契约

该单元为纯编排层，无 API 依赖，无 mock 数据。

---

## 6. 与其他单元依赖关系

| 依赖方向 | 说明 |
|----------|------|
| **依赖其他单元** | **仅依赖组件接口（import）**，不强依赖组件实现。子组件未就绪时使用占位 div 替代 |
| **被依赖** | 本单元是首页的**唯一入口**，其余 6 个单元渲染于此容器中 |

### 与其他 7 个 spec 的关系

```
homepage-layout.md         ← 编排层（本 spec）
  ├── homepage-hero.md       ← 插槽 1
  ├── homepage-nav-cards.md  ← 插槽 2
  ├── homepage-hot-posts.md  ← 插槽 3
  ├── homepage-hot-attractions.md ← 插槽 4
  ├── homepage-city-grid.md  ← 插槽 5
  └── homepage-ai-entry.md   ← 插槽 6 (fixed)
```

**交付顺序建议**：本 spec 先交付骨架（各插槽用占位 div），随后 6 个子单元并行开发，逐一替换占位 div 为真实组件。

---

## 7. 验收标准

- [ ] `GET /` 返回 HTTP 200，页面正常渲染
- [ ] `<title>` 标签内容为 "WanderChina — Explore China, One City at a Time"
- [ ] 6 个 `<section>` 插槽按顺序渲染，id 属性正确
- [ ] 桌面端（≥1280px）：内容 max-width 1280px 居中
- [ ] 移动端（<768px）：无横向溢出，各区块间距正常
- [ ] AI 悬浮按钮 fixed 在右下角，z-index 正确
- [ ] 替换任意占位 div 为真实组件后，其余区块不受影响
