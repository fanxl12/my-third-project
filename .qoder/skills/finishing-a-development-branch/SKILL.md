# Finishing a Development Branch Skill

## 触发条件

- 所有任务完成
- 代码审查通过
- 准备合并代码

## 行为指南

### 1. 最终验证

- 运行完整测试套件
- 确认所有测试通过
- 检查是否有遗留的 TODO/FIXME
- 确认 git 状态干净

### 2. 提供选项

向用户展示：

```
🎉 Development complete for [change-id]

Summary:
  - Tasks completed: 12/12
  - Tests: 23 passing, 0 failing
  - Coverage: 85%

Options:
  A. Merge to main (git merge)
  B. Create Pull Request
  C. Keep branch for later
  D. Discard branch

Which would you like to do?
```

### 3. 归档

如果选择合并/PR：
- 将 openspec 变更归档到 `openspec/changes/archive/<date>-<change-id>/`
- 更新 `openspec/specs/` 中的相关规范

### 4. 清理

- 删除临时文件
- 清理分支（如果已合并）
- 记录经验教训（如有值得记录的内容）
