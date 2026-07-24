# Test-Driven Development Skill

## 触发条件

- 任何编写实现代码的场景（非测试文件）
- 修改已有功能时

## 行为指南

### RED — 先写失败的测试

1. 在写任何实现代码之前，先写测试
2. 测试必须能够运行并且 **失败**（RED）
3. 验证测试失败的原因是正确的（不是因为测试本身有 bug）
4. 如果发现之前写了没有测试的实现代码，**先删除实现代码**，再写测试

### GREEN — 写最小实现

1. 只写让测试通过的最少代码
2. 不要提前设计未来可能需要的功能（YAGNI）
3. 如果测试仍然失败，检查是测试错了还是实现错了
4. 测试通过后立即停止添加代码

### REFACTOR — 改进代码结构

1. 在绿灯状态下重构
2. 改进代码可读性、消除重复（DRY）
3. 重构过程中始终保持测试绿灯
4. 每次重构后运行所有相关测试

### 提交策略

- 每个 RED → GREEN → REFACTOR 循环后提交
- 提交信息格式: `test(scope): [RED] failing test for X` → `feat(scope): [GREEN] minimal impl for X` → `refactor(scope): [REFACTOR] clean up X`

## TDD 反模式（禁止）

| 反模式 | 说明 |
|--------|------|
| 先写代码后补测试 | 违反 TDD 基本规则 |
| 一次写多个测试 | 一次只写一个测试，渐进式推进 |
| 跳过 RED 阶段 | 必须看到测试失败 |
| 过度实现 | 写了超过当前测试需要的代码 |
| 测试依赖外部状态 | 测试必须独立、可重复 |

## 输出格式

每完成一个 TDD 循环后报告：

```
🔴 RED:   Added failing test for [功能]
🟢 GREEN: Implemented [最小实现]
🔵 REFACTOR: [重构内容]
✅ All tests passing. Ready for next cycle.
```
