# Tasks: homepage-hero

## Phase 1: 实现 HeroSection

- [ ] 1.1 替换 `components/homepage/hero.tsx`：写入 HeroSection Client Component
  - `'use client'` 指令
  - 渐变背景（from-blue-900 to-blue-700）+ 渐变遮罩
  - 品牌标语 `<h1>` + 副标题（响应式字号）
  - 搜索框（shadcn Input, non-ref controlled useRef）
  - 提交按钮（shadcn Button + Search 图标）
  - Sonner toast 调用
  - 300ms 防抖（禁用按钮）
  - 空/空格输入忽略
  - maxLength=200
  - aria-label / aria-hidden

## Phase 2: 编译验证

- [ ] 2.1 运行 `pnpm build`：确认编译成功
- [ ] 2.2 运行 `pnpm dev`：确认 Hero 区渲染 + 搜索交互正常

## Phase 3: 提交

- [ ] 3.1 frontend 子模块提交：`feat: implement HeroSection with search and toast`
- [ ] 3.2 根仓库提交：`feat: add homepage-hero change`
