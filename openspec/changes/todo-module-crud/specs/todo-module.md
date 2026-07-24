# Spec: ToDo 待办清单模块

## 1. 模块边界

### 包含的功能

| 功能 | 说明 |
|------|------|
| 创建待办 | 新增一条待办事项（标题、描述、标签、优先级、截止时间） |
| 查询待办列表 | 获取全部待办，支持按完成状态过滤 |
| 查询单条待办 | 按 ID 获取详情 |
| 更新待办 | 修改标题、描述、标签、优先级、截止时间、完成状态 |
| 删除待办 | 按 ID 删除 |

### 不包含的功能

- ❌ 用户注册/登录（单用户私有）
- ❌ 协作/分享/多租户
- ❌ 全文搜索
- ❌ 分页（单用户数据量小，YAGNI）
- ❌ 文件附件
- ❌ 排序/筛选（除完成状态过滤外）

---

## 2. 核心场景

### 2.1 正常路径

#### SC-01: 创建待办

```
WHEN 用户提交创建请求，包含合法的 title（10~100 字符）、描述、标签、优先级、截止时间
THEN 后端返回 200，响应体中包含新创建的 TodoItem
  AND TodoItem.id 不为空
  AND TodoItem.completed 为 false
  AND TodoItem.priority 为 MEDIUM（默认值）
  AND TodoItem.createdAt / updatedAt 为当前时间
```

#### SC-02: 查询待办列表（全部）

```
WHEN 用户发起 GET /api/todos 请求
THEN 后端返回 200，响应体中包含 TodoItem[]
  AND 列表包含所有已创建待办（无论完成状态）
```

#### SC-03: 查询待办列表（按完成状态过滤）

```
WHEN 用户发起 GET /api/todos?completed=true 请求
THEN 后端返回 200，响应体中仅包含 completed=true 的待办
WHEN 用户发起 GET /api/todos?completed=false 请求
THEN 后端返回 200，响应体中仅包含 completed=false 的待办
```

#### SC-04: 查询单条待办

```
WHEN 用户发起 GET /api/todos/{id} 请求，且 id 存在
THEN 后端返回 200，响应体中包含对应的 TodoItem
  AND 字段值与创建时一致
```

#### SC-05: 更新待办（修改标题）

```
WHEN 用户发起 PUT /api/todos/{id} 请求，携带新的 title
THEN 后端返回 200，响应体中 TodoItem.title 已更新
  AND TodoItem.updatedAt 已更新
  AND 其他字段保持不变
```

#### SC-06: 更新待办（标记完成）

```
WHEN 用户发起 PUT /api/todos/{id} 请求，携带 { completed: true }
THEN 后端返回 200，响应体中 TodoItem.completed 为 true
  AND TodoItem.updatedAt 已更新
```

#### SC-07: 删除待办

```
WHEN 用户发起 DELETE /api/todos/{id} 请求，且 id 存在
THEN 后端返回 200，响应体 data 为 null
  AND 后续 GET /api/todos 不再包含该待办
```

#### SC-08: 更新待办（更新优先级）

```
WHEN 用户发起 PUT /api/todos/{id} 请求，携带 { priority: "HIGH" }
THEN 后端返回 200，响应体中 TodoItem.priority 已更新
  AND TodoItem.updatedAt 已更新
  AND 其他字段保持不变
```

#### SC-09: 更新待办（更新标签）

```
WHEN 用户发起 PUT /api/todos/{id} 请求，携带 { tags: "生活,健康" }
THEN 后端返回 200，响应体中 TodoItem.tags 已替换为 "生活,健康"
  AND TodoItem.updatedAt 已更新
```

#### SC-10: 更新待办（更新截止时间）

```
WHEN 用户发起 PUT /api/todos/{id} 请求，携带 { dueDate: "2026-07-20T18:00:00" }
THEN 后端返回 200，响应体中 TodoItem.dueDate 已更新
  AND TodoItem.updatedAt 已更新

WHEN 用户发起 PUT /api/todos/{id} 请求，携带 { dueDate: null }
THEN 后端返回 200，响应体中 TodoItem.dueDate 为 null
  AND TodoItem.updatedAt 已更新
```

---

### 2.2 异常路径

#### EC-01: 创建待办 — title 为空或空白

```
WHEN 用户提交创建请求，title 为空字符串（或仅含空白字符）
THEN 后端返回 400
  AND 响应体包含校验错误信息（如 "title must not be blank"）
```

#### EC-02: 创建待办 — title 长度不足或超长

```
WHEN 用户提交创建请求，title 长度在 1~9 字符之间
THEN 后端返回 400
  AND 响应体包含校验错误信息（如 "title length must be between 10 and 100"）

WHEN 用户提交创建请求，title 超过 100 字符
THEN 后端返回 400
  AND 响应体包含校验错误信息（如 "title length must be between 10 and 100"）
```

#### EC-03: 创建待办 — 不合法的优先级值

```
WHEN 用户提交创建请求，priority 不为 LOW / MEDIUM / HIGH
THEN 后端返回 400
  AND 响应体包含校验错误信息（如 "priority must be one of: LOW, MEDIUM, HIGH"）
```

#### EC-04: 查询/更新/删除不存在的待办

```
WHEN 用户对不存在的 id 执行 GET /api/todos/{id}
THEN 后端返回 404
  AND 响应体 code 为 404
  AND message 为 "todo not found, id={id}"

WHEN 用户对不存在的 id 执行 PUT /api/todos/{id}
THEN 后端返回 404
  AND 响应体 code 为 404
  AND message 为 "todo not found, id={id}"

WHEN 用户对不存在的 id 执行 DELETE /api/todos/{id}
THEN 后端返回 404
  AND 响应体 code 为 404
  AND message 为 "todo not found, id={id}"
```

> id 不存在时，GET/PUT/DELETE 统一返回 404，message 中携带请求的 id 方便排查。

#### EC-05: 更新待办 — id 合法但请求体为空

```
WHEN 用户发起 PUT /api/todos/{id} 请求，请求体为空对象 {}
THEN 后端返回 400
  AND 响应体提示至少提供一个可更新字段
```

#### EC-06: 更新待办 — title 为空或超长或不满足最低长度

```
WHEN 用户发起 PUT /api/todos/{id} 请求，title 为空字符串或小于 10 字符
THEN 后端返回 400
  AND 响应体包含校验错误信息

WHEN 用户发起 PUT /api/todos/{id} 请求，title 超过 100 字符
THEN 后端返回 400
  AND 响应体包含校验错误信息
```

#### EC-07: 删除待办 — 重复删除

```
WHEN 用户对已删除的 id 再次发起 DELETE 请求
THEN 后端返回 404
  AND 响应体 code 为 404
  AND message 为 "todo not found, id={id}"
```

#### EC-08: 创建/更新 — description 超长

```
WHEN 用户提交创建或更新请求，description 超过 500 字符
THEN 后端返回 400
  AND 响应体包含校验错误信息（如 "description must not exceed 500 characters"）
```

#### EC-09: 创建/更新 — tags 超长

```
WHEN 用户提交创建或更新请求，tags 超过 255 字符
THEN 后端返回 400
  AND 响应体包含校验错误信息（如 "tags must not exceed 255 characters"）
```

#### EC-10: 创建/更新 — dueDate 格式非法

```
WHEN 用户提交创建或更新请求，dueDate 不是合法的 ISO-8601 日期时间格式
THEN 后端返回 400
  AND 响应体包含校验错误信息（如 "dueDate must be ISO-8601 format, e.g. 2026-07-15T18:00:00"）
```

#### EC-11: 更新 — completed 非法值

```
WHEN 用户提交更新请求，completed 不是布尔值（如字符串 "yes" 或数字 1）
THEN 后端返回 400
  AND 响应体包含校验错误信息（如 "completed must be a boolean"）
```

#### EC-12: 请求体 JSON 解析失败

```
WHEN 用户提交请求，请求体 JSON 格式错误（如多余逗号、缺失引号）
THEN 后端返回 400
  AND 响应体包含错误信息（如 "Request body is malformed JSON"）
```

---

## 3. 数据结构

### 3.1 通用约定

- 所有字段命名使用 **camelCase**（前后端统一）
- 所有时间字段统一使用 **UTC 时区**，ISO-8601 格式（如 `"2026-07-12T10:00:00Z"`）
- 后端 Jackson 配置 `spring.jackson.date-format=yyyy-MM-dd'T'HH:mm:ss'Z'` 并设置 `time-zone=UTC`
- 所有响应均包装在 `ApiResponse<T>` 中
- 请求 Content-Type: `application/json`

### 3.2 TodoItem 实体

| 字段 | 类型 | 约束 | 示例 |
|------|------|------|------|
| id | Long | 自增主键，不可为空 | 1 |
| title | String | 必填，**10~100 字符** | "去超市买明天的早餐牛奶" |
| description | String | 可选，0~500 字符 | null 或 "记得买脱脂的" |
| completed | Boolean | 默认 false | false |
| tags | String | 可选，0~255 字符，逗号分隔 | "生活,购物" |
| priority | String | 枚举：LOW / MEDIUM / HIGH，默认 MEDIUM | "MEDIUM" |
| dueDate | String | 可选，ISO-8601 日期时间，可 null | "2026-07-15T18:00:00" |
| createdAt | String | ISO-8601 日期时间，不可为空 | "2026-07-12T18:00:00" |
| updatedAt | String | ISO-8601 日期时间，不可为空 | "2026-07-12T18:00:00" |

> tags 约束补充：单个标签前后自动 trim 空格，单个标签最长 20 字符，标签名不可包含逗号。更新时传入的逗号分隔值将**全量替换**所有已有标签。

### 3.3 API 端点

| 方法 | 路径 | 请求体 | 响应体 |
|------|------|--------|--------|
| GET | `/api/todos` | — | `ApiResponse<List<TodoItem>>` |
| GET | `/api/todos?completed=true` | — | `ApiResponse<List<TodoItem>>` |
| GET | `/api/todos/{id}` | — | `ApiResponse<TodoItem>` |
| POST | `/api/todos` | `CreateTodoRequest` | `ApiResponse<TodoItem>` |
| PUT | `/api/todos/{id}` | `UpdateTodoRequest` | `ApiResponse<TodoItem>` |
| DELETE | `/api/todos/{id}` | — | `ApiResponse<Void>` |

> PUT 端点以**部分更新**语义工作（类似 PATCH），未传入的字段保持原值。若需严格遵循 RESTful 规范，后续可切换为 PATCH 方法。

### 3.4 请求体结构

#### CreateTodoRequest

| 字段 | 类型 | 必填 | 约束 | 示例 |
|------|------|------|------|------|
| title | String | ✅ | **10~100 字符**，不可全空格 | "去超市买明天的早餐牛奶" |
| description | String | ❌ | 0~500 字符，可 null | "记得买脱脂的" |
| tags | String | ❌ | 0~255 字符，逗号分隔多个标签 | "生活,购物" |
| priority | String | ❌ | 不传时默认 MEDIUM，传值必须为 LOW/MEDIUM/HIGH | "HIGH" |
| dueDate | String | ❌ | ISO-8601 日期时间字符串，可 null | "2026-07-15T18:00:00" |

#### UpdateTodoRequest

| 字段 | 类型 | 必填 | 约束 | 示例 |
|------|------|------|------|------|
| title | String | ❌ | 传值时 10~100 字符 | "去超市买明天的豆奶" |
| description | String | ❌ | 传值时 0~500 字符 | null |
| tags | String | ❌ | 传值时 0~255 字符，逗号分隔，**全量替换**已有标签 | "生活,健康" |
| priority | String | ❌ | 传值时必须为 LOW/MEDIUM/HIGH | "LOW" |
| completed | Boolean | ❌ | 传值时 true/false | true |
| dueDate | String | ❌ | ISO-8601 格式，**不传时保持原值**，传 null 清除截止时间 | null |

> **至少需要一个字段参与更新**，不允许空对象。未传入的字段保持原值不变。

### 3.5 响应体结构

```json
// 成功示例
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "title": "去超市买明天的早餐牛奶",
    "description": "记得买脱脂的",
    "completed": false,
    "tags": "生活,购物",
    "priority": "MEDIUM",
    "dueDate": "2026-07-15T18:00:00",
    "createdAt": "2026-07-12T18:00:00",
    "updatedAt": "2026-07-12T18:00:00"
  }
}

// 列表示例
{
  "code": 200,
  "message": "success",
  "data": [
    { "id": 1, "title": "去超市买明天的早餐牛奶", "description": null, "completed": false, "tags": "生活", "priority": "HIGH", "dueDate": "2026-07-15T18:00:00", "createdAt": "...", "updatedAt": "..." },
    { "id": 2, "title": "完成Q3产品需求文档撰写", "description": "下班前提交", "completed": true, "tags": "工作", "priority": "MEDIUM", "dueDate": null, "createdAt": "...", "updatedAt": "..." }
  ]
}

// 404 示例
{
  "code": 404,
  "message": "todo not found, id=99",
  "data": null
}

// 400 示例
{
  "code": 400,
  "message": "title length must be between 10 and 100",
  "data": null
}
```

---

## 4. 验收标准

### 后端

- [ ] `POST /api/todos` 创建成功返回 200 + TodoItem（含 tags/priority/dueDate）+ 响应体符合 §3.2 结构
- [ ] `POST /api/todos` title 为空或空白字符串返回 400 + 校验错误信息
- [ ] `POST /api/todos` title 长度不足 10 字符返回 400
- [ ] `POST /api/todos` title 超过 100 字符返回 400
- [ ] `POST /api/todos` description 超过 500 字符返回 400
- [ ] `POST /api/todos` tags 超过 255 字符返回 400
- [ ] `POST /api/todos` priority 非法值返回 400
- [ ] `POST /api/todos` dueDate 格式非法返回 400
- [ ] `GET /api/todos` 返回所有待办列表
- [ ] `GET /api/todos?completed=true` 仅返回已完成项
- [ ] `GET /api/todos?completed=false` 仅返回未完成项
- [ ] `GET /api/todos/{id}` 存在时返回对应项（含 tags/priority/dueDate）
- [ ] `GET /api/todos/{id}` 不存在时返回 404 + message "todo not found, id={id}"
- [ ] `PUT /api/todos/{id}` 更新标题成功，其他字段保持不变
- [ ] `PUT /api/todos/{id}` 标记完成成功
- [ ] `PUT /api/todos/{id}` 更新 priority/tags/dueDate 成功
- [ ] `PUT /api/todos/{id}` 空请求体返回 400
- [ ] `PUT /api/todos/{id}` 不存在返回 404 + message 含 id
- [ ] `PUT /api/todos/{id}` completed 传非 Boolean 值返回 400
- [ ] `DELETE /api/todos/{id}` 删除成功
- [ ] `DELETE /api/todos/{id}` 不存在返回 404
- [ ] `DELETE /api/todos/{id}` 重复删除返回 404
- [ ] 请求体 JSON 格式错误时返回 400
- [ ] `mvn test` 后端全部单元测试通过（≥20 个用例）

### 前端

- [ ] 待办列表页：有数据时正确渲染列表（含标签 el-tag/优先级颜色/截止时间），无数据时显示空状态提示
- [ ] 创建待办：表单输入 title + description + tags + priority 下拉 + dueDate 日期选择，提交后新项出现在列表顶部
- [ ] 标记完成：点击复选框切换 completed 状态，UI 实时同步
- [ ] 删除待办：点击删除按钮，确认弹窗后移除
- [ ] 按完成状态筛选：Tab 切换 "全部/已完成/未完成"，列表即时刷新
- [ ] 创建失败提示：title 为空或不足 10 字符时表单内联校验提示
- [ ] 网络错误处理：API 调用失败时显示 Element Plus 消息提示
- [ ] `pnpm build` 通过（vue-tsc 无错）

