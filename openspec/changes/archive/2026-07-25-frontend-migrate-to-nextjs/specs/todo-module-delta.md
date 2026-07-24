# Spec Delta: todo-module.md 前端部分适配

> 本 delta 描述 `openspec/specs/todo-module.md` 为适配 Next.js 前端所需的前端相关变更。
> 仅修改前端验收标准部分，不修改后端 API 定义、数据模型、业务场景。

---

## 变更项

### 1. §4 验收标准 → 前端（第 384–394 行）

旧文本中引用 Vue 专属组件名和技术栈，需替换为 Next.js 等价物：

| 旧（Vue 3） | 新（Next.js） |
|---|---|
| `el-tag` | Tailwind `span` 标签（彩色 Tag 徽章） |
| `el-date-picker`（隐含） | 原生 `<input type="date">` 或 Headless UI DatePicker |
| `TodoView.vue` | `app/todos/page.tsx` |
| `TodoForm.vue` | `components/todo/TodoForm.tsx` |
| `TodoList.vue` | `components/todo/TodoList.tsx` |
| `TodoItem.vue` | `components/todo/TodoItem.tsx` |
| `vue-tsc` | `tsc --noEmit`（或 `next build` 内置类型检查） |
| Element Plus 消息提示 | 自定义 Toast 组件或 Tailwind alert |

### 2. 前端验收标准文本替换

旧：
```markdown
### 前端

- [ ] 待办列表页：有数据时正确渲染列表（含标签 el-tag/优先级颜色/截止时间），无数据时显示空状态提示
- [ ] 创建待办：表单输入 title + description + tags + priority 下拉 + dueDate 日期选择，提交后新项出现在列表顶部
- [ ] 标记完成：点击复选框切换 completed 状态，UI 实时同步
- [ ] 删除待办：点击删除按钮，确认弹窗后移除
- [ ] 按完成状态筛选：Tab 切换 "全部/已完成/未完成"，列表即时刷新
- [ ] 创建失败提示：title 为空或不足 10 字符时表单内联校验提示
- [ ] 网络错误处理：API 调用失败时显示 Element Plus 消息提示
- [ ] `pnpm build` 通过（vue-tsc 无错）
```

新：
```markdown
### 前端

- [ ] 待办列表页：有数据时正确渲染列表（含标签徽章/优先级颜色/截止时间），无数据时显示空状态提示
- [ ] 创建待办：表单输入 title + description + tags + priority 下拉 + dueDate 日期选择，提交后新项出现在列表顶部
- [ ] 标记完成：点击复选框切换 completed 状态，UI 实时同步
- [ ] 删除待办：点击删除按钮，确认弹窗后移除
- [ ] 按完成状态筛选：Tab 切换 "全部/已完成/未完成"，列表即时刷新
- [ ] 创建失败提示：title 为空或不足 10 字符时表单内联校验提示
- [ ] 网络错误处理：API 调用失败时显示错误提示（Toast 或内联消息）
- [ ] `pnpm build` 通过（Next.js 构建 + TypeScript 类型检查无错）
```

### 3. 不变部分

以下部分不受影响：
- §1 模块边界
- §2 核心场景（SC-01 ~ SC-10, EC-01 ~ EC-12）
- §3 数据结构（TodoItem、API 端点、请求/响应体）
- §4 验收标准 → 后端（所有后端验收标准不变）
