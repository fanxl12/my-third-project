# Design: homepage-layout

## Context

WanderChina 前端当前处于脚手架状态：`app/page.tsx` 渲染 HealthChecker 占位页，`app/layout.tsx` 使用 "My Third Project" 作为 metadata title。首页的 7 个 spec（layout + 6 个子组件区块）已全部定稿，shadcn/ui + lucide-react 工具链已就绪。本变更是首页实现的第一步——先搭骨架（路由 + 布局 + 占位组件），后续 6 个变更各自实现子组件并替换占位。

## Goals / Non-Goals

**Goals:**
- 替换 `app/page.tsx` 为符合 homepage-layout spec 的 HomePage Server Component
- 更新 `app/layout.tsx` metadata 为 WanderChina 品牌信息 + Open Graph
- 创建 6 个占位组件（dashed-border mock），确保布局可编译、可预览
- 统一间距常量 `SECTION_GAP`、响应式容器 `max-w-7xl`

**Non-Goals:**
- 不实现任何子组件的真实内容（Hero 背景图、导航卡片、帖子列表等由各自变更负责）
- 不添加后端 API 或数据获取逻辑
- 不引入新依赖

## Decisions

| 决策 | 选择 | 理由 |
|------|------|------|
| HomePage 组件类型 | Server Component（无 `'use client'`） | spec 要求；SEO metadata 需在服务端输出；子组件可按需声明为 Client Component |
| 占位组件位置 | `components/homepage/*.tsx` | 与 spec 代码骨架的导入路径 `@/components/homepage/` 一致 |
| 占位组件接口 | 默认导出函数组件，无 props | YAGNI — 当前阶段只需渲染占位 div；后续变更扩展 props |
| OG 图片 | 暂用纯色 SVG 占位，后续替换为 1200×630 JPG | 不阻塞布局实现；OG 图片设计由设计资源线后续交付 |
| SECTION_GAP 常量 | 在 `page.tsx` 文件内定义 `const SECTION_GAP = 'py-20 md:py-12'` | spec 明确；单文件使用无需提取到共享模块 |
| AiAssistantEntry 定位 | `fixed bottom-6 right-6`，在 `<main>` 末尾独立渲染 | spec 契约表要求；不纳入 `max-w-7xl` 容器 |
| metadata 定义位置 | `page.tsx` 中 `export const metadata` + `layout.tsx` 中 `metadata.template` | Next.js App Router 惯例；page 级 metadata 覆盖 layout 级 template |

## 组件目录结构

```
frontend/components/homepage/
├── hero.tsx              # HeroSection 占位
├── nav-cards.tsx         # NavCards 占位
├── hot-posts.tsx         # HotPosts 占位
├── hot-attractions.tsx   # HotAttractions 占位
├── city-grid.tsx         # CityGrid 占位
└── ai-entry.tsx          # AiAssistantEntry 占位（fixed 定位）
```

每个占位组件渲染 spec 要求的 dashed-border mock 容器：
```tsx
export default function HeroSection() {
  return (
    <div className="rounded-lg border-2 border-dashed border-gray-300 p-12 text-center text-gray-400">
      Mock: HeroSection
    </div>
  )
}
```

## Risks / Trade-offs

| 风险 | 缓解 |
|------|------|
| 占位组件无任何交互逻辑，后续替换时需要理解组件契约 | 每个占位组件文件名和 mock 文本与 spec 定义的组件名对齐，spec 是唯一事实来源 |
| `layout.tsx` metadata 改为 template 模式后，子页面的 metadata 可能需要调整 | 当前仅有首页一个路由，无子页面冲突风险 |
| `next build` 时 OG 图片 404 可能导致 Lighthouse 扣分 | 后续设计交付后替换为真正的 1200×630 JPG；当前阶段可接受 |
