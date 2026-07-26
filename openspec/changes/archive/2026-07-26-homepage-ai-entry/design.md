# Design: homepage-ai-entry

## Context

AiAssistantEntry 是首页最复杂的交互组件——一个悬浮在 viewport 右下角的 AI 对话入口。与其说是一个"区块"，不如说是一个全局 UI 组件。它在 `page.tsx` 中渲染在 `<main>` 外部，不在 `max-w-7xl` 容器内，而是 `fixed` 定位。

## Decisions

| 决策 | 选择 | 理由 |
|------|------|------|
| 面板挂载策略 | 始终渲染 + CSS 控制显隐 | 避免卸载/挂载闪烁，动画更流畅 |
| 动画方式 | CSS `transition-all duration-300` | 零依赖，Tailwind 原生支持 |
| 面板显隐 | `opacity` + `translate-y` + `pointer-events` | 同时控制可见性和交互性 |
| body scroll lock | useEffect 设置 `document.body.style.overflow` | 移动端面板展开时禁止滚动 |
| 焦点管理 | `useRef` + `autoFocus` 属性 | 面板展开时输入框自动聚焦 |
| 防抖 | 无额外防抖，动画期间 CSS 过渡防抖 | spec 允许连续点击，动画不叠加即可 |

## 组件结构

```
AiAssistantEntry (page.tsx 中 <main> 外部)
├── FloatingButton (Bot icon, 56×56px, fixed bottom-right, z-40)
├── PanelContainer (fixed bottom-20 right-6, z-50)
│   ├── Overlay (仅移动端: bg-black/30, z-40, 点击关闭)
│   ├── Panel (rounded-2xl shadow-2xl bg-white)
│   │   ├── Header: "AI Assistant" + X close button
│   │   ├── Body: "How can I help you today?" (empty hint)
│   │   └── Footer: Input + Send button
```

## 状态管理

```typescript
function useAiChatPanel() {
  const [isOpen, setIsOpen] = useState(false)
  const [inputValue, setInputValue] = useState('')
  const inputRef = useRef<HTMLInputElement>(null)

  const toggle = useCallback(() => {
    setIsOpen(prev => !prev)
  }, [])

  const close = useCallback(() => {
    setIsOpen(false)
    setInputValue('')
    // body overflow restored in useEffect
  }, [])

  const handleSend = useCallback(() => {
    const value = inputValue.trim()
    if (!value) return
    toast('AI Assistant is coming soon! Browse attraction guides in the meantime.', { duration: 3000 })
    setInputValue('')
    inputRef.current?.focus()
  }, [inputValue])

  useEffect(() => {
    if (isOpen) {
      document.body.style.overflow = 'hidden'
    } else {
      document.body.style.overflow = ''
    }
    return () => { document.body.style.overflow = '' }
  }, [isOpen])

  return { isOpen, inputValue, setInputValue, toggle, close, handleSend, inputRef }
}
```

## 动画实现

使用 CSS transition 而非 framer-motion，保持零依赖：

```tsx
// 面板容器
const panelClasses = cn(
  'fixed z-50 transition-all duration-300 ease-out',
  isOpen
    ? 'opacity-100 translate-y-0 pointer-events-auto'
    : 'opacity-0 translate-y-4 pointer-events-none',
  // 响应式定位
  'bottom-20 right-4 md:right-6',
  // 响应式尺寸
  'w-[calc(100vw-32px)] md:w-[380px] h-[60vh] md:h-[480px]',
)
```

## 关键样式

### 悬浮按钮
```tsx
<button
  className="fixed bottom-4 right-4 md:bottom-6 md:right-6 z-40
    flex h-12 w-12 md:h-14 md:w-14 items-center justify-center rounded-full
    bg-blue-500 text-white shadow-lg transition-colors duration-200 hover:bg-blue-600"
  aria-label={isOpen ? 'Close AI Assistant chat' : 'Open AI Assistant chat'}
  aria-expanded={isOpen}
  onClick={toggle}
>
  <Bot className="h-6 w-6" aria-hidden="true" />
</button>
```

### 面板
```tsx
<div className={panelClasses}>
  {/* Mobile overlay */}
  {isOpen && (
    <div
      className="fixed inset-0 z-40 bg-black/30 md:hidden"
      onClick={close}
      aria-hidden="true"
    />
  )}
  {/* Panel content (relative to keep above overlay) */}
  <div className="relative z-50 flex h-full flex-col rounded-2xl bg-white shadow-2xl">
    {/* Header */}
    <div className="flex items-center justify-between border-b p-4">
      <h2 className="text-base font-semibold text-gray-900">AI Assistant</h2>
      <button onClick={close} aria-label="Close chat panel">
        <X className="h-5 w-5 text-gray-400 hover:text-gray-600" />
      </button>
    </div>
    {/* Body */}
    <div className="flex flex-1 items-center justify-center p-4">
      <p className="text-sm text-gray-400">How can I help you today?</p>
    </div>
    {/* Footer */}
    <div className="border-t p-4">
      <div className="flex items-center gap-2">
        <Input ... />
        <Button ... disabled={!inputValue.trim()}>Send</Button>
      </div>
    </div>
  </div>
</div>
```

## z-index 层级

| 元素 | z-index |
|------|---------|
| Sonner toast | z-60（Sonner 默认 + `richColors`） |
| 面板内容 | z-50 |
| 移动端 overlay | z-40 |
| 悬浮按钮 | z-40 |

## Edge Cases

| 场景 | 处理 |
|------|------|
| JS 禁用 | 按钮显示为静态 `<button>`，点击无反应 |
| 超长输入 | `maxLength={500}` 截断 |
| 连续快速点击 | CSS transition 自动处理动画防抖 |
| 组件卸载 | useEffect 清理 body overflow |
| 桌面端 overlay | 仅移动端渲染（`md:hidden`） |

## Risks / Trade-offs

| 风险 | 缓解 |
|------|------|
| body scroll lock 可能影响其他页面 | 组件卸载时清理 overflow，仅当前页面生效 |
| slide-up 动画在移动端可能卡顿 | 使用 GPU 加速属性（`translate` + `opacity`，非 `top`/`height`） |
