# Tasks: homepage-data-components

## Phase 1: 共享工具

- [ ] 1.1 创建 `lib/mock-api.ts`：导出 `mockFetch<T>()` 函数
  - 接收 data, delayMs, shouldReject, signal 参数
  - AbortController 兼容（abort 时 reject AbortError）
  - 验证：类型安全、超时 reject 正常

## Phase 2: 实现 HotPosts

- [ ] 2.1 替换 `components/homepage/hot-posts.tsx`：写入 HotPosts + useHotPosts Hook
  - 定义 HotPost 类型 + MOCK_POSTS 数据（10 条）
  - useHotPosts Hook：四态 + retry + AbortController
  - HotPosts 组件：Skeleton（loading）/ 帖子卡片 grid（success）/ 空态 / 错误+Retry
  - 卡片：标题截断、摘要截断、Upvote、元信息、Link 跳转 `/post/{id}`
  - 验证：800ms 后 10 张卡片渲染，空数组显示空态

## Phase 3: 实现 HotAttractions

- [ ] 3.1 替换 `components/homepage/hot-attractions.tsx`：写入 HotAttractions + useHotAttractions Hook
  - 定义 HotAttraction 类型 + MOCK_ATTRACTIONS 数据（8 条）
  - useHotAttractions Hook：四态 + retry + AbortController
  - HotAttractions 组件：Skeleton（loading）/ 景点卡片 grid（success）/ 空态 / 错误+Retry
  - 卡片：封面图（next/image/fallback）、城市 Badge、名称、摘要、Link `/attractions/{id}`
  - hover：上浮 4px + shadow 加深 + 图片 scale
  - 验证：600ms 后 8 张卡片渲染，封面图 fallback 正常

## Phase 4: 编译验证

- [ ] 4.1 运行 `pnpm build`：确认编译成功
- [ ] 4.2 运行 `pnpm lint`：确认无 ESLint error
- [ ] 4.3 运行 `pnpm dev`：确认两个组件加载 → 渲染 → 交互正常

## Phase 5: 提交

- [ ] 5.1 frontend 子模块提交：`feat: implement HotPosts and HotAttractions with mock API`
- [ ] 5.2 根仓库提交：`feat: add homepage-data-components change`
