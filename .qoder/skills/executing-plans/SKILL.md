# Executing Plans Skill

## 触发条件

- 有已批准的实施计划
- 替代 subagent-driven-development 的轻量执行模式

## 行为指南

### 1. 批量执行

- 将任务按阶段分组，逐批执行
- 每批执行前确认阶段目标
- 每批执行后设置人工检查点

### 2. 检查点规则

每个检查点：
- 运行所有相关测试
- 展示进度和完成情况
- 等待用户确认后继续下一批

### 3. 与 TDD 集成

- 每个任务严格遵循 RED-GREEN-REFACTOR
- 不跳过测试阶段
- 批处理不意味着跳过验证

## 输出格式

```
📦 Batch 1/3: Setup Phase
  ✅ 1.1 Create test file
  ✅ 1.2 Write failing test
  ───── Checkpoint ─────
  Tests: 1 failing (expected)
  Continue to Batch 2? [Y/n]

📦 Batch 2/3: Core Implementation
  ✅ 2.1 Implement minimal solution
  ✅ 2.2 Refactor
  ───── Checkpoint ─────
  Tests: 5 passing, 0 failing
  Continue to Batch 3? [Y/n]
```
