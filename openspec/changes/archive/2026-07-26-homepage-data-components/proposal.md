# Proposal: homepage-data-components

## Why

首页剩余 3 个区块中，hot-posts 和 hot-attractions 共享相同的架构模式——自定义 Hook + mock API + Skeleton 四态管理。本批次将这两个组件一起实现，提取公共的 mock API 工具，避免重复代码。

## What Changes

1. **替换 `components/homepage/hot-posts.tsx`**：实现 HotPosts Client Component + `useHotPosts` Hook
2. **替换 `components/homepage/hot-attractions.tsx`**：实现 HotAttractions Client Component + `useHotAttractions` Hook
3. **新增 `lib/mock-api.ts`**：共享的 mock API 工具（延时模拟 + AbortController 兼容）

## 影响范围

| 影响对象 | 变更类型 |
|----------|----------|
| `frontend/components/homepage/hot-posts.tsx` | 替换（占位 → 真实实现） |
| `frontend/components/homepage/hot-attractions.tsx` | 替换（占位 → 真实实现） |
| `frontend/lib/mock-api.ts` | 新增（共享 mock 工具） |

## 验收标准

- [ ] HotPosts 展示 3 个 Skeleton 加载态 → 10 个帖子卡片（按 upvoteCount 降序）
- [ ] HotPosts 空态：Inbox 图标 + "No hot posts this week"
- [ ] HotPosts 错误态：AlertCircle 图标 + "Failed to load posts" + Retry 按钮
- [ ] HotAttractions 展示 4 个 Skeleton 加载态 → 8 个景点卡片 grid（4/2/1 列）
- [ ] HotAttractions 空态：ImageOff 图标 + "No attractions available"
- [ ] HotAttractions 错误态：AlertCircle 图标 + "Failed to load attractions" + Retry 按钮
- [ ] 帖子卡片部分字段缺失 → "Anonymous" / "0" 等 fallback
- [ ] 封面图加载失败 → ImageOff fallback
- [ ] 组件卸载时 AbortController 取消请求
- [ ] `pnpm build` 编译成功
