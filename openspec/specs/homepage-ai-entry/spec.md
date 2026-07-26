# Spec: Homepage — AI 助手入口

## Purpose

本规范定义 WanderChina 首页全局悬浮 AI 对话入口。纯前端 UI 组件，真实对话能力由后续 `specs/ai-assistant.md` 承接。AiAssistantEntry 为全 Client Component，使用 `useAiChatPanel` Hook 管理面板展开/收起、输入状态和 toast 提交。

技术基线：React 19 + Next.js 16 App Router + Tailwind CSS 4 + shadcn/ui + lucide-react。

---

## Requirements

### Requirement: 悬浮按钮

A circular floating button with lucide-react `Bot` icon SHALL be fixed to the bottom-right corner of the viewport (`position: fixed`), always visible regardless of scroll position.

#### Scenario: 悬浮按钮始终可见

- **WHEN** 用户浏览首页任意位置（包括滚动至页面底部）
- **THEN** 右下角展示圆形悬浮按钮：桌面端 56×56px（`bottom-6 right-6`），移动端 48×48px（`bottom-4 right-4`）
- **AND** 背景色 `bg-blue-500 hover:bg-blue-600`，Bot 图标 24×24 `text-white`
- **AND** 带阴影 `shadow-lg`，`z-40`，不随页面滚动消失

#### Scenario: JavaScript 禁用

- **WHEN** 浏览器禁用 JavaScript，页面以 SSR HTML 渲染
- **THEN** 悬浮按钮显示为静态元素，点击无反应（无 toast、无面板）
- **AND** 按钮仍可见，用户不困惑

### Requirement: 聊天面板展开/收起

Clicking the floating button SHALL toggle a chat panel with 300ms slide-up/slide-down animation. Panel SHALL include Header (title + X close button), Body (empty hint), and Footer (Input + Send button). Panel SHALL close on X click, Escape key, or overlay click (mobile only).

#### Scenario: 点击展开面板

- **WHEN** 面板处于收起状态，用户点击悬浮按钮
- **THEN** 面板 slide-up 展开（300ms ease-out）
- **AND** 面板样式：白色背景、`rounded-2xl`、`shadow-2xl`；桌面端 380×480px，移动端 `calc(100vw-32px)`×60vh
- **AND** 面板定位在按钮上方 16px（`fixed bottom-20 right-6`）
- **AND** 面板内部：Header "AI Assistant" + X 关闭按钮、Body 空态提示 "How can I help you today?"、Footer Input + Send Button
- **AND** 输入框自动聚焦（`autoFocus`）
- **AND** 移动端面板展开时 body scroll 锁定（`overflow: hidden`）

#### Scenario: 关闭面板

- **WHEN** 用户点击 X 关闭按钮、按 Escape 键、或（移动端）点击面板外 overlay
- **THEN** 面板 slide-down 收起（300ms ease-in）
- **AND** 输入框内容清空，body overflow 恢复

#### Scenario: 快速切换防抖

- **WHEN** 用户在 350ms 内（动画期间）连续点击悬浮按钮 3 次
- **THEN** 面板状态以最后一次点击为准
- **AND** 不出现面板卡在半开状态，动画不叠加导致闪烁

### Requirement: 消息提交（占位行为）

Submitting a non-empty message SHALL clear the input and show a Sonner toast "AI Assistant is coming soon!". Empty or whitespace-only input SHALL be ignored with Send button remaining disabled.

#### Scenario: 正常提交消息

- **WHEN** 输入框内容为 "What documents do I need for the Forbidden City?"，用户点击 Send 或按 Enter
- **THEN** 输入框清空，Sonner toast 弹出 "AI Assistant is coming soon! Browse attraction guides in the meantime."（`top-center`，3 秒）
- **AND** 面板保持展开，焦点回到输入框

#### Scenario: 空输入不触发

- **WHEN** 输入框内容为空或全空格
- **THEN** `handleSend` 不触发，不弹出 toast，Send 按钮保持 `disabled` 态（`opacity-50, cursor-not-allowed`）

#### Scenario: 超长文本

- **WHEN** 用户粘贴 > 500 字符的文本
- **THEN** 输入框内容截断至 500 字符（`maxLength` 约束），不弹 toast，无报错

### Requirement: z-index 层级规范

The floating button SHALL be `z-40`, the chat panel `z-50`, and Sonner toast `z-60` to ensure toast always appears above the panel.

#### Scenario: toast 与面板的层级

- **WHEN** 面板展开且 toast 弹出
- **THEN** toast（`z-60`）显示在面板（`z-50`）之上
- **AND** 移动端 overlay（`z-40`）不覆盖 toast

### Requirement: 可访问性

The floating button SHALL have `aria-label="Open AI Assistant chat"` and `aria-expanded` toggling with panel state. Panel SHALL trap focus in input field when opened and close on Escape.

#### Scenario: 辅助技术兼容

- **WHEN** 屏幕阅读器解析 AI 入口
- **THEN** 悬浮按钮有 `aria-label="Open AI Assistant chat"`，面板展开时 `aria-expanded="true"`，收起时 `aria-expanded="false"`
- **AND** 面板展开时焦点自动移至输入框，关闭按钮有 `aria-label="Close chat panel"`
- **AND** Escape 键关闭面板

### Requirement: 响应式适配

Desktop (≥768px) SHALL render panel without overlay; mobile (<768px) SHALL render semi-transparent overlay and constrain panel width to `calc(100vw - 32px)`.

#### Scenario: 桌面端

- **WHEN** 视口 ≥768px
- **THEN** 面板无背景 overlay，固定定位在按钮上方

#### Scenario: 移动端

- **WHEN** 视口 <768px
- **THEN** 面板展开时显示半透明 overlay（`bg-black/30`），点击 overlay 关闭面板
- **AND** 面板宽度不超出视口（`max-width: calc(100vw - 32px)`）
