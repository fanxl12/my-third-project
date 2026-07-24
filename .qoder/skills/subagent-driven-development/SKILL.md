# Subagent-Driven Development Skill

## 触发条件

- 任务清单已准备完毕
- 用户说"开始执行"

## 行为指南

### 1. 调度策略

- 每个任务分配一个独立的 subagent 执行
- subagent 只能看到当前任务上下文
- 按依赖顺序执行，无依赖的任务可并行

### 2. 两阶段审查

每个 subagent 完成任务后，进行两阶段审查：

**阶段 1 — 规范合规审查:**
- 实现是否符合设计文档？
- 是否符合任务要求？
- 是否有遗漏的验收标准？

**阶段 2 — 代码质量审查:**
- 代码是否可读、可维护？
- 是否遵循项目编码规范？
- 是否有潜在的性能问题或安全风险？
- 测试是否充分？

### 3. 失败处理

- 如果审查发现 critical 问题，**阻塞**执行
- 修复后重新审查
- 如果连续 3 次不通过，人工介入

### 4. 进度报告

每完成一个任务后报告：

```
✅ Task 1.2 complete: Write failing test for basic case
🔍 Review Stage 1: ✅ Spec compliant
🔍 Review Stage 2: ✅ Code quality OK
📊 Progress: 2/12 tasks complete
▶️  Starting Task 2.1: Implement minimal solution
```
