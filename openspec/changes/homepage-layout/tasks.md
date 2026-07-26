# Tasks: homepage-layout

## 1. 创建占位组件

- [ ] 1.1 创建 `components/homepage/hero.tsx` — 默认导出 HeroSection，渲染 dashed-border mock 容器
- [ ] 1.2 创建 `components/homepage/nav-cards.tsx` — 默认导出 NavCards
- [ ] 1.3 创建 `components/homepage/hot-posts.tsx` — 默认导出 HotPosts
- [ ] 1.4 创建 `components/homepage/hot-attractions.tsx` — 默认导出 HotAttractions
- [ ] 1.5 创建 `components/homepage/city-grid.tsx` — 默认导出 CityGrid
- [ ] 1.6 创建 `components/homepage/ai-entry.tsx` — 默认导出 AiAssistantEntry

## 2. 更新 layout.tsx Metadata

- [ ] 2.1 修改 `app/layout.tsx`：metadata title 改为 `template: '%s | WanderChina'`，添加 description 和 Open Graph 默认值
  - 验证：`<title>` 在首页输出完整品牌标题，子页面可使用 template 插值

## 3. 替换首页 page.tsx

- [ ] 3.1 替换 `app/page.tsx`：移除 HealthChecker 导入和占位内容，写入 HomePage Server Component
  - 导入 6 个占位组件
  - 定义 `SECTION_GAP = 'py-20 md:py-12'` 常量
  - 导出 `metadata` 对象（WanderChina 品牌信息 + Open Graph）
  - 渲染 `<main>` 内含 6 个 section 区块，秩序：Hero → NavCards → HotPosts → HotAttractions → CityGrid → AiAssistantEntry
  - ②–⑤ 包裹在 `<div className="mx-auto max-w-7xl px-4">` 中
  - Hero 全宽，AiAssistantEntry 在 `<main>` 末尾独立渲染
  - 验证：对照 spec 代码骨架逐行核对

## 4. OG 图片占位

- [ ] 4.1 在 `public/images/og-homepage.jpg` 添加占位 OG 图片（可用纯色 1200×630 SVG 或创建目录后标记为 TODO）
  - 验证：`og:image` meta 标签存在，路径 `/images/og-homepage.jpg`

## 5. 编译验证

- [ ] 5.1 运行 `pnpm build`：确认编译成功，无 TypeScript 错误
- [ ] 5.2 运行 `pnpm lint`：确认 ESLint 无新增 error
- [ ] 5.3 运行 `pnpm dev`，访问 `http://localhost:3000`：确认 6 个区块按序渲染，Hero 全宽，其余居中

## 6. 提交

- [ ] 6.1 frontend 子模块提交：`feat: implement homepage layout with placeholder components`
- [ ] 6.2 根仓库提交：`feat: add homepage-layout change artifacts`

---

**预计总耗时**：~20 分钟（6 个占位组件 5 分钟 + layout/page 更新 5 分钟 + OG 图片 2 分钟 + 编译验证 5 分钟 + 提交 3 分钟）
