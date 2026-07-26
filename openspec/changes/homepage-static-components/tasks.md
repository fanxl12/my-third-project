# Tasks: homepage-static-components

## Phase 1: 实现 NavCards

- [ ] 1.1 替换 `components/homepage/nav-cards.tsx`：写入 NavCards Server Component + NavCard Client Component
  - 定义 NAV_ITEMS 常量数组（3 项）
  - NavCards 导出：grid 容器（grid-cols-1 md:grid-cols-3）
  - NavCard：Link 包裹 + lucide-react 图标 + hover 动效 + aria-label
  - 验证：桌面端 3 列水平排列，移动端竖向堆叠

## Phase 2: 实现 CityGrid

- [ ] 2.1 替换 `components/homepage/city-grid.tsx`：写入 CityGrid Server Component + CityItem Client Component
  - 定义 CITIES 常量数组（8 城）
  - CityGrid 导出：标题 "Explore by City" + grid 容器
  - CityItem：Link + 圆形图标容器 + 首字母 fallback + hover 交互
  - 验证：桌面端 4 列，移动端 3 列，8 城顺序固定

## Phase 3: 编译验证

- [ ] 3.1 运行 `pnpm build`：确认编译成功
- [ ] 3.2 运行 `pnpm lint`：确认无 ESLint error

## Phase 4: 提交

- [ ] 4.1 frontend 子模块提交：`feat: implement NavCards and CityGrid static components`
- [ ] 4.2 根仓库提交：`feat: add homepage-static-components change artifacts`
