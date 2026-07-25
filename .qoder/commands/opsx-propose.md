---
description: 创建变更提案 — proposal + design + tasks + specs
---

# /opsx-propose — 创建变更提案

## 触发条件

- 用户输入 `/opsx-propose <change-id>`
- 探索完成后，用户确认要创建正式提案
- 用户说"创建提案"、"写 spec"

## 行为指南

### 1. 创建变更目录

在 `openspec/changes/<change-id>/` 下创建完整的变更目录结构，**必须同时创建 `specs/` 子目录**：

```
openspec/changes/<change-id>/
├── proposal.md    # 为什么做、变更什么
├── design.md      # 技术方案
├── tasks.md       # 实现任务清单
└── specs/         # 此变更的规范增量（强制！）
```

### 2. 编写 proposal.md

```markdown
# Proposal: <change-id>

## 为什么做这个变更？
<!-- 业务背景、用户痛点、动机 -->

## 变更了什么？
<!-- 高层次的变更描述 -->

## 影响范围
<!-- 受影响的模块、API、数据模型 -->
- 

## 风险评估
| 风险 | 影响 | 缓解措施 |
|------|------|----------|
|      |      |          |

## 验收标准
- [ ] 
```

### 3. 编写 design.md

```markdown
# Design: <change-id>

## 概述
<!-- 技术方案一句话总结 -->

## 技术方案
<!-- 推荐的实现路径，为什么这样选 -->

## 关键决策
<!-- 记录重要的技术决策及原因 -->

## 数据模型（如适用）
<!-- 新增/修改的数据结构 -->

## API 设计（如适用）
<!-- 接口定义 -->

## 组件结构（如适用）
<!-- 前端组件树 -->
```

### 4. 编写 tasks.md

将设计分解为可执行任务：

```markdown
# Tasks: <change-id>

## Phase 1: Setup
- [ ] 1.1 创建测试文件 ...
- [ ] 1.2 [RED] 写失败测试 ...

## Phase 2: Core
- [ ] 2.1 [GREEN] 最小实现 ...
- [ ] 2.2 [REFACTOR] 重构 ...

## Phase 3: Polish
- [ ] 3.1 边界测试 ...
```

### 5. 输出确认

```
📋 变更提案已创建: openspec/changes/<change-id>/

✅ proposal.md — [摘要]
✅ design.md   — [技术方案摘要]
✅ tasks.md    — N 个任务，预计 ~M 分钟
✅ specs/      — 规范增量已就绪

🔍 请审阅以上文件，确认后输入 /opsx-apply <change-id> 开始实现
```

## 关联指令

- 提案确认后 → `/opsx-apply`
