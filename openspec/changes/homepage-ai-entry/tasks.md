# Tasks: homepage-ai-entry

## Phase 1: 实现 AiAssistantEntry

- [ ] 1.1 替换 `components/homepage/ai-entry.tsx`：写入 AiAssistantEntry 组件
  - 悬浮按钮：Bot 图标、fixed bottom-right、`bg-blue-500 hover:bg-blue-600`、`shadow-lg`、`z-40`
  - 面板：slide-up/down 动画（300ms CSS transition）、`rounded-2xl`、`shadow-2xl`、`z-50`
  - 面板结构：Header（"AI Assistant" + X close button）/ Body（"How can I help you today?"）/ Footer（Input + Send button）
  - 移动端：body scroll lock、overlay `bg-black/30`、点击 overlay 关闭
  - 提交：非空输入 → Sonner toast + 清空输入框；空输入忽略，Send 按钮 disabled
  - Edge cases：`maxLength={500}`、Escape 键关闭、`autoFocus`
  - 可访问性：`aria-label`、`aria-expanded`、关闭按钮 `aria-label`
  - 验证：按钮渲染 → 点击展开面板 → 输入提交 toast → Escape 关闭

## Phase 2: 编译验证

- [ ] 2.1 运行 `pnpm build`：确认编译成功
- [ ] 2.2 运行 `pnpm lint`：确认无 ESLint error

## Phase 3: 提交

- [ ] 3.1 frontend 子模块提交：`feat: implement AiAssistantEntry floating chat panel`
- [ ] 3.2 根仓库提交：`feat: add homepage-ai-entry change`
