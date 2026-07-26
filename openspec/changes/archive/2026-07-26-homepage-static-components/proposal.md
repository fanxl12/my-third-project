# Proposal: homepage-static-components

## Why

首页当前 6 个区块均为占位组件（dashed-border mock），需要替换为真实实现。按照分批交付策略，本批次先交付两个静态数据驱动的组件——NavCards（功能导航卡片）和 CityGrid（城市快速入口网格）。这两个组件无 API 依赖、无 mock 数据获取逻辑，复杂度最低，适合先行完成。

## What Changes

1. **替换 `components/homepage/nav-cards.tsx`**：实现 NavCards Server Component + NavCard Client Component，渲染三张导航卡片（Travel Community / Attraction Guides / AI Assistant），带 Link 跳转和 hover 动效
2. **替换 `components/homepage/city-grid.tsx`**：实现 CityGrid Server Component + CityItem Client Component，渲染 8 个 MVP 城市网格，带 SVG 图标 fallback 和 hover 动效

## 影响范围

| 影响对象 | 变更类型 |
|----------|----------|
| `frontend/components/homepage/nav-cards.tsx` | 替换（占位 → 真实实现） |
| `frontend/components/homepage/city-grid.tsx` | 替换（占位 → 真实实现） |

无新增依赖，无新增文件。

## 验收标准

- [ ] NavCards 渲染三张等宽卡片：Travel Community / Attraction Guides / AI Assistant
- [ ] 每张卡片含 lucide-react 图标、标题、描述，点击 Link 跳转
- [ ] 桌面端 hover 卡片上浮 4px + shadow 加深，200ms ease-out
- [ ] CityGrid 渲染 8 个城市，4 列 grid（桌面/平板），3 列 grid（移动端）
- [ ] SVG 图标缺失时显示首字母 fallback
- [ ] 城市点击跳转 `/city/{id}`，Link 包裹
- [ ] `pnpm build` 编译成功
