# Spec: Homepage — 热门帖子

## Purpose

本规范定义 WanderChina 首页热门帖子列表组件。展示过去 7 天按 Upvote 数降序排列的 Top 10 帖子，通过 mock API 获取数据。HotPosts 为 Client Component，使用 `useHotPosts` 自定义 Hook 管理 loading / success / empty / error 四态。

技术基线：React 19 + Next.js 16 App Router + Tailwind CSS 4 + shadcn/ui + lucide-react。

---

## Requirements

### Requirement: 帖子列表加载与渲染

The HotPosts component SHALL fetch mock data on mount, displaying 3 Skeleton cards during loading, 10 post cards on success, and supporting empty and error states.

#### Scenario: 帖子列表正常加载

- **WHEN** HotPosts 组件挂载，mock API 在 800ms 后返回 10 条帖子数据
- **THEN** 立即展示 3 个 Skeleton 卡片（灰色占位 + shimmer 动画）
- **AND** mock API resolve 后 Skeleton 消失，10 篇帖子卡片按 `upvoteCount` 降序渲染
- **AND** 每张卡片展示：标题（`text-base font-semibold`，2 行截断）、摘要（`text-sm text-gray-600`，2 行截断）、Upvote 区域（lucide-react `ArrowUp` 图标 + 数字，`text-orange-500`）、元信息行（作者名 · 城市 Badge · 时间 ago）

#### Scenario: 帖子列表为空

- **WHEN** mock API 返回空数组 `[]`
- **THEN** 展示空状态：lucide-react `Inbox` 图标（48×48，`text-gray-300`）+ 文案 "No hot posts this week. Check back later!"
- **AND** 不展示任何卡片，不展示错误/重试按钮

#### Scenario: 帖子列表加载失败

- **WHEN** mock API 模拟网络超时 reject "Network timeout"
- **THEN** Skeleton 消失，展示错误状态：lucide-react `AlertCircle` 图标（`text-red-400`）+ 文案 "Failed to load posts" + shadcn/ui `Button` "Retry"（outline variant）
- **AND** 用户点击 "Retry" → 重新进入 loading → 重新发起 mock API 请求

#### Scenario: Mock API 返回格式错误

- **WHEN** mock API 返回非数组数据（如 `{ error: "unexpected" }`）
- **THEN** 组件捕获异常，展示错误状态 + Retry 按钮
- **AND** 控制台输出详细错误日志（仅 dev 模式）

### Requirement: 帖子卡片交互

Each post card SHALL be a clickable `<Link>` to `/post/{postId}`. Cards SHALL handle edge cases: missing authorName → "Anonymous" fallback, upvoteCount=0 → display normally, title >80 chars → line-clamp-2 truncation.

#### Scenario: 帖子卡片点击跳转

- **WHEN** 用户点击帖子卡片任意区域
- **THEN** Next.js `<Link>` 客户端路由导航到 `/post/{postId}`

#### Scenario: 部分字段缺失

- **WHEN** mock 数据中某条帖子缺少 `authorName` 字段
- **THEN** 显示为 "Anonymous"（fallback 值），卡片其余部分正常展示，不崩溃

#### Scenario: Upvote 数为 0

- **WHEN** 某条帖子 `upvoteCount = 0`
- **THEN** `ArrowUp` 图标 + "0" 正常显示，该帖子排在列表末尾

#### Scenario: 标题超长

- **WHEN** 帖子标题 > 80 字符
- **THEN** 标题在第 2 行截断（`line-clamp-2`），末尾显示 "..."，卡片高度不撑破布局

### Requirement: 数据获取 Hook

The `useHotPosts` hook SHALL manage four states (loading/success/empty/error), expose a `retry()` function, and cancel pending requests on component unmount via AbortController.

#### Scenario: 组件卸载时取消请求

- **WHEN** HotPosts 组件在 mock API 返回前卸载
- **THEN** AbortController 取消未完成请求
- **AND** 控制台无 "setState on unmounted" 警告

#### Scenario: Mock API 极慢响应

- **WHEN** mock API 延迟 5000ms（模拟极慢网络）
- **THEN** Skeleton 持续显示，页面上方热门景点等区块不受影响
- **AND** 无白屏或页面崩溃

### Requirement: 可访问性

Skeleton SHALL have `aria-busy="true"` and `aria-label="Loading posts"`. Post card links SHALL have descriptive `aria-label`. Time formatting SHALL use native `Intl.RelativeTimeFormat`.

#### Scenario: 辅助技术兼容

- **WHEN** 屏幕阅读器解析热门帖子区
- **THEN** Skeleton 有 `aria-busy="true"` 和 `aria-label="Loading posts"`
- **AND** 每张卡片 `<Link>` 有 `aria-label="Read post: {title}"`
- **AND** 时间格式使用 `Intl.RelativeTimeFormat`（Today / Yesterday / X days ago）
