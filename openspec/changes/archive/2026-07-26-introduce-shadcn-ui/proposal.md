# Proposal: introduce-shadcn-ui

## Why

首页 7 个 spec 单元（Hero、NavCards、HotPosts、HotAttractions、CityGrid、AiEntry、Layout）明确要求基于 **shadcn/ui** 组件库 + **lucide-react** 图标库构建 UI：

- **Hero 区** 需要 `Input`、`Button`、`Sonner`（toast）
- **导航栏** 需要 `Card` + lucide-react `MessageCircle`/`MapPin`/`Bot`
- **热门帖子** 需要 `Card`、`Skeleton`、`Badge`、`Button`
- **热门景点** 需要 `Card`、`Skeleton`、`Badge`
- **AI 入口** 需要 `Button`、`Input`、`Sonner`

当前前端工程状态：Tailwind CSS v4 ✅ 已安装；shadcn/ui ❌ 未初始化；lucide-react ❌ 未安装。

此外，`AGENTS.md` 选型原则第 1 条中"避免引入第三方 UI 库"的表述需要更新为"使用 shadcn/ui + lucide-react"，禁止事项清单需要补充。

## 变更了什么？

1. **初始化 shadcn/ui**（`npx shadcn@latest init`），生成 `components.json` + `lib/utils.ts`
2. **安装 shadcn/ui 组件**：`input`、`button`、`sonner`、`card`、`skeleton`、`badge`
3. **安装 lucide-react**（`pnpm add lucide-react`）图标库
4. **更新 AGENTS.md**：
   - 选型原则第 1 条：明确 shadcn/ui + lucide-react 为允许的 UI 库
   - 项目结构表格：更新前端技术栈描述
   - 禁止事项清单：新增 3 条约束

## 影响范围

| 影响对象 | 变更类型 |
|----------|----------|
| `frontend/package.json` | 新增 `lucide-react` 依赖 |
| `frontend/components.json` | 新增（shadcn/ui 配置文件） |
| `frontend/lib/utils.ts` | 新增（`cn()` 工具函数） |
| `frontend/components/ui/` | 新增目录（6 个 shadcn 组件） |
| `AGENTS.md` | 修改选型原则 + 禁止清单 |

## 风险评估

| 风险 | 影响 | 缓解措施 |
|------|------|----------|
| shadcn/ui v2 + Tailwind CSS v4 兼容性 | shadcn 组件样式与 Tailwind v4 语法不一致 | shadcn 官方已支持 Tailwind v4（canary channel），初始化时指定 `--cwd` 确保配置正确 |
| lucide-react 与 React 19 兼容 | 运行时类型错误 | lucide-react 0.462+ 已支持 React 19 |
| shadcn/ui 初始化覆盖已有配置 | postcss/tailwind 配置被修改 | 备份并对比已有配置，手动合并而非全量覆盖 |

## 验收标准

- [ ] `pnpm add lucide-react` 成功安装，`package.json` 含 `lucide-react` 依赖
- [ ] `npx shadcn@latest init` 生成 `components.json`（使用默认配置：TypeScript + Tailwind v4 + CSS variables）
- [ ] `npx shadcn@latest add input button sonner card skeleton badge` 成功添加 6 个组件到 `components/ui/`
- [ ] `frontend/lib/utils.ts` 文件存在，导出 `cn()` 函数
- [ ] `pnpm build` 编译成功，无 shadcn/ui 相关错误
- [ ] `AGENTS.md` 选型原则更新为明确允许 shadcn/ui + lucide-react
- [ ] `AGENTS.md` 禁止清单新增 3 条约束
