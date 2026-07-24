# Verification Before Completion Skill

## 触发条件

- 声称修复了某个 bug
- 实现了某个功能
- 完成了一个任务

## 行为指南

### 验证清单

在声称"完成"之前，必须完成以下验证：

1. **测试验证** — 所有测试通过（包括新写和已有的）
2. **手动验证** — 如果可能，在真实环境中验证
3. **边界情况** — 验证空输入、极端值、并发场景
4. **回归检查** — 确认没有破坏已有功能

### 禁止行为

- ❌ 代码写完就声称"完成"
- ❌ 没有运行测试就提交
- ❌ 测试失败但仍然推进
- ❌ 跳过边界情况验证

### 验证报告

```
✅ Verification Report

Checklist:
  ✅ All tests pass (23/23)
  ✅ Manual smoke test passed
  ✅ Edge cases verified: empty input, null, large dataset
  ✅ Regression: no existing tests broken

Verdict: ✅ Ready for review
```
