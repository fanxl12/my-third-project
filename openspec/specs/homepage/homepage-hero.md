# Spec: Homepage — Hero 区

## 1. 模块边界

### 包含的功能

| 功能 | 说明 |
|------|------|
| 品牌主视觉背景图 | 全宽展示，使用占位图片 URL，支持 CSS 背景或 Next.js Image 组件 |
| 品牌标语 | 英文标语（例："Explore China, One City at a Time"），居中叠加在背景图上方 |
| 副标题 | 英文副标题（例："Curated guides, real traveler tips, and instant AI assistance for your China adventure."） |
| 搜索框 UI | 搜索输入框 + 搜索按钮，纯 UI 占位，不实现搜索逻辑 |

### 不包含的功能

- ❌ 搜索逻辑（全文搜索、自动补全、搜索结果页）
- ❌ 背景图轮播 / Banner 广告位
- ❌ 动态内容（标语、副标题为固定文案）
- ❌ 用户交互以外的视觉动效（本期不做复杂动画）

---

## 2. 核心场景

### SC-01: 用户首次进入首页

```
GIVEN 用户访问首页 "/"
WHEN 页面加载完成
THEN 系统展示 Hero 区
  AND 品牌主视觉背景图完整加载（使用 placeholder blur 过渡）
  AND 品牌标语居中显示，字体大小 responsive
  AND 副标题显示在标语下方
  AND 搜索框位于 Hero 区底部，placeholder 文案为 "Search destinations, guides..." 
  AND 搜索框聚焦时显示蓝色边框，失焦恢复默认样式
```

### SC-02: 搜索框交互（仅 UI）

```
GIVEN Hero 区搜索框已渲染
WHEN 用户点击搜索框
THEN 搜索框获得焦点，边框高亮
WHEN 用户输入文字
THEN 搜索框内容更新
WHEN 用户按下回车或点击搜索按钮
THEN 输入框内容清空
  AND 显示 toast 提示 "Search is coming soon!"
```

---

## 3. 组件定义

### 3.1 组件结构

```
HeroSection
├── BackgroundImage       # 品牌主视觉背景
├── HeroContent           # 文字内容层（半透明遮罩）
│   ├── BrandSlogan       # 品牌标语 <h1>
│   ├── Subtitle          # 副标题 <p>
│   └── SearchBar         # 搜索框
│       ├── SearchInput   # <input type="text">
│       └── SearchButton  # <button> 搜索图标
```

### 3.2 组件 Props（SearchBar）

| Prop | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `placeholder` | `string` | `"Search destinations, guides..."` | 搜索框占位文本 |
| `onSearch` | `(query: string) => void` | `undefined` | 搜索回调，本期仅 toast |

---

## 4. Mock 数据契约

Hero 区为纯静态组件，无 API 依赖。所有内容为硬编码常量：

```typescript
// components/homepage/hero/constants.ts
export const HERO_CONTENT = {
  backgroundImage: "/images/hero-bg-placeholder.jpg", // 占位背景图
  slogan: "Explore China, One City at a Time",
  subtitle: "Curated guides, real traveler tips, and instant AI assistance for your China adventure.",
  searchPlaceholder: "Search destinations, guides...",
}
```

背景图规格要求：1920×900px, JPEG/WebP, ≤ 300KB, 文件存放于 `frontend/public/images/`

---

## 5. 与其他单元依赖关系

| 依赖方向 | 说明 |
|----------|------|
| **被依赖** | 无 |
| **依赖其他单元** | 无 |

Hero 区是最顶层的前端组件，不依赖任何其他单元。布局编排层（page.tsx）在组合阶段引入该组件。
