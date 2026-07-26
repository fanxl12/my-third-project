# Proposal: homepage-ai-entry

## Why

首页最后一个区块——AI 助手悬浮入口。作为全局 UI 组件，始终悬浮在右下角，提供对话面板展开/收起功能。

## What Changes

1. **替换 `components/homepage/ai-entry.tsx`**：实现 AiAssistantEntry 全 Client Component
   - 悬浮按钮（Bot 图标，fixed bottom-right）
   - 聊天面板（slide-up/down 动画，输入框 + Send 按钮 + toast）
   - 移动端 overlay + body scroll lock
   - Escape 键关闭 / focus trap

## 影响范围

| 影响对象 | 变更类型 |
|----------|----------|
| `frontend/components/homepage/ai-entry.tsx` | 替换（占位 → 真实实现） |

## 验收标准

- [ ] 悬浮按钮 56×56px（桌面）/ 48×48px（移动），`bg-blue-500`，`z-40`
- [ ] 点击按钮 → 面板 slide-up 展开（300ms），输入框 autoFocus
- [ ] 点击 X / Escape / overlay（移动端）→ 面板 slide-down 收起
- [ ] 提交非空消息 → toast "AI Assistant is coming soon!"
- [ ] 空输入不触发，Send 按钮 disabled
- [ ] 面板 z-50，toast z-60，overlay z-40
- [ ] 移动端 body scroll lock + overlay 点击关闭
- [ ] aria-label / aria-expanded / 焦点管理
- [ ] `pnpm build` 编译成功
