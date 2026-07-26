# Spec: Homepage — AI 助手入口

## 1. 模块边界

### 包含的功能

| 功能 | 说明 |
|------|------|
| 悬浮按钮 | 固定在页面右下角的圆形悬浮按钮（AI 图标），始终可见 |
| 展开/收起 | 点击悬浮按钮展开一个聊天输入框面板，再次点击收起 |
| 聊天输入框 UI | 文本输入框 + 发送按钮，仅为 UI 占位 |
| Coming Soon 提示 | 提交消息时弹出 toast "AI Assistant is coming soon!" |
| 响应式适配 | 移动端悬浮按钮尺寸缩小，面板适配小屏 |

### 不包含的功能

- ❌ 真实 AI 对话能力（由后续 spec `ai-assistant.md` 承接）
- ❌ RAG 检索、流式响应、对话历史
- ❌ 多轮会话上下文管理
- ❌ 与景点攻略模块的数据交换
- ❌ 用户认证（未登录用户也可看到入口）

---

## 2. 核心场景

### SC-01: 悬浮按钮始终可见

```
GIVEN 用户浏览首页任意位置
WHEN 页面渲染完成
THEN 页面右下角展示 AI 助手悬浮按钮
  AND 按钮为圆形（56px 桌面端 / 48px 移动端）
  AND 按钮使用品牌色背景 + 白色 AI 图标
  AND 按钮不随页面滚动消失（position: fixed）
  AND z-index 高于页面内容但低于 modal 层
```

### SC-02: 点击展开聊天面板

```
GIVEN 悬浮按钮已渲染且处于收起状态
WHEN 用户点击悬浮按钮
THEN 聊天面板从右下角向上弹出（slide-up 动画，300ms）
  AND 面板包含：标题 "AI Assistant"、关闭按钮、文本输入框、发送按钮
  AND 输入框自动聚焦
WHEN 用户再次点击悬浮按钮或关闭图标
THEN 聊天面板收起（slide-down 动画，300ms）
  AND 输入框内容清空
```

### SC-03: 提交消息（占位行为）

```
GIVEN 聊天面板已展开且输入框有非空文本
WHEN 用户点击发送按钮或按下 Enter
THEN 输入框清空
  AND 弹出 toast "AI Assistant is coming soon! Browse attraction guides in the meantime."
  AND 面板保持展开状态
```

---

## 3. 组件定义

### 3.1 组件结构

```
AiAssistantEntry
├── FloatingButton        # 圆形悬浮按钮（fixed, bottom-6 right-6）
│   └── AiIcon (SVG)      # AI/机器人图标
└── ChatPanel             # 可展开的聊天面板（条件渲染）
    ├── PanelHeader        # 标题 "AI Assistant" + 关闭按钮
    ├── ChatInput          # <textarea> 或 <input>
    └── SendButton         # 发送按钮
```

### 3.2 组件状态管理

```typescript
// hooks/useAiChatPanel.ts
function useAiChatPanel(): {
  isOpen: boolean
  toggle: () => void
  message: string
  setMessage: (text: string) => void
  handleSend: () => void   // 本期仅 show toast
}
```

状态使用 React `useState` 管理，不引入第三方状态库。

---

## 4. Mock 数据契约

该组件为纯前端 UI，无 API 依赖。

```typescript
// components/homepage/ai-entry/constants.ts
export const AI_ENTRY_COMING_SOON_MESSAGE =
  "AI Assistant is coming soon! Browse attraction guides in the meantime."
```

真实 AI 对话能力由后续独立 spec（`specs/ai-assistant.md`）定义，届时替换 `handleSend` 逻辑为真实 API 调用。

---

## 5. 与其他单元依赖关系

| 依赖方向 | 说明 |
|----------|------|
| **被依赖** | 无。该组件由布局编排层（page.tsx）直接引入 |
| **依赖其他单元** | 无 |

该单元是完全独立的前端组件。与功能导航栏中 "AI Assistant" 入口卡片的关系：
- 导航栏卡片：点击跳转到 `/ai-assistant` 独立页面
- 悬浮按钮：在任何页面均可唤起聊天面板

两者功能互补，但实现上无代码级依赖。
