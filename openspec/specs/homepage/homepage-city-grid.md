# Spec: Homepage — 城市快速入口

## 1. 模块边界

### 包含的功能

| 功能 | 说明 |
|------|------|
| 8 城图标网格 | 展示 8 个 MVP 城市的图标网格，每格含城市图标和英文名称 |
| 城市跳转 | 点击城市进入 `/city/{cityId}` 攻略页（占位 URL） |
| hover 交互 | 鼠标悬停时城市图标放大 + 名称变色 |

### 不包含的功能

- ❌ 城市动态排序 / 推荐
- ❌ 城市列表 API 化（本期为静态常量）
- ❌ 城市攻略页面（仅提供 Link 占位）
- ❌ 城市搜索 / 筛选
- ❌ 城市图片懒加载以外的优化

---

## 2. 核心场景

### SC-01: 城市网格渲染

```
GIVEN 城市快速入口组件挂载
WHEN 页面渲染完成
THEN 展示 "🌆 Explore by City" 标题
  AND 展示 8 个城市图标，按 4 列 grid 排列（桌面端）
  AND 每个图标包含：城市特色 SVG 图标 + 英文名称
  AND 城市顺序固定：
      Beijing, Shanghai, Xi'an, Chengdu,
      Guangzhou, Guilin, Lijiang, Hangzhou
```

### SC-02: 城市点击跳转

```
GIVEN 城市网格已渲染
WHEN 用户点击任意城市图标
THEN 导航到 "/city/{cityId}"（使用 Next.js <Link>，客户端路由）
```

### SC-03: 移动端适配

```
GIVEN 视口宽度 < 768px
WHEN 城市网格渲染
THEN 展示 4 列 grid → 调整为 3 列
  AND 每个图标宽度自适应
  AND 图标尺寸从 48px 缩小至 36px
```

---

## 3. 组件定义

### 3.1 组件结构

```
CityGrid
├── SectionTitle            # "🌆 Explore by City"
├── CityGridContainer       # CSS Grid 容器
│   └── CityIcon[] × 8     # 城市图标
│       ├── Icon (SVG)      # 城市地标 SVG 图标
│       ├── CityName        # 英文城市名
│       └── Link            # <Link href="/city/{id}">
```

### 3.2 组件 Props

| Prop | 类型 | 说明 |
|------|------|------|
| `cities` | `CityEntry[]` | 城市列表（默认 8 个 MVP 城市） |

```typescript
interface CityEntry {
  id: string         // "beijing"
  nameEn: string     // "Beijing"
  slug: string       // URL slug
}
```

---

## 4. 静态城市数据

```typescript
// components/homepage/city-grid/constants.ts
export const MVP_CITIES: CityEntry[] = [
  { id: "beijing",    nameEn: "Beijing",    slug: "beijing" },
  { id: "shanghai",   nameEn: "Shanghai",   slug: "shanghai" },
  { id: "xian",       nameEn: "Xi'an",      slug: "xian" },
  { id: "chengdu",    nameEn: "Chengdu",    slug: "chengdu" },
  { id: "guangzhou",  nameEn: "Guangzhou",  slug: "guangzhou" },
  { id: "guilin",     nameEn: "Guilin",     slug: "guilin" },
  { id: "lijiang",    nameEn: "Lijiang",    slug: "lijiang" },
  { id: "hangzhou",   nameEn: "Hangzhou",   slug: "hangzhou" },
]
```

每个城市的 SVG 图标采用地标简笔画风格（24×24），对应文件：
```
frontend/public/images/cities/beijing.svg
frontend/public/images/cities/shanghai.svg
...
```

如 SVG 图标缺失，使用首字母圆形占位图作为 fallback。

---

## 5. 与其他单元依赖关系

| 依赖方向 | 说明 |
|----------|------|
| **被依赖** | 无 |
| **依赖其他单元** | 无 |

城市列表为静态常量，跳转目标为占位 URL，后续可通过配置文件或 CMS 扩展。
