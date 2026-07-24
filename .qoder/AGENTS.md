# Qoder Agent Configuration

<!--
  Harness 层 — Qoder IDE 的 Agent 配置入口。
  整合 OpenSpec (Spec-Driven Development) + Superpowers (Agent Skills) 理念。
  
  每次会话启动时，Qoder 会加载此文件作为 agent 的系统指令。
  文件引用 .qoder/skills/ 和 .qoder/rules/ 中的技能和规则。
-->

## 核心理念

本项目采用 **Harness + OpenSpec + Superpowers** 三位一体的 AI 编码方法论：

1. **Harness (.qoder/)** — Agent 运行环境和配置层，Qoder IDE 作为执行载体
2. **OpenSpec (openspec/)** — 规范驱动开发，先对齐需求再写代码
3. **Superpowers (.qoder/skills/)** — 可组合的技能库，驱动标准化开发流程

## 工作流程

当收到开发需求时，按以下流程执行：

**OpenSpec 快捷指令流：**
```
/opsx:explore  →  /opsx:propose  →  /opsx:apply  →  /opsx:verify  →  /opsx:archive
```

**底层技能流：**
```
需求输入
  ↓
[/opsx:explore]      ← 探索代码库、分析方案（对应 brainstorming）
  ↓
[/opsx:propose]      ← 创建 openspec/changes/<id>/（proposal + design + tasks）
  ↓
[/opsx:apply]        ← 按 tasks.md 逐项 TDD 实现
  ↓  ↑
  ├── [test-driven-dev]  ← RED-GREEN-REFACTOR 循环
  ├── [code-review]      ← 两阶段审查（规范合规 + 代码质量）
  └── [systematic-debug] ← 遇到问题时系统化调试
  ↓
[/opsx:verify]       ← 规范合规 + 测试 + 代码质量验证
  ↓
[/opsx:archive]      ← 归档到 archive/、更新 specs/

## 核心原则

- **Spec First, Code Second**: 永远先写 spec 对齐，再写代码
- **TDD (RED-GREEN-REFACTOR)**: 先写失败的测试，再写最小实现，最后重构
- **YAGNI**: 不需要的功能不写
- **DRY**: 保持代码简洁，消除重复
- **Systematic Over Ad-Hoc**: 系统化流程优于临时应对
- **Evidence Over Claims**: 用测试结果说话

## 技能清单

Agent 在执行任务前会检查相关技能。以下是配置的技能列表：

| 技能 | 触发时机 | 描述 |
|------|----------|------|
| **OpenSpec 快捷指令** |||
| `/opsx:explore` | 收到新需求时 | 探索代码库、分析方案、输出推荐路径 |
| `/opsx:propose` | 方案确认后 | 创建 openspec 变更目录（proposal/design/tasks/specs） |
| `/opsx:apply` | 提案审阅通过后 | 按 tasks.md 逐项 TDD 实现 |
| `/opsx:verify` | 实现完成后 | 规范合规 + 测试 + 代码质量三维验证 |
| `/opsx:archive` | 验证通过后 | 归档变更到 archive/、更新 specs/ |
| **底层技能** |||
| brainstorming | 收到模糊需求时 | Socratic 式需求澄清与方案设计 |
| writing-plans | 设计确认后 | 将设计分解为可执行的小任务 |
| executing-plans | 有实施计划时 | 批量执行任务并设检查点 |
| subagent-driven-development | 有任务清单时 | 每个任务独立 subagent + 两阶段审查 |
| test-driven-development | 编写代码时 | RED-GREEN-REFACTOR 循环 |
| systematic-debugging | 遇到问题时 | 四阶段根因分析 |
| requesting-code-review | 任务完成后 | 计划合规 + 代码质量审查 |
| finishing-a-development-branch | 所有任务完成时 | 测试验证、合并/PR 决策 |
| verification-before-completion | 修复完成时 | 确认修复真正有效 |
| writing-skills | 创建新技能时 | 按最佳实践编写技能 |

## 规则清单

Agent 始终遵循以下规则（详见 `.qoder/rules/`）：

- [TDD Rule](.qoder/rules/tdd-rule.md) — 强制测试驱动开发
- [Spec-First Rule](.qoder/rules/spec-first-rule.md) — 规范先行
- [Code Quality Rule](.qoder/rules/code-quality-rule.md) — 代码质量标准
- [Git Conventions](.qoder/rules/git-conventions.md) — Git 工作流约定
