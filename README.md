# my-third-project

基于 **Harness + OpenSpec + Superpowers** 三位一体方法论构建的 AI 编码全栈开发项目。

## 核心理念

| 层级 | 理念 | 说明 |
|------|------|------|
| **Harness** | Agent 运行环境 | Qoder IDE 作为 AI 编码助手的执行载体，配置在 `.qoder/` |
| **OpenSpec** | 规范驱动开发 (SDD) | 先写 spec 对齐需求，再写代码。所有变更在 `openspec/` 下管理 |
| **Superpowers** | Agent 技能体系 | 可组合的技能库驱动标准化开发流程，位于 `.qoder/skills/` |

## 项目结构

```
.
├── AGENTS.md                     # AI 编码助手总约束入口（所有 AI 必读）
├── README.md                     # 本文件 — 项目说明（给人看）
├── .gitignore
│
├── openspec/                     # OpenSpec — 规范驱动开发
│   ├── project.md                #   项目概览、技术栈、架构决策
│   ├── specs/                    #   系统规范（事实来源）
│   └── changes/                  #   变更提案 & 归档
│       ├── TEMPLATE.md           #     变更模板
│       └── archive/              #     已完成变更
│
└── .qoder/                       # Qoder Harness — Agent 配置
    ├── AGENTS.md                 #   Qoder 专用 agent 配置
    ├── rules/                    #   Agent 行为规则（4 条，始终生效）
    │   ├── tdd-rule.md           #     强制 TDD
    │   ├── spec-first-rule.md    #     规范先行
    │   ├── code-quality-rule.md  #     YAGNI / DRY / 安全
    │   └── git-conventions.md    #     Conventional Commits
    └── skills/                   #   Superpowers 技能库（10 个）
        ├── brainstorming/        #     需求澄清 & 方案设计
        ├── writing-plans/        #     任务分解
        ├── executing-plans/      #     批量执行 + 检查点
        ├── subagent-driven-dev/  #     Subagent 并行开发
        ├── test-driven-dev/      #     RED → GREEN → REFACTOR
        ├── systematic-debugging/ #     四阶段根因分析
        ├── requesting-code-review/#    两阶段代码审查
        ├── finishing-branch/     #     分支收尾 & 归档
        ├── verification/         #     完成前强制验证
        └── writing-skills/       #     技能编写指南
```

## 开发工作流

```
需求输入
  ↓
Brainstorming     → 澄清需求，输出 design.md
  ↓
Writing Plans     → 分解为 2-5 分钟的小任务
  ↓
Subagent-Driven   → 独立 subagent 执行每个任务
  ↓   ↑
  ├── TDD          → RED → GREEN → REFACTOR（强制）
  ├── Code Review  → 规范合规 + 代码质量（两阶段）
  └── Debug        → 复现 → 诊断 → 修复 → 预防
  ↓
Verification      → 测试、边界、回归全通过
  ↓
Finish & Archive  → 归档变更，更新 specs
```

## 对 AI 编码助手的约束

参见 [AGENTS.md](./AGENTS.md) —— **所有 AI 工具进入项目时的第一入口**。

核心原则：
- **Spec First** — 先写 spec 再写代码
- **TDD** — 先写测试再写实现
- **YAGNI / DRY** — 简洁、不重复、不过度设计

## 快速开始

1. 阅读 [AGENTS.md](./AGENTS.md) 了解 AI 行为约束
2. 阅读 [openspec/project.md](./openspec/project.md) 了解项目技术栈和架构
3. 使用 brainstorming 技能讨论你的需求
4. 按工作流推进开发

## 参考

- [OpenSpec](https://github.com/Fission-AI/openspec) — Spec-driven development for AI coding assistants
- [Superpowers](https://github.com/obra/superpowers) — Agentic skills framework & software development methodology
