# Spec: Homepage — 热门景点

## 1. 模块边界

### 包含的功能

| 功能 | 说明 |
|------|------|
| 热门景点卡片列表 | 展示按浏览量排序的 6-8 个景点卡片 |
| 景点卡片 | 每张卡片包含封面图、英文名称、城市名称、简短摘要 |
| 加载状态 | 数据加载中展示 skeleton placeholder（4 个骨架卡片） |
| 空状态 | 无数据时展示 "No attractions available" |
| 错误状态 | 请求失败时展示错误提示 + 重试按钮 |

### 不包含的功能

- ❌ 浏览量实时统计（本期使用 mock 数据）
- ❌ 景点详情页（仅提供 Link 占位）
- ❌ 城市筛选功能（本期不做）
- ❌ 景点收藏 / 评分
- ❌ 真实 API 实现

---

## 2. 核心场景

### SC-01: 热门景点列表加载

```
GIVEN mock API 返回 8 条景点数据
WHEN 热门景点组件挂载
THEN 展示 loading skeleton（4 个灰色卡片占位）
WHEN 数据加载完成
THEN 展示 6-8 张景点卡片（2 行 × 4 列 grid，移动端单列）
  AND 每张卡片包含：封面图、英文名称、城市名称标签、摘要
  AND 卡片按 viewCount 降序排列
```

### SC-02: 景点卡片 hover 效果

```
GIVEN 热门景点卡片已渲染
WHEN 用户鼠标悬停在卡片上
THEN 卡片轻微上浮（translateY -4px）
  AND 封面图 scale 放大至 1.05
  AND 过渡动画 duration 200ms
```

### SC-03: 景点卡片点击跳转

```
GIVEN 热门景点卡片已渲染
WHEN 用户点击景点卡片
THEN 导航到 "/attractions/{attractionId}"（占位 URL）
```

---

## 3. 组件定义

### 3.1 组件结构

```
HotAttractions
├── SectionTitle          # "🏛️ Popular Attractions"
├── AttractionCard[]      # 景点卡片 grid 列表（6-8 个）
│   ├── CoverImage        # Next.js Image，aspect-ratio 4/3，带 placeholder blur
│   ├── CityBadge         # 城市名称标签（圆角 pill）
│   ├── Name              # 景点英文名称
│   ├── Summary           # 摘要（1 行截断）
│   └── Link              # <Link href="/attractions/{id}">
├── LoadingSkeleton       # 加载骨架（4 个）
├── EmptyState            # 空状态
└── ErrorState            # 错误状态 + 重试按钮
```

### 3.2 数据 Hook

```typescript
// hooks/useHotAttractions.ts
function useHotAttractions(): {
  attractions: HotAttraction[]
  loading: boolean
  error: Error | null
  retry: () => void
}
```

---

## 4. Mock 数据契约

### 4.1 API 接口（Mock）

```
GET /api/attractions/hot?limit=8
```

### 4.2 响应 TypeScript 类型

```typescript
interface HotAttraction {
  id: string
  nameEn: string
  cityName: string
  coverImageUrl: string
  summary: string           // 前 80 字符摘要
  viewCount: number
}

// GET /api/attractions/hot → ApiResponse<HotAttraction[]>
```

### 4.3 Mock 数据示例

```typescript
// mocks/hotAttractions.ts
export const MOCK_HOT_ATTRACTIONS: HotAttraction[] = [
  {
    id: "attr-001",
    nameEn: "Forbidden City",
    cityName: "Beijing",
    coverImageUrl: "/images/attractions/forbidden-city.jpg",
    summary: "The world's largest imperial palace complex with over 9000 rooms.",
    viewCount: 15234,
  },
  {
    id: "attr-002",
    nameEn: "Terracotta Warriors",
    cityName: "Xi'an",
    coverImageUrl: "/images/attractions/terracotta-warriors.jpg",
    summary: "An army of 8000 life-size terracotta figures guarding China's first emperor.",
    viewCount: 12890,
  },
  // ... 共 6-8 条
]
```

封面图规格要求：800×600px, JPEG/WebP, ≤ 150KB, 文件存放于 `frontend/public/images/attractions/`

---

## 5. 与其他单元依赖关系

| 依赖方向 | 说明 |
|----------|------|
| **被依赖** | 无 |
| **依赖其他单元** | 无 |

真实 API 由后续景点攻略模块 spec 提供。本期仅实现 mock 数据 + 前端组件。
