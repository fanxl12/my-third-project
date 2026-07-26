# Design: homepage-data-components

## Context

hot-posts 和 hot-attractions 均依赖 mock API 获取数据，共享"加载 → 成功/空/错误"四态管理。将公共 mock 工具提取为 `lib/mock-api.ts`，两个组件各自使用独立的 Hook 和 mock 数据。

## Decisions

| 决策 | 选择 | 理由 |
|------|------|------|
| Hook 存储位置 | 内联在各自组件文件中 | 仅被一个组件使用，YAGNI 不抽取 |
| 共享工具 | `lib/mock-api.ts`，导出 `mockFetch<T>()` | 注入 AbortController + 模拟延时 + 类型安全 |
| Skeleton 样式 | 使用 shadcn/ui `Skeleton` 组件 | 已安装，与 spec 一致 |
| Badge 样式 | 使用 shadcn/ui `Badge` 组件 | 已安装，帖子城市和景点城市均用 |
| next/image 封面图 | mock 数据中用 `placehold.co` 或渐变占位 | 尚无真实图片资源；使用纯色占位 + 文字覆盖 |
| 时间格式化 | `Intl.RelativeTimeFormat` | spec 要求，原生 API 零依赖 |

## 共享工具: lib/mock-api.ts

```typescript
export async function mockFetch<T>(
  data: T,
  delayMs: number,
  shouldReject = false,
  signal?: AbortSignal,
): Promise<T> {
  return new Promise((resolve, reject) => {
    const timer = setTimeout(() => {
      if (shouldReject) {
        reject(new Error('Network timeout'))
      } else {
        resolve(data)
      }
    }, delayMs)

    signal?.addEventListener('abort', () => {
      clearTimeout(timer)
      reject(new DOMException('Aborted', 'AbortError'))
    })
  })
}
```

## 组件结构

### hot-posts

```
components/homepage/hot-posts.tsx
- HotPost 类型定义
- MOCK_POSTS 数据（10 条，含作者/摘要/upvote/城市/时间）
- useHotPosts() Hook
  - state: 'loading' | 'success' | 'empty' | 'error'
  - data: HotPost[]
  - retry(): void
  - AbortController
- HotPosts() — Client Component
  - Skeleton × 3（loading 态）
  - 帖子卡片 grid（success 态）
  - 空态提示（empty 态）
  - 错误 + Retry（error 态）
```

### hot-attractions

```
components/homepage/hot-attractions.tsx
- HotAttraction 类型定义
- MOCK_ATTRACTIONS 数据（8 条，含城市/封面/浏览次数）
- useHotAttractions() Hook
  - state: 'loading' | 'success' | 'empty' | 'error'
  - data: HotAttraction[]
  - retry(): void
  - AbortController
- HotAttractions() — Client Component
  - Skeleton × 4（loading 态）
  - 景点卡片 grid + hover 动效（success 态）
  - 空态提示（empty 态）
  - 错误 + Retry（error 态）
```

## Mock 数据示例

### MOCK_POSTS（10 条，按 upvoteCount 降序预排序）

每条包含：id, title, summary, authorName?, city, upvoteCount, createdAt (ISO)

```typescript
const MOCK_POSTS: HotPost[] = [
  {
    id: '1',
    title: 'Hidden Temples of Xi'an: Beyond the Terracotta Warriors',
    summary: 'Discover ancient Buddhist temples tucked away in the backstreets of Xi'an...',
    authorName: 'Wei Zhang',
    city: "Xi'an",
    upvoteCount: 142,
    createdAt: '2026-07-20T08:00:00Z',
  },
  // ... 9 条更多
]
```

### MOCK_ATTRACTIONS（8 条，按 viewCount 降序预排序）

每条包含：id, nameEn, city, summary, viewCount, imageUrl

```typescript
const MOCK_ATTRACTIONS: HotAttraction[] = [
  {
    id: '1',
    nameEn: 'The Great Wall',
    city: 'Beijing',
    summary: 'One of the Seven Wonders of the World',
    viewCount: 24800,
    imageUrl: '', // 空字符串触发 next/image fallback
  },
  // ... 7 条更多
]
```

## 四态 Hook 模式

```typescript
type AsyncState<T> =
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'empty' }
  | { status: 'error'; error: string }

function useHotPosts() {
  const [state, setState] = useState<AsyncState<HotPost[]>>({ status: 'loading' })
  const abortRef = useRef<AbortController>()

  const fetch = useCallback(() => {
    abortRef.current?.abort()
    const controller = new AbortController()
    abortRef.current = controller

    setState({ status: 'loading' })

    mockFetch(MOCK_POSTS, 800, false, controller.signal)
      .then((data) => {
        setState(data.length === 0 ? { status: 'empty' } : { status: 'success', data })
      })
      .catch((err) => {
        if (err.name === 'AbortError') return
        setState({ status: 'error', error: err.message })
      })
  }, [])

  useEffect(() => {
    fetch()
    return () => abortRef.current?.abort()
  }, [fetch])

  return { state, retry: fetch }
}
```

## 关键样式

### HotPosts 卡片

```tsx
// 卡片容器（Link 包裹）
className="group rounded-xl border border-gray-100 bg-white p-5 shadow-sm transition-shadow duration-200 hover:shadow-md"

// 标题 2 行截断
className="text-base font-semibold text-gray-900 line-clamp-2"

// 摘要 2 行截断
className="mt-1 text-sm text-gray-600 line-clamp-2"

// Upvote 区域
<div className="flex items-center gap-1 text-orange-500">
  <ArrowUp className="h-4 w-4" aria-hidden="true" />
  <span className="text-sm font-medium">{upvoteCount}</span>
</div>

// 元信息行
className="mt-3 flex items-center gap-2 text-xs text-gray-400"
```

### HotAttractions 卡片

```tsx
// 卡片容器
className="group rounded-xl bg-white shadow-sm transition-all duration-200 ease-out hover:-translate-y-1 hover:shadow-lg"

// 封面图容器（4:3）
<div className="relative aspect-[4/3] overflow-hidden rounded-t-xl">
  {/* next/image 或 fallback */}
</div>

// 城市 Badge（叠加在封面左下角）
<Badge className="absolute bottom-2 left-2">{city}</Badge>

// 名称
<h3 className="mt-3 px-3 text-sm font-semibold text-gray-900 line-clamp-1">{nameEn}</h3>

// 摘要
<p className="mt-1 px-3 pb-3 text-xs text-gray-500 line-clamp-1">{summary}</p>
```

## 响应式 Grid

```tsx
// HotPosts grid
<div className="grid grid-cols-1 gap-4 md:grid-cols-2">

// HotAttractions grid
<div className="grid grid-cols-1 gap-4 sm:grid-cols-2 md:grid-cols-4">
```

## Risks / Trade-offs

| 风险 | 缓解 |
|------|------|
| next/image 需要真实图片 URL | mock 数据用空字符串，组件检测 `!imageUrl` 时渲染纯色 fallback |
| 10 条帖子 mock 数据全靠手写 | 使用 GPT/内联生成 10 条合理数据，满足 spec 验证即可 |
| mock API 延时导致 CI 测试缓慢 | 生产构建时 mock 不执行（tree-shaken 掉），仅 dev 阶段运行 |
