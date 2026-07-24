# Git Conventions

## 规则类型: ALWAYS

## 描述

Agent 进行 Git 操作时必须遵循的约定。

## 规则

### 1. 提交信息格式

使用 Conventional Commits 格式：

```
<type>(<scope>): <description>

[optional body]
```

类型（type）:
- `feat`: 新功能
- `fix`: Bug 修复
- `test`: 添加或修改测试
- `refactor`: 重构（不改变功能）
- `docs`: 文档变更
- `chore`: 构建/工具变更

TDD 提交示例：
- `test(auth): [RED] failing test for login validation`
- `feat(auth): [GREEN] minimal impl for login validation`
- `refactor(auth): [REFACTOR] extract validation helper`

### 2. 分支命名

- `feat/<change-id>` — 功能分支
- `fix/<change-id>` — 修复分支
- `refactor/<change-id>` — 重构分支

### 3. 提交粒度

- 每个 RED → GREEN → REFACTOR 循环后提交
- 一个提交只做一件事
- 避免 mega-commit（超大提交）

### 4. 禁止行为

- ❌ 不提交 WIP（Work In Progress）
- ❌ 不修改已推送的提交历史（除非在自己的分支上）
- ❌ 不提交包含密码/密钥的文件
- ❌ 不跳过 hooks（`--no-verify`）
