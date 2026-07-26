# Design: homepage-static-components

## Context

占位组件已就绪，shadcn/ui + lucide-react 工具链已安装。本变更实现两个静态驱动的子组件，替换 `components/homepage/` 下的对应占位文件。

## Goals / Non-Goals

**Goals:**
- 替换 `nav-cards.tsx` 为真实 NavCards（三张导航卡片 + Link + hover）
- 替换 `city-grid.tsx` 为真实 CityGrid（8 城网格 + SVG fallback + hover）
- 严格遵循 `openspec/specs/homepage-nav-cards/spec.md` 和 `openspec/specs/homepage-city-grid/spec.md`

**Non-Goals:**
- 不添加后端 API 或数据获取逻辑
- 不引入新依赖
- 不修改其他 4 个区块的占位组件

## Decisions

| 决策 | 选择 | 理由 |
|------|------|------|
| NavCard 组件类型 | NavCards 为 Server Component，单个 NavCard 为 Client Component | spec 要求；hover 动效需要客户端交互 |
| 卡片数据定义 | 编译期常量数组 `NAV_ITEMS`，内联在 `nav-cards.tsx` | 只有 3 条数据，YAGNI 不需要抽取 |
| CityItem 组件类型 | CityGrid 为 Server Component，单个 CityItem 为 Client Component | spec 要求；SVG onError fallback 和 hover 需客户端 |
| SVG 图标策略 | 使用 lucide-react 图标（Landmark 等）作为 inline SVG | spec 要求 SVG 图标；lucide-react 已有地标类图标可复用；CityItem 用首字母 fallback |
| 城市数据定义 | 编译期常量数组 `CITIES`，内联在 `city-grid.tsx` | 8 条固定数据，YAGNI |

## NavCards 组件结构

```
components/homepage/nav-cards.tsx
- NAV_ITEMS 常量数组（3 项，含 icon, title, description, href）
- NavCards() — Server Component，渲染 grid 容器
- NavCard({ icon, title, description, href }) — Client Component
  - Link 包裹卡片
  - hover: group-hover 实现上浮 + shadow 变化
  - lucide-react 图标（MessageCircle / MapPin / Bot）
```

### 关键样式

```tsx
// 卡片容器
className="rounded-xl bg-white p-8 shadow-md transition-all duration-200 ease-out hover:-translate-y-1 hover:shadow-lg"

// 图标
<MessageCircle className="h-8 w-8 text-blue-500" aria-hidden="true" />

// 标题
<h3 className="text-lg font-semibold text-gray-900">{title}</h3>

// 描述
<p className="text-sm text-gray-500">{description}</p>
```

## CityGrid 组件结构

```
components/homepage/city-grid.tsx
- CITIES 常量数组（8 项，含 id, nameEn, nameZh）
- CityGrid() — Server Component，渲染标题 + grid 容器
- CityItem({ city }) — Client Component
  - Link 包裹
  - 圆形背景容器 + 首字母 fallback（无外部 SVG）
  - hover: scale(1.1) + bg-blue-50 + text-blue-600
```

### 关键样式

```tsx
// grid 容器
className="grid grid-cols-3 md:grid-cols-4 gap-6"

// 圆形图标容器
className="mx-auto flex h-16 w-16 items-center justify-center rounded-full bg-gray-100 transition-colors duration-200 ease-out group-hover:bg-blue-50"

// 首字母 fallback
className="flex h-8 w-8 items-center justify-center rounded-full bg-blue-500 text-sm font-bold text-white"
```

## 代码骨架

### nav-cards.tsx

```tsx
import Link from 'next/link'
import { MessageCircle, MapPin, Bot, type LucideIcon } from 'lucide-react'

interface NavItem {
  icon: LucideIcon
  title: string
  description: string
  href: string
  ariaLabel: string
}

const NAV_ITEMS: NavItem[] = [
  {
    icon: MessageCircle,
    title: 'Travel Community',
    description: 'Connect with fellow travelers and share your China experiences.',
    href: '/community',
    ariaLabel: 'Navigate to Travel Community',
  },
  {
    icon: MapPin,
    title: 'Attraction Guides',
    description: 'Expert-curated guides to China\'s most iconic destinations.',
    href: '/attractions',
    ariaLabel: 'Navigate to Attraction Guides',
  },
  {
    icon: Bot,
    title: 'AI Assistant',
    description: 'Your intelligent travel companion for planning your trip.',
    href: '/ai-assistant',
    ariaLabel: 'Navigate to AI Assistant',
  },
]
```

### city-grid.tsx

```tsx
import Link from 'next/link'

interface City {
  id: string
  nameEn: string
}

const CITIES: City[] = [
  { id: 'beijing', nameEn: 'Beijing' },
  { id: 'shanghai', nameEn: 'Shanghai' },
  { id: 'xian', nameEn: "Xi'an" },
  { id: 'chengdu', nameEn: 'Chengdu' },
  { id: 'guangzhou', nameEn: 'Guangzhou' },
  { id: 'guilin', nameEn: 'Guilin' },
  { id: 'lijiang', nameEn: 'Lijiang' },
  { id: 'hangzhou', nameEn: 'Hangzhou' },
]
```

## Risks / Trade-offs

| 风险 | 缓解 |
|------|------|
| lucide-react 中缺少中国城市地标 SVG 图标 | 统一使用首字母 fallback（蓝色圆形 + 首字母），避免逐个寻找合适图标 |
| nav-cards hover 在移动端可能产生 sticky hover 残影 | 使用 `md:hover:` 仅在桌面端生效 |
