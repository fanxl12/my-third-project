# Project Overview

<!--
  项目概览文档 — OpenSpec 的入口。
  描述项目的目标、技术栈、约定和架构决策。
  这是 AI 编码助手理解项目上下文的核心文档。
-->

## 项目名称

my-third-project

## 项目简介

（在此描述项目的核心目标和业务背景）

## 技术栈

<!-- 随项目演进而更新此列表 -->

- **前端**: Vue 3 + TypeScript + Vite 6 + Element Plus + Tailwind CSS 4
- **后端**: Java 21 + Spring Boot 3.4 + Maven
- **数据库**: （待定 — 暂无持久化需求）
- **基础设施**: （待定）

## 架构决策

<!-- 记录重要的架构决策 (ADR) -->

| 决策 | 日期 | 状态 | 描述 |
|------|------|------|------|
| 采用 OpenSpec SDD | 2026-07 | ✅ 已采纳 | 使用 Spec-Driven Development 工作流 |
| 采用 Superpowers 技能体系 | 2026-07 | ✅ 已采纳 | 使用 agent skills 驱动开发流程 |
| 前后端分离 + 平级目录 | 2026-07 | ✅ 已采纳 | `backend/` + `frontend/` 同级独立构建 |
| 前端 Vue 3 + Vite + Element Plus + Tailwind | 2026-07 | ✅ 已采纳 | 前端框架选型 |
| Spring Boot 3.4 + JDK 21 | 2026-07 | ✅ 已采纳 | 后端框架选型 |
| Maven Wrapper | 2026-07 | ✅ 已采纳 | 构建工具，无需预装 Maven |

## 开发约定

- **Spec First**: 先写 spec，对齐需求，再写代码
- **TDD**: 测试驱动开发，RED → GREEN → REFACTOR
- **YAGNI**: 不要过度设计
- **DRY**: 保持代码简洁，避免重复

## 项目结构

```
.
├── AGENTS.md         # AI 编码助手总约束入口
├── README.md         # 项目说明（给人看）
├── backend/          # 后端（Spring Boot 3.4 + JDK 21 + Maven）
│   ├── pom.xml
│   └── src/
│       ├── main/java/com/fanxl/demo/
│       └── test/java/com/fanxl/demo/
├── frontend/         # 前端（Vue 3 + Vite + Element Plus + Tailwind）
├── openspec/         # OpenSpec 规范驱动开发
│   ├── project.md    # 项目概览（本文件）
│   ├── specs/        # 当前规范（事实来源）
│   └── changes/      # 变更提案
└── .qoder/           # Qoder Agent 配置
    ├── skills/       # Superpowers 风格技能
    └── rules/        # Agent 行为规则
```
