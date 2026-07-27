# Proposal: homepage-hero-international-redesign

## Why

当前 Hero 区为纯色渐变背景 + 基础文案的 placeholder 状态，视觉上缺乏产品感，不符合面向外国游客的国际化旅游产品定位。WanderChina 的核心价值是"在华实时任务执行助手"，Hero 区需要在首屏立刻传达：**这是为外国游客在中国旅行时解决问题的工具**——不是又一个泛旅游攻略站。

目标用户（20-40 岁英语母语游客）习惯的国际化网站风格是：大量留白、真实摄影背景、清晰的价值主张、现代微动效。当前 Hero 区与之差距明显。

## What Changes

1. **重写 `components/homepage/hero.tsx`**：全面升级 HeroSection Client Component
   - 真实城市摄影背景图（中国地标）+ 渐变遮罩 + CSS 呼吸缩放动画
   - 新文案：「China, Simplified.」主标题 + 具体价值主张副标题
   - 毛玻璃搜索栏（backdrop-blur）+ 热门城市快捷标签行
   - 双 CTA 按钮：Explore Destinations + Ask AI
   - 入场动画（标题→副标题→搜索栏→CTA 依次 fade-up）
   - 滚动指示器（底部跳动箭头）
   - 保留 Sonner toast "Search is coming soon!" 行为
2. **更新 `app/page.tsx` SEO metadata**：title 和 description 与新文案对齐
3. **更新 `globals.css`**：新增 @keyframes 动画定义（fade-up、breathing-scale、bounce）

## Capabilities

### Modified Capabilities

- `homepage-hero`: 品牌主视觉、文案、搜索栏样式、交互动效全面升级

## Impact

| 影响对象 | 变更类型 |
|----------|----------|
| `frontend/components/homepage/hero.tsx` | 重写 |
| `frontend/app/page.tsx` | 修改（SEO metadata 文案更新） |
| `frontend/app/globals.css` | 修改（新增 keyframes） |

## 风险评估

| 风险 | 影响 | 缓解措施 |
|------|------|----------|
| 背景图未就绪（尚无正式品牌摄影） | Hero 区无图显示 | 使用 Unsplash 免费高质量中国城市照片作为默认背景；同时保留 CSS 渐变 fallback |
| 入场动画在低端设备卡顿 | 用户体验下降 | 使用 CSS `@keyframes` 而非 JS 动画库（零依赖）；`prefers-reduced-motion` 媒体查询关闭动画 |
| 热门城市标签点击后目标页 404 | 用户困惑 | 标签指向的 `/city/{id}` 路由当前为 404，标签点击行为留待城市路由实现后激活；现阶段标签仅作视觉引导 |
| framer-motion 额外依赖 | 包体积增大 | 不使用 framer-motion，全部动画用 CSS `@keyframes` + `animation-delay` 实现 |

## 验收标准

- [ ] Hero 区全屏渲染：真实城市照片背景 + 渐变遮罩 + 呼吸缩放动画
- [ ] 新主标题 "China, Simplified." 响应式字号 + fade-up 入场
- [ ] 副标题传达三支柱价值主张（guides + community + AI）
- [ ] 搜索栏毛玻璃效果（backdrop-blur），保留空/空格忽略 + toast 行为
- [ ] 6 个热门城市快捷标签行，hover 变色
- [ ] 双 CTA 按钮：Explore Destinations + Ask AI
- [ ] 底部跳动箭头滚动指示器（CSS bounce 动画）
- [ ] `prefers-reduced-motion` 时关闭所有动画
- [ ] WCAG AA 可访问性：aria-label、键盘导航、对比度 ≥ 4.5:1
- [ ] `pnpm build` 编译成功
