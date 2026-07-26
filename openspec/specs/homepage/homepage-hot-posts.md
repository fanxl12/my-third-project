# Spec: Homepage — 热门帖子

## 1. 模块边界

### 包含的功能

| 功能 | 说明 |
|------|------|
| 热门帖子列表 | 展示过去 7 天按 Upvote 数排序的 Top 10 帖子 |
| 帖子卡片 | 每张卡片展示标题、摘要（截断 2 行）、Upvote 数、发布时间 |
| 加载状态 | 数据加载中展示 skeleton placeholder（3 个骨架卡片） |
| 空状态 | 无数据时展示 "No posts yet" 友好提示 |
| 错误状态 | API 请求失败时展示 "Failed to load posts" + 重试按钮 |

### 不包含的功能

- ❌ 实时 Upvote 交互（点赞/取消点赞）
- ❌ 帖子详情页（仅提供 Link 占位）
- ❌ 分页 / 无限滚动
- ❌ 帖子发布 / 编辑 / 删除
- ❌ 真实 API 实现（本期使用 mock 数据）

---

## 2. 核心场景

### SC-01: 帖子列表正常加载

```
GIVEN mock API 返回 10 篇帖子数据
WHEN 热门帖子组件挂载
THEN 系统展示 loading skeleton 状态（3 个灰色占位卡片）
WHEN 数据加载完成
THEN 展示 10 篇帖子卡片列表
  AND 每张卡片包含：标题（bold）、摘要（2 行截断）、Upvote 数 + 图标、发布时间
  AND 卡片按 Upvote 数降序排列
```

### SC-02: 帖子列表为空

```
GIVEN mock API 返回空数组 []
WHEN 数据加载完成
THEN 展示 "No hot posts this week. Check back later!" 空状态提示
  AND 不展示卡片列表
```

### SC-03: 帖子列表加载失败

```
GIVEN mock API 请求失败（网络错误或 500）
WHEN 组件捕获错误
THEN 展示 "Failed to load posts" 错误提示
  AND 展示 "Retry" 按钮
WHEN 用户点击 "Retry" 按钮
THEN 重新发起 API 请求，重新进入 loading 状态
```

### SC-04: 帖子卡片点击

```
GIVEN 热门帖子卡片已渲染
WHEN 用户点击帖子卡片
THEN 导航到 "/post/{postId}"（占位 URL）
```

---

## 3. 组件定义

### 3.1 组件结构

```
HotPosts
├── SectionTitle          # "🔥 Trending This Week"
├── PostCard[]            # 帖子卡片列表（最多 10 个）
│   ├── PostTitle         # 帖子标题
│   ├── PostSummary       # 摘要（2 行截断，text-ellipsis）
│   ├── PostMeta          # Upvote 数 + 发布时间
│   │   ├── UpvoteIcon    # SVG 图标
│   │   ├── UpvoteCount   # 数字
│   │   └── TimeAgo       # "3 days ago"
│   └── Link              # <Link href="/post/{id}">
├── LoadingSkeleton       # 加载骨架（3 个）
├── EmptyState            # 空状态提示
└── ErrorState            # 错误状态 + 重试按钮
```

### 3.2 数据 Hook

```typescript
// hooks/useHotPosts.ts
function useHotPosts(): {
  posts: HotPost[]
  loading: boolean
  error: Error | null
  retry: () => void
}
```

---

## 4. Mock 数据契约

### 4.1 API 接口（Mock）

```
GET /api/posts/hot?range=7d&limit=10
```

### 4.2 响应 TypeScript 类型

```typescript
interface HotPost {
  id: string
  title: string
  summary: string          // 前 120 字符摘要
  upvoteCount: number
  authorName: string
  cityName: string
  createdAt: string         // ISO 8601
}

// GET /api/posts/hot → ApiResponse<HotPost[]>
```

### 4.3 Mock 数据示例

```typescript
// mocks/hotPosts.ts
export const MOCK_HOT_POSTS: HotPost[] = [
  {
    id: "post-001",
    title: "Best time to visit the Great Wall in autumn?",
    summary: "I'm planning a trip to Beijing in October. What's the best section of the Great Wall to visit for autumn colors...",
    upvoteCount: 42,
    authorName: "TravelMike",
    cityName: "Beijing",
    createdAt: "2026-07-23T10:30:00Z",
  },
  // ... 共 10 条
]
```

Mock 数据存放在 `frontend/mocks/hotPosts.ts`，在开发环境下通过 Next.js API Route 或 MSW 拦截生效。

---

## 5. 与其他单元依赖关系

| 依赖方向 | 说明 |
|----------|------|
| **被依赖** | 无 |
| **依赖其他单元** | 无 |

真实 API 由后续社区模块 spec 提供。本期仅实现 mock 数据 + 前端组件，可独立交付、独立测试。
