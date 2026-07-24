# /opsx:apply — 执行实现

## 触发条件

- 用户输入 `/opsx:apply <change-id>`
- 提案审核通过，准备实现
- 用户说"开始实现"、"apply"

## 行为指南

### 1. 加载变更上下文

1. 读取 `openspec/changes/<change-id>/proposal.md` — 理解动机和范围
2. 读取 `openspec/changes/<change-id>/design.md` — 理解技术方案
3. 读取 `openspec/changes/<change-id>/tasks.md` — 获取任务清单

### 2. 按任务清单逐项执行

严格按 `tasks.md` 中的顺序执行，每个任务遵循 TDD：

```
对每个任务:
  1. 读取任务描述和验证步骤
  2. [RED]   写失败的测试
  3. [GREEN] 写最小实现让测试通过
  4. [REFACTOR] 重构优化
  5. 验证: 所有测试通过
  6. 标记任务为 ✅
  7. 向用户报告进度
```

### 3. 实现约束

- **严格遵循设计文档** — 不偏离 design.md 中的技术方案
- **TDD 强制** — 每个任务先写测试，不允许跳过
- **最小实现** — 只写让测试通过的代码（YAGNI）
- **保持简洁** — 函数 ≤ 30 行，嵌套 ≤ 3 层

### 4. 进度报告

每完成一个任务后报告：

```
🔄 执行中: <change-id>

Phase 1: Setup
  ✅ 1.1 创建测试文件
  ✅ 1.2 [RED] 写失败测试

Phase 2: Core
  🔄 2.1 [GREEN] 最小实现...
  ⬜ 2.2 [REFACTOR] 重构

📊 进度: 3/8 任务完成
```

### 5. 遇到问题

如果任务无法按计划执行：
- 记录偏差原因
- 更新 design.md 或 tasks.md
- 如果偏差较大，暂停并告知用户

### 6. 全部完成

```
✅ 实现完成: <change-id>

📊 摘要:
  - 任务: 8/8 完成
  - 测试: 12 个通过, 0 个失败
  - 新增文件: 5 个
  - 修改文件: 2 个

📝 下一步: 输入 /opsx:verify <change-id> 验证实现
          输入 /opsx:archive <change-id> 归档变更
```

## 关联技能

- 实现完成 → `/opsx:verify` → `/opsx:archive`
- 本技能内部调用 `test-driven-development` 和 `subagent-driven-development`
