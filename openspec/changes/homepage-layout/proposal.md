# Proposal: homepage-layout

## Why

首页当前为脚手架占位页（显示 "My Third Project" + HealthChecker），需要替换为符合 [homepage-layout spec](../../specs/homepage-layout/spec.md) 的完整首页布局——注册 `/` 路由、编排 6 个区块插槽、统一 SEO metadata。其余 6 个 spec（Hero、NavCards、HotPosts、HotAttractions、CityGrid、AiEntry）的实现由各自变更承接，本变更负责"骨架"编排和路由注册。

## What Changes

1. **替换 `app/page.tsx`**：从 HealthChecker 占位页替换为 HomePage Server Component，按固定顺序渲染 6 个 `<section>` 区块
2. **更新 `app/layout.tsx`**：metadata title 从 "My Third Project" 改为 "WanderChina — Explore China, One City at a Time"，添加 description 和 Open Graph 标签
3. **创建 6 个占位组件**：在 `components/homepage/` 下创建各区块的占位 mock 组件（dashed-border 占位 div），满足 spec 中"子组件占位容错"要求
4. **添加 OG 图片占位**：`public/images/og-homepage.jpg`（或 SVG 占位）

## Capabilities

### New Capabilities

- `homepage-layout`: 首页路由注册、6 区块插槽编排、SEO metadata、响应式容器约束、统一间距（实现 [homepage-layout spec](../../specs/homepage-layout/spec.md)）

### Modified Capabilities

_无。本变更实现已有 spec，不修改现有规范需求。_

## Impact

| 影响对象 | 变更类型 |
|----------|----------|
| `frontend/app/page.tsx` | 替换（移除 HealthChecker 占位，写入 HomePage） |
| `frontend/app/layout.tsx` | 修改（更新 metadata） |
| `frontend/components/homepage/hero.tsx` | 新增（占位组件） |
| `frontend/components/homepage/nav-cards.tsx` | 新增（占位组件） |
| `frontend/components/homepage/hot-posts.tsx` | 新增（占位组件） |
| `frontend/components/homepage/hot-attractions.tsx` | 新增（占位组件） |
| `frontend/components/homepage/city-grid.tsx` | 新增（占位组件） |
| `frontend/components/homepage/ai-entry.tsx` | 新增（占位组件） |
| `frontend/public/images/og-homepage.jpg` | 新增（OG 社交分享图） |

## 风险评估

| 风险 | 影响 | 缓解措施 |
|------|------|----------|
| 6 个占位组件后续被替换时可能破坏布局 | 替换真实组件后排版异常 | layout 层统一控制间距和容器，子组件只需关注自身内容；占位组件接口与 spec 契约对齐 |
| `next build` 因缺失的组件导入报错 | 编译失败 | spec 要求"缺失组件触发编译期错误"，此为预期行为；所有导入路径先在占位组件中创建 |

## 验收标准

- [ ] `app/page.tsx` 按 spec 代码骨架渲染 HomePage（6 个 section + SEO metadata）
- [ ] `app/layout.tsx` metadata 更新为 WanderChina 品牌信息 + Open Graph 标签
- [ ] 6 个占位组件存在于 `components/homepage/`，渲染 dashed-border mock 容器
- [ ] `pnpm build` 编译成功，无 TypeScript 错误
- [ ] `pnpm dev` 启动后访问 `http://localhost:3000`，按序显示 6 个区块
- [ ] Hero 区全宽渲染，其余 5 区块在 `max-w-7xl` 容器内居中
- [ ] `<title>` 为 WanderChina 品牌标题
