# AGENTS.md — AI 编码总入口

<!--
  ╔══════════════════════════════════════════════════════════════╗
  ║  本文件是项目级 AI 约束总入口。                             ║
  ║  所有 AI 编码助手（Qoder、Claude Code、Cursor 等）进入项目  ║
  ║  时首先加载本文件，然后按需加载 .qoder/ 下的详细配置。     ║
  ║                                                              ║
  ║  核心理念: Harness + OpenSpec + Superpowers                 ║
  ╚══════════════════════════════════════════════════════════════╝
-->

## 你是这个项目的 AI 编码助手

你的任务是在保证代码质量和一致性的前提下，高效地帮助完成开发工作。

## 核心约束（永远遵守）

### 1. 语言
- 所有对话、注释、文档使用 **中文**
- 代码标识符（变量名、函数名、类名等）使用 **英文**
- Git 提交信息使用 **英文**（Conventional Commits 格式）

### 2. 开发流程（强制）

使用 OpenSpec 快捷指令驱动开发全流程：

```
/opsx-explore <topic>    → 探索需求、分析方案
/opsx-propose <change-id> → 创建变更提案（proposal + design + tasks + specs）
/opsx-apply <change-id>   → 按任务清单逐项实现（TDD）
/opsx-verify <change-id>  → 验证实现（规范合规 + 测试 + 代码质量）
/opsx-archive <change-id> → 归档变更、更新系统规范
```

底层流程:
```
接收需求 → 澄清需求 → 写 spec → 对齐确认 → 写测试(RED) → 写实现(GREEN) → 重构(REFACTOR) → 审查 → 归档
```

**禁止跳步。** 没有 spec 不写代码，没有测试不写实现。

### 3. 规范管理

所有变更在 `openspec/changes/` 下管理：
- `proposal.md` — 为什么要做
- `design.md` — 技术方案
- `tasks.md` — 任务 Checklist
- `specs/` — 规范增量

实现完成后归档到 `openspec/changes/archive/`，更新 `openspec/specs/`。

### 4. 代码标准

- **YAGNI** — 只写当前需要的，不提前设计
- **DRY** — 第 3 次重复时提取抽象
- **TDD** — RED → GREEN → REFACTOR，严格遵守
- **简洁** — 函数 ≤ 30 行，嵌套 ≤ 3 层
- **安全** — 不硬编码密钥，输入必须校验

### 5. 行为准则

- 收到模糊需求时，**先提问澄清**，不要猜测
- 每次修改前，**确认影响范围**
- 完成一个逻辑单元后，**主动审查**
- 遇到错误时，**系统性调试**（复现 → 诊断 → 修复 → 预防）
- 提交代码前，**验证所有测试通过**

### 6. 技术选型约束

<!--
  技术选型决策及核心约束，AI 生成代码时须严格遵循此技术边界。
  选型依据详见本文件末尾与 openspec/project.md。
-->

#### 选型结论

| 层 | 框架 | 部署 |
|----|------|------|
| **前端** | Next.js (React + TypeScript) | Vercel |
| **后端** | Spring Boot 3.4 + JDK 21 + Maven + Spring Data JPA + MySQL + Flyway | Docker + 云服务器 |

#### 选型约束

| 约束 | 值 |
|------|-----|
| 团队规模 | 3 人 |
| 团队技能 | 全员熟悉 React，Spring Boot 开发经验充足 |
| 项目周期 | 8 周上线 MVP |
| 部署环境 | Vercel（前端）/ Docker（后端） |
| 开发范式 | AI 辅助开发为主 |
| 长期定位 | 线上可持续运行迭代 |

#### 选型原则（AI 生成代码时遵循）

1. **前端**：生成 Next.js App Router 代码（Server Components 优先），样式使用 Tailwind CSS 4，状态管理优先用 React Context/useReducer，避免引入第三方 UI 库（除非 spec 明确要求）
2. **后端**：生成 Spring Boot REST API 代码，遵循 Controller → Service → Repository 三层架构，使用 Spring Data JPA 作为 ORM，Flyway 管理数据库迁移，MySQL 8.0+ 作为数据库，API 返回统一的 `ApiResponse<T>` 包装
3. **前后端通信**：前端通过 HTTP REST JSON 调用后端，不混合前端页面与后端逻辑
4. **不越界**：不要在前端子模块生成 Java 代码，不要在后端子模块生成 TSX 代码

## 配置索引

本文件提供全局约束。详细行为规范分布在以下位置：

| 文件/目录 | 用途 | 加载时机 |
|-----------|------|----------|
| `.qoder/AGENTS.md` | Qoder Harness 层配置 | Qoder 会话启动 |
| `.qoder/rules/` | 4 条 Always 规则 | 始终生效 |
| `.qoder/skills/` | 15 个技能（含 OpenSpec 指令） | 按触发条件激活 |
| `openspec/project.md` | 项目上下文（技术栈、架构） | 进入项目时 |
| `openspec/specs/` | 系统规范（事实来源） | 写代码时参考 |
| `openspec/changes/` | 变更记录 & 模板 | 创建新变更时 |

## 项目结构（AIWorkSpace）

本仓库是 **AIWorkSpace 编排层**，采用 Git Submodule 组织子项目：

| 子模块 | 路径 | 技术栈 | 职责 |
|--------|------|--------|------|
| `backend` | `./backend` | Spring Boot 3.4 + Maven + Spring Data JPA + MySQL + Flyway | REST API 后端服务 |
| `frontend` | `./frontend` | Next.js + React + TypeScript + Tailwind CSS 4 | SSR/SSG 前端应用 |

### 子模块工作约定

- 每个子模块拥有**独立的 `.git` 仓库**，在根仓库中以 submodule 引用
- 进入子模块目录后，AI 应像操作独立项目一样读写代码
- **修改子模块代码后**，需先进入子模块目录 `git add && git commit`，再回到根目录提交 submodule 指针变更
- 子模块 URL 当前使用相对路径，后续可替换为远程仓库地址
- 根仓库不直接追踪 `backend/` 和 `frontend/` 下的文件，所有变更由子模块自身管理

### 初始克隆

```bash
git clone --recurse-submodules <repo-url>
# 或已克隆后：
git submodule update --init --recursive
```

## 禁止事项

- ❌ 不创建不必要的文档（README 类、总结类 .md 文件）
- ❌ 不跳过 TDD 流程
- ❌ 不提交包含密钥/令牌的代码
- ❌ 不在未确认 spec 的情况下写实现
- ❌ 不过度设计（YAGNI）
- ❌ 不提交调试代码（console.log 等）
