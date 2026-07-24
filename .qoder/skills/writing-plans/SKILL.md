# Writing Plans Skill

## 触发条件

- 设计文档已确认，准备开始实现
- 用户说"开始实现"、"执行计划"等

## 行为指南

### 1. 任务分解原则

将设计分解为可独立执行的小任务：
- 每个任务 **2-5 分钟** 完成
- 每个任务是 **可验证的**（有明确的完成标准）
- 任务是 **有序的**（考虑依赖关系）
- 遵循 TDD：每个实现任务前有对应的测试任务

### 2. 任务规范

每个任务必须包含：
- **任务 ID**: 如 `1.1`, `1.2`, `2.1`
- **描述**: 一句话说明要做什么
- **文件路径**: 精确到文件
- **验证步骤**: 如何确认任务完成
- **依赖**: 前置任务 ID（如有）

### 3. 任务文件格式

将任务清单写入 `openspec/changes/<change-id>/tasks.md`：

```markdown
# Tasks for <change-id>

## 1. Setup

- [ ] 1.1 Create test file `src/__tests__/feature.test.ts`
  - Verify: File exists, test runner recognizes it
  
- [ ] 1.2 [RED] Write failing test for basic case
  - Verify: `npm test -- feature.test` fails with expected error

## 2. Core Implementation

- [ ] 2.1 [GREEN] Implement minimal solution
  - Depends on: 1.2
  - Verify: `npm test -- feature.test` passes

- [ ] 2.2 [REFACTOR] Clean up implementation
  - Depends on: 2.1
  - Verify: All tests still pass
```

### 4. 输出格式

任务清单编写完成后，展示摘要：

```
📋 Implementation Plan for [change-id]

Total tasks: N
Estimated time: ~M minutes

Phase 1: Setup (tasks 1.1-1.2)
Phase 2: Core (tasks 2.1-2.2)
Phase 3: Polish (tasks 3.1-3.3)

Ready to execute? Use subagent-driven-development or executing-plans.
```
