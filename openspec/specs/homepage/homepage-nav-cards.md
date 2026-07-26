# Spec: Homepage — 功能导航栏

## 1. 模块边界

### 包含的功能

| 功能 | 说明 |
|------|------|
| 旅游社区入口卡片 | 图标 + 标题 "Travel Community" + 描述 + 链接 `/community` |
| 景点攻略入口卡片 | 图标 + 标题 "Attraction Guides" + 描述 + 链接 `/attractions` |
| AI助手入口卡片 | 图标 + 标题 "AI Assistant" + 描述 + 链接 `/ai-assistant` |
| 卡片 hover 效果 | 鼠标悬停时卡片轻微上浮 + 阴影加深 |

### 不包含的功能

- ❌ 卡片动态排序 / 推荐逻辑
- ❌ 卡片内容通过 API 配置（本期为静态）
- ❌ 卡片点击埋点统计（本期不做）
- ❌ 卡片内部嵌入动画

---

## 2. 核心场景

### SC-01: 用户浏览导航栏

```
GIVEN 首页功能导航栏已渲染在 Hero 区下方
WHEN 用户看到导航栏
THEN 展示三张等宽卡片，水平排列（移动端竖向堆叠）
  AND 每张卡片包含：SVG 图标 + 英文标题 + 一句话描述
  AND 卡片顺序固定：旅游社区 → 景点攻略 → AI助手
```

### SC-02: 用户点击入口卡片

```
GIVEN 功能导航栏已渲染
WHEN 用户点击任意卡片
THEN 使用 Next.js <Link> 导航到对应路径
  AND 不触发全页刷新（客户端路由）
```

### SC-03: 移动端适配

```
GIVEN 视口宽度 < 768px
WHEN 功能导航栏渲染
THEN 三张卡片竖向堆叠
  AND 每张卡片宽度 100%
  AND 卡片之间垂直间距 16px
```

---

## 3. 组件定义

### 3.1 组件结构

```
NavCards
├── NavCard (Community)
│   ├── Icon (SVG)
│   ├── Title
│   └── Description
├── NavCard (Attractions)
│   ├── Icon (SVG)
│   ├── Title
│   └── Description
└── NavCard (AI)
    ├── Icon (SVG)
    ├── Title
    └── Description
```

### 3.2 组件 Props

| Prop | 类型 | 说明 |
|------|------|------|
| `icon` | `React.ReactNode` | SVG 图标 |
| `title` | `string` | 英文标题 |
| `description` | `string` | 一句话描述 |
| `href` | `string` | 跳转链接路径 |

---

## 4. 静态卡片内容

```typescript
// components/homepage/nav-cards/constants.ts
export const NAV_CARDS = [
  {
    id: "community",
    title: "Travel Community",
    description: "Ask questions, share experiences, and connect with fellow travelers.",
    href: "/community",
  },
  {
    id: "attractions",
    title: "Attraction Guides",
    description: "Step-by-step guides with booking tips, transport info, and insider advice.",
    href: "/attractions",
  },
  {
    id: "ai-assistant",
    title: "AI Assistant",
    description: "Get instant answers about attractions, visas, and travel tips powered by AI.",
    href: "/ai-assistant",
  },
]
```

图标：每个卡片使用内联 SVG（24×24），颜色统一为品牌色 `#3B82F6`（Tailwind blue-500）。

---

## 5. 与其他单元依赖关系

| 依赖方向 | 说明 |
|----------|------|
| **被依赖** | 无 |
| **依赖其他单元** | 无 |

导航栏是纯前端静态组件，跳转目标路径为占位 URL，对应页面由后续 spec 实现。
