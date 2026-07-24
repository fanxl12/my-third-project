# /opsx:archive — 归档变更

## 触发条件

- 用户输入 `/opsx:archive <change-id>`
- 实现完成并验证通过
- 变更已合并到主分支

## 行为指南

### 1. 归档前检查

确认以下条件满足：
- 所有任务在 tasks.md 中标记为完成
- 所有测试通过
- 代码已合并到主分支（或 PR 已合并）
- specs/ 中的规范增量已确认

### 2. 执行归档

将变更目录移动到归档：

```
openspec/changes/<change-id>/
  → openspec/changes/archive/<YYYY-MM-DD>-<change-id>/
```

日期格式: `YYYY-MM-DD`，取当前日期。

### 3. 更新规范

如果变更涉及系统规范的修改：
1. 将 `openspec/changes/archive/<date>-<change-id>/specs/` 中的规范
2. 合并到 `openspec/specs/` 对应的文件中
3. 如果 specs/ 下还没有对应文件，直接复制过去
4. 确保 specs/ 始终反映当前系统状态

### 4. 输出

```
📦 归档完成: <change-id>

✅ 变更已移至: openspec/changes/archive/2026-07-12-<change-id>/
✅ 规范已更新: openspec/specs/
   - 新增: auth-api.md
   - 更新: data-model.md

📊 本次变更摘要:
  - 提案: [一句话]
  - 新增文件: N 个
  - 修改文件: M 个
  - 测试: 全部通过

🎉 变更已归档，可以开始下一个功能了。
   输入 /opsx:explore <新需求> 开始探索。
```

## 关联技能

- 归档完成 → 可以开始新的 `/opsx:explore`
- 本技能与 `finishing-a-development-branch` 互补：archive 侧重文件归档，finishing-branch 侧重 git 分支操作
