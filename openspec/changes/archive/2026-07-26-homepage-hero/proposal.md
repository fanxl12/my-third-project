# Proposal: homepage-hero

## Why

Hero 区是首页首屏品牌展示区，当前为 dashed-border 占位组件。需要替换为符合 [homepage-hero spec](../../specs/homepage-hero/spec.md) 的真实实现——品牌主视觉 + 搜索交互 + 响应式布局。

## What Changes

1. **替换 `components/homepage/hero.tsx`**：从占位组件替换为 HeroSection Client Component
   - 品牌标语 + 副标题（响应式字号）
   - 渐变遮罩背景 + next/image 背景图
   - 搜索框（shadcn Input）+ 搜索按钮（shadcn Button + lucide-react Search 图标）
   - Sonner toast "Search is coming soon!"（300ms debounce）
   - WCAG AA 可访问性

## 影响范围

| 影响对象 | 变更类型 |
|----------|----------|
| `frontend/components/homepage/hero.tsx` | 替换（占位 → 真实实现） |

## 验收标准

- [ ] Hero 区 `min-h-screen` 全屏渲染，渐变遮罩覆盖背景
- [ ] 响应式标语字号：移动端 text-3xl / 平板 text-5xl / 桌面 text-6xl
- [ ] 搜索框 submit 弹出 Sonner toast，空/空格输入忽略
- [ ] 300ms debounce 防止快速点击重复 toast
- [ ] Input maxLength=200，空格输入不清空
- [ ] `aria-label`、`aria-hidden`、`aria-busy` 正确设置
- [ ] `pnpm build` 编译成功
