# Requesting Code Review Skill

## 触发条件

- 完成一个任务或一组任务后
- 提交 PR 之前
- 用户说"review 一下"

## 行为指南

### 审查维度

**1. 规范合规（与计划对比）:**
- 实现是否匹配 `openspec/changes/<change-id>/design.md`？
- 任务是否全部完成？
- 是否有遗漏的验收标准？

**2. 代码质量:**
- 可读性：命名清晰、逻辑易懂
- 可维护性：模块化、低耦合
- 性能：无明显性能陷阱
- 安全：无常见安全漏洞

**3. 测试:**
- 测试是否覆盖核心逻辑？
- 是否有边界情况测试？
- 测试是否独立、可重复？

### 问题分级

| 级别 | 说明 | 处理 |
|------|------|------|
| 🔴 Critical | 阻塞合并（bug、安全漏洞、设计偏差） | 必须修复 |
| 🟡 Warning | 建议改进（可读性、潜在问题） | 建议修复 |
| 🟢 Info | 可选的优化建议 | 自行决定 |

### 审查输出

```
📋 Code Review: [change-id]

🔴 Critical (0)
  (none)

🟡 Warning (2)
  - [file.ts:L42] 函数过长，建议拆分
  - [file.ts:L78] 缺少错误处理

🟢 Info (1)
  - [file.ts:L15] 可以考虑使用 const 替代 let

📊 Summary: 0 critical, 2 warnings, 1 info
🔄 Action: Warnings should be addressed before proceeding.
```
