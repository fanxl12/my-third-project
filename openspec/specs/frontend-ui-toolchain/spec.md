# Spec Delta: Frontend UI Toolchain

> 为 WanderChina 前端工程引入 shadcn/ui + lucide-react UI 工具链标准。

---

## ADDED Requirements

### Requirement: UI 组件库选型

The project SHALL use shadcn/ui as the sole UI component library, complemented by lucide-react for icons and Tailwind CSS v4 for atomic styling. No other third-party UI libraries (MUI, Ant Design, Chakra UI, Bootstrap) SHALL be introduced.

#### Scenario: 开发者创建新组件时选择 UI 方案

- **WHEN** 开发者需要为页面添加按钮、输入框、卡片、骨架屏、标签或 Toast 通知
- **THEN** 必须从 `@/components/ui/` 导入对应的 shadcn/ui 组件（Button / Input / Card / Skeleton / Badge / Sonner）
- **AND** 图标必须从 `lucide-react` 导入，禁止手写 SVG
- **AND** 样式通过 Tailwind 原子类 `className` 覆盖，禁止内联 `style={{}}`

#### Scenario: 开发者尝试引入 MUI 或其他 UI 库

- **WHEN** 开发者试图 `pnpm add @mui/material` 或类似第三方 UI 库
- **THEN** 代码审查应拒绝该变更
- **AND** 提示开发者使用已有的 shadcn/ui 组件或通过 `npx shadcn@latest add` 按需添加新组件

### Requirement: shadcn/ui 组件清单

The project SHALL have exactly six shadcn/ui components pre-installed: Button, Input, Sonner, Card, Skeleton, Badge. Additional components SHALL be installed on-demand via `npx shadcn@latest add` when explicitly required by a spec.

#### Scenario: 首页 Hero 区使用 Input 和 Button

- **WHEN** 首页 Hero 区 spec 要求渲染搜索框和搜索按钮
- **THEN** 组件代码从 `@/components/ui/input` 导入 `Input`，从 `@/components/ui/button` 导入 `Button`
- **AND** 搜索按钮包含 `lucide-react` 的 `Search` 图标
- **AND** 点击按钮弹出 `sonner` toast："Search is coming soon!"

#### Scenario: 热门帖子加载骨架屏

- **WHEN** 热门帖子组件等待 mock API 返回数据
- **THEN** 使用 `@/components/ui/skeleton` 渲染 3 个骨架卡片占位
- **AND** Skeleton 带 `aria-busy="true"` 和 shimmer 动画

#### Scenario: 后续组件需要 Dialog 弹窗

- **WHEN** 新的 spec 明确要求使用模态弹窗
- **THEN** 执行 `npx shadcn@latest add dialog` 安装 Dialog 组件
- **AND** 从 `@/components/ui/dialog` 导入使用
- **AND** 不自行实现类似功能

### Requirement: CSS 变量体系

The project SHALL use shadcn/ui's CSS variable system (via Tailwind v4 `@theme inline`) for theming. Global color, radius, and spacing tokens SHALL be defined in `app/globals.css`.

#### Scenario: 全局主题色切换

- **WHEN** 需要将品牌色从默认 slate 改为蓝色系
- **THEN** 修改 `globals.css` 中 `--primary` CSS 变量值
- **AND** 所有 shadcn 组件自动继承新主题色，无需逐个组件修改

#### Scenario: 组件样式覆盖

- **WHEN** 需要定制单个 Button 的颜色
- **THEN** 使用 Tailwind 类名覆盖（如 `className="bg-blue-500 hover:bg-blue-600"`）
- **AND** 禁止修改 `components/ui/button.tsx` 源码（保持可升级性）

### Requirement: AGENTS.md 约束更新

The project's AGENTS.md SHALL reflect the shadcn/ui + lucide-react toolchain as the approved UI layer, and SHALL explicitly prohibit alternative UI libraries, inline styles, and reinventing existing shadcn/ui components.

#### Scenario: AI 编码助手生成前端代码

- **WHEN** AI 编码助手为 WanderChina 生成 TSX 组件代码
- **THEN** 必须使用 shadcn/ui 组件（从 `@/components/ui/` 导入）和 lucide-react 图标
- **AND** 样式使用 Tailwind CSS 类名
- **AND** 禁止生成 `style={{}}` 内联样式
- **AND** 禁止引入 MUI、Ant Design、Chakra UI 等第三方 UI 库
- **AND** 禁止自行实现已有的 shadcn/ui 组件功能（如手写 toast 而非使用 Sonner）

#### Scenario: 代码审查检查 UI 合规性

- **WHEN** reviewer 审查前端 PR
- **THEN** 检查是否引入非 shadcn/ui 的第三方 UI 库依赖
- **AND** 检查是否存在内联 `style={{}}` 对象
- **AND** 检查是否重复实现了 shadcn/ui 已有组件
- **AND** 违反者要求修改后再提交

---

## MODIFIED Requirements

_无修改的已有需求。_

## REMOVED Requirements

_无移除的已有需求。_
