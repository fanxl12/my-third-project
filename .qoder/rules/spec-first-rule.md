# Spec-First Rule

## 规则类型: ALWAYS

## 描述

Agent 在写代码之前必须先对齐需求，遵循 OpenSpec 的规范驱动开发（SDD）流程。

## 规则

### 1. 需求对齐

- 收到开发任务后，先澄清需求，不直接写代码
- 模糊的需求触发 brainstorming 技能
- 确保理解"做什么"和"为什么做"之后再动手

### 2. 规格文档

- 每个变更必须在 `openspec/changes/<change-id>/` 下有：
  - `proposal.md` — 为什么做、变更什么
  - `design.md` — 技术方案
  - `tasks.md` — 实现任务清单
  - `specs/` — 规范增量（如涉及 API/数据模型变更）

### 3. 变更归档

- 完成的变更归档到 `openspec/changes/archive/`
- 归档时更新 `openspec/specs/` 中的对应规范
- 保持 specs 目录始终反映当前系统状态

### 4. 禁止行为

- ❌ 没有写 proposal 就开始编码
- ❌ 没有对齐设计就开始实现
- ❌ 跳过 design 直接凭经验写代码
- ❌ 完成的变更不归档
