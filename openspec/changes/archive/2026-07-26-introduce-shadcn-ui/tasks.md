# Tasks: introduce-shadcn-ui

## Phase 1: 依赖安装

- [ ] 1.1 安装 lucide-react：`cd frontend && pnpm add lucide-react`
  - 验证：`package.json` 含 `lucide-react` 依赖，`pnpm-lock.yaml` 更新
- [ ] 1.2 初始化 shadcn/ui：`npx shadcn@latest init --defaults`
  - 验证：`components.json` 存在；`lib/utils.ts` 存在且导出 `cn()`；`clsx` + `tailwind-merge` 已安装
- [ ] 1.3 审查 `globals.css`：确认 `@import "tailwindcss"` 未被移除，shadcn CSS 变量正确追加

## Phase 2: 安装 shadcn/ui 组件

- [ ] 2.1 `npx shadcn@latest add input` — 验证：`components/ui/input.tsx` 存在
- [ ] 2.2 `npx shadcn@latest add button` — 验证：`components/ui/button.tsx` 存在
- [ ] 2.3 `npx shadcn@latest add sonner` — 验证：`components/ui/sonner.tsx` 存在
- [ ] 2.4 `npx shadcn@latest add card` — 验证：`components/ui/card.tsx` 存在
- [ ] 2.5 `npx shadcn@latest add skeleton` — 验证：`components/ui/skeleton.tsx` 存在
- [ ] 2.6 `npx shadcn@latest add badge` — 验证：`components/ui/badge.tsx` 存在

## Phase 3: 编译验证

- [ ] 3.1 运行 `pnpm build`：确认编译无错误，无 shadcn/ui 相关 warning
- [ ] 3.2 运行 `pnpm lint`：确认 ESLint 无新增 error
- [ ] 3.3 运行 `pnpm type-check`：确认 TypeScript 类型检查通过

## Phase 4: 更新 AGENTS.md 约束

- [ ] 4.1 更新选型原则第 1 条：允许 shadcn/ui + lucide-react
- [ ] 4.2 更新项目结构表格：前端技术栈含 shadcn/ui + lucide-react
- [ ] 4.3 禁止事项清单新增 3 条（禁止其他 UI 库、禁止内联 style、禁止重复造轮子）

## Phase 5: 提交

- [ ] 5.1 frontend 子模块提交：`feat: introduce shadcn/ui and lucide-react`
- [ ] 5.2 根仓库提交：`feat: add shadcn/ui toolchain spec + update AGENTS.md`

---

**预计总耗时**：~15 分钟（安装 5 分钟 + 验证 5 分钟 + AGENTS.md 编辑 5 分钟）
