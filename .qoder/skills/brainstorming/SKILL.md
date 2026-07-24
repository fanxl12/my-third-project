# Brainstorming Skill

## 触发条件

- 用户提出模糊或高层次的需求
- 用户说"我想做 X 但不确定怎么做"
- 任何需要在写代码前进行方案设计的场景

## 行为指南

### 1. 需求澄清

在收到需求后，不要立即开始编码。通过 Socratic 式提问澄清：
- 这个功能的用户是谁？他们的核心诉求是什么？
- 成功的标准是什么？如何衡量？
- 有哪些约束条件？（技术栈、时间、性能等）

### 2. 方案探索

- 提出 2-3 个可行的技术方案
- 对每个方案分析利弊（复杂度、可维护性、性能、风险）
- 推荐最优方案并解释原因
- 遵循 YAGNI 原则，选择最简单可行的方案

### 3. 设计输出

将设计写入 `openspec/changes/<change-id>/design.md`，包含：
- **概述**: 变更的高层次描述
- **技术方案**: 推荐的实现路径
- **关键决策**: 为什么这样选择
- **数据模型**（如适用）
- **API 设计**（如适用）
- **组件结构**（如适用）

### 4. 分段展示

- 将设计文档分成小块展示给用户
- 每个部分展示后等待用户确认
- 不要一次性倾倒大量信息

### 5. 创建 OpenSpec 变更目录

在设计被确认后，创建完整的 OpenSpec 变更目录：
```
openspec/changes/<change-id>/
├── proposal.md    # 为什么做、变更什么
├── design.md      # 技术方案（本技能产出）
├── tasks.md       # 实现任务（由 writing-plans 技能填充）
└── specs/         # 规范增量
```

## 输出示例

```
Let me think through this...

What you're building: [一句话总结]
Users: [目标用户]
Success looks like: [验收标准]

Option A: [方案名]
- Pros: ...
- Cons: ...

Option B: [方案名]
- Pros: ...
- Cons: ...

Recommendation: Option X because...

Ready to proceed with design doc?
```
