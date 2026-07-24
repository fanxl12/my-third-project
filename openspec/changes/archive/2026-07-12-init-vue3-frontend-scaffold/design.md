# Design: init-vue3-frontend-scaffold

## 概述

搭建 Vue 3 前端骨架，Vite 构建（对标 Maven Wrapper），TypeScript，最小可用，通过代理对接后端。

## 技术决策

| 决策 | 选项 | 理由 |
|------|------|------|
| 框架 | Vue 3 (Composition API + `<script setup>`) | 团队 Vue 生态偏好 |
| 语言 | TypeScript 5.x | 类型安全，Vue 3 原生集成 |
| 构建工具 | Vite 6 | 极速 HMR，Vue 官方推荐 |
| 包管理 | pnpm + Corepack | 免预装，对标 mvnw，Vite 官方推荐，无幽灵依赖 |
| 组件库 | Element Plus | Vue 3 最成熟组件库，表格/表单丰富 |
| 样式 | Tailwind CSS 4 | 原子化 + 组件库互补 |
| 路由 | Vue Router 4 | SPA 标配 |
| 状态管理 | Pinia | Vue 官方推荐，轻量 |
| HTTP | Axios | 对接 Spring Boot JSON API |
| 代码风格 | ESLint + Prettier | 统一团队风格 |

## 目录结构

```
frontend/
├── .npmrc                          # pnpm shamefully-hoist 配置
├── eslint.config.js                # ESLint 扁平配置
├── .prettierrc                     # Prettier 配置
├── index.html                      # Vite 入口 HTML
├── package.json                    # 依赖声明
├── pnpm-lock.yaml                  # 锁定文件
├── tsconfig.json                   # TypeScript 配置
├── vite.config.ts                  # Vite 配置（含代理）
└── src/
    ├── App.vue                     # 根组件
    ├── main.ts                     # 应用入口
    ├── router/
    │   └── index.ts                # Vue Router 配置
    ├── stores/
    │   └── counter.ts              # Pinia 示例 store
    ├── views/
    │   └── HomeView.vue            # 首页视图
    ├── components/
    │   └── HelloWorld.vue          # 示例组件
    ├── api/
    │   └── index.ts                # Axios 实例 + /api/health 请求
    └── style.css                   # Tailwind 入口
```

## 关键配置

### package.json scripts
```json
{
  "scripts": {
    "dev": "vite",
    "build": "vue-tsc -b && vite build",
    "preview": "vite preview"
  },
  "packageManager": "pnpm@10.x"
}
```

### vite.config.ts 代理
```ts
export default defineConfig({
  server: {
    proxy: {
      '/api': 'http://localhost:8080'
    }
  }
})
```

### 入口页面
- `localhost:5173/` → 首页（展示与后端的 `/api/health` 联调结果）
- 使用 Element Plus 按钮 + Tailwind 布局样式验证集成

## 不包含（YAGNI）

- ❌ 后台管理布局（侧边栏/顶部导航）
- ❌ 登录/权限
- ❌ Mock 服务
- ❌ 国际化 (i18n)
- ❌ PWA
- ❌ 测试框架（后续按需引入 Vitest）
