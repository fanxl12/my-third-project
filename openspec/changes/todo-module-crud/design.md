# Design: todo-module-crud

## 概述

新增 ToDo 待办清单 CRUD 功能，后端 Spring Boot 3.4 + MyBatis-Plus + PostgreSQL，前端 Vue 3 + Element Plus，前后端分离联调。

## 数据库

| 项目 | 配置 |
|------|------|
| 类型 | PostgreSQL |
| 主机 | 192.168.9.2:5432 |
| 数据库 | fan_demo |
| 用户名 | postgres |
| 密码 | postgres |
| 连接池 | HikariCP (Spring Boot 默认) |
| DDL | `schema.sql` 初始化表结构（MyBatis-Plus 不支持自动 DDL） |

### 表结构

**todo_item** 表

| 字段 | 类型 | 约束 |
|------|------|------|
| id | BIGSERIAL | PRIMARY KEY |
| title | VARCHAR(100) | NOT NULL |
| description | VARCHAR(500) | NULLABLE |
| completed | BOOLEAN | DEFAULT false |
| tags | VARCHAR(255) | NULLABLE，逗号分隔 |
| priority | VARCHAR(10) | NOT NULL DEFAULT 'MEDIUM'，枚举 LOW/MEDIUM/HIGH |
| due_date | TIMESTAMP | NULLABLE |
| created_at | TIMESTAMP | NOT NULL DEFAULT NOW() |
| updated_at | TIMESTAMP | NOT NULL DEFAULT NOW() |

> MyBatis-Plus 通过 `@TableField` 显式指定字段映射，数据库字段使用 snake_case，Java 字段使用 camelCase。

### 建表 SQL（`schema.sql`）

```sql
CREATE TABLE IF NOT EXISTS todo_item (
    id BIGSERIAL PRIMARY KEY,
    title VARCHAR(100) NOT NULL,
    description VARCHAR(500),
    completed BOOLEAN DEFAULT false,
    tags VARCHAR(255),
    priority VARCHAR(10) NOT NULL DEFAULT 'MEDIUM',
    due_date TIMESTAMP,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX IF NOT EXISTS idx_todo_completed ON todo_item(completed);
CREATE INDEX IF NOT EXISTS idx_todo_due_date ON todo_item(due_date);
CREATE INDEX IF NOT EXISTS idx_todo_priority ON todo_item(priority);
```

## 后端架构

### 包结构

```
com.fanxl.demo.todo/
├── entity/
│   └── TodoItem.java              # MyBatis-Plus 实体
├── mapper/
│   └── TodoMapper.java            # MyBatis-Plus Mapper（继承 BaseMapper）
├── service/
│   ├── TodoService.java           # 接口
│   └── TodoServiceImpl.java       # 实现
├── controller/
│   └── TodoController.java        # REST 接口
├── dto/
│   ├── CreateTodoRequest.java     # 创建请求 DTO
│   └── UpdateTodoRequest.java     # 更新请求 DTO
└── exception/
    └── GlobalExceptionHandler.java  # 统一异常处理
```

### API 端点

| 方法 | 路径 | 请求体 | 响应体 |
|------|------|--------|--------|
| GET | `/api/todos` | — | `ApiResponse<List<TodoItem>>` |
| GET | `/api/todos?completed=bool` | — | `ApiResponse<List<TodoItem>>` |
| GET | `/api/todos/{id}` | — | `ApiResponse<TodoItem>` |
| POST | `/api/todos` | `CreateTodoRequest` | `ApiResponse<TodoItem>` |
| PUT | `/api/todos/{id}` | `UpdateTodoRequest` | `ApiResponse<TodoItem>` |
| DELETE | `/api/todos/{id}` | — | `ApiResponse<Void>` |

### DTO 定义

**CreateTodoRequest**
| 字段 | 类型 | 必填 | 约束 |
|------|------|------|------|
| title | String | ✅ | 10~100 字符，@Size(min=10, max=100) + @NotBlank |
| description | String | ❌ | 0~500 字符，@Size(max=500) |
| tags | String | ❌ | 0~255 字符，@Size(max=255) |
| priority | String | ❌ | 不传默认 MEDIUM，传值 @Pattern(regexp="LOW|MEDIUM|HIGH") |
| dueDate | String | ❌ | ISO-8601 格式，可 null（后端 @JsonFormat + 若传值需 @Pattern 校验格式）

**UpdateTodoRequest**
| 字段 | 类型 | 必填 | 约束 |
|------|------|------|------|
| title | String | ❌ | 传值时 10~100 字符 |
| description | String | ❌ | 传值时 0~500 字符 |
| tags | String | ❌ | 传值时 0~255 字符 |
| priority | String | ❌ | 传值时必须为 LOW/MEDIUM/HIGH |
| completed | Boolean | ❌ | 传值时 true/false |
| dueDate | String | ❌ | ISO-8601 格式，传 null 清除 |

> 至少提供一个字段，否则返回 400。

### Maven 新增依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
    <version>3.5.9</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

### application.yml 新增配置

```yaml
spring:
  datasource:
    url: jdbc:postgresql://192.168.9.2:5432/fan_demo
    username: postgres
    password: postgres
    driver-class-name: org.postgresql.Driver
  sql:
    init:
      mode: always
      schema-locations: classpath:schema.sql
  jackson:
    date-format: yyyy-MM-dd'T'HH:mm:ss'Z'
    time-zone: UTC

mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
      id-type: auto

> log-impl 输出 SQL 日志用于开发调试，后续可关闭。

### 异常处理

| 异常 | HTTP 状态码 | Message 示例 |
|------|-------------|-------------|
| `MethodArgumentNotValidException` | 400 | "title must not be blank" |
| `ConstraintViolationException` | 400 | "description size must not exceed 500" |
| `HttpMessageNotReadableException` | 400 | "request body is malformed JSON" |
| `ResourceNotFoundException` | 404 | "todo not found, id=99" |
| 未处理异常 | 500 | "internal server error" |

### 实体映射策略

MyBatis-Plus 实体通过注解显式定义表映射规则：

| Java 字段 | 表字段 | 映射方式 |
|-----------|--------|----------|
| `id` | `id` | `@TableId(type = IdType.AUTO)` |
| `title` | `title` | 自动映射 |
| `description` | `description` | 自动映射 |
| `completed` | `completed` | 自动映射 |
| `tags` | `tags` | 自动映射 |
| `priority` | `priority` | 自动映射 |
| `dueDate` | `due_date` | `@TableField` 手动映射（snake_case vs camelCase） |
| `createdAt` | `created_at` | `@TableField(fill = FieldFill.INSERT)` |
| `updatedAt` | `updated_at` | `@TableField(fill = FieldFill.INSERT_UPDATE)` |

> `createdAt` / `updatedAt` 通过 MyBatis-Plus 的 MetaObjectHandler 自动填充。`dueDate` 由于 `due_date` 与 `dueDate` 无法通过默认下划线转换正确映射（`duedate` 而非 `due_date`），需显式指定 `@TableField("due_date")`。

## 前端架构

### 路由

| 路径 | 组件 | 导航 |
|------|------|------|
| `/` | `HomeView` | 首页 |
| `/todos` | `TodoView` | 待办页面 |

### 新增文件

```
src/
├── types/
│   └── todo.ts                     # TypeScript 接口定义
├── api/
│   └── todo.ts                     # ToDo API 调用封装
├── views/
│   └── TodoView.vue                # 待办主页面
├── components/todo/
│   ├── TodoForm.vue                # 创建表单组件
│   ├── TodoList.vue                # 待办列表组件
│   └── TodoItem.vue                # 单条待办组件
```

### 组件职责

| 组件 | 职责 |
|------|------|
| `TodoView` | 页面容器，维护数据状态，编排子组件 |
| `TodoForm` | title 输入 + 添加按钮，表单校验 |
| `TodoList` | 渲染列表、Tab 筛选（全部/已完成/未完成） |
| `TodoItem` | 展示单条数据、复选框切换、删除按钮 |

### 数据流

```
TodoView (状态管理中心)
  ├─ todos: TodoItem[]
  ├─ loading: boolean
  ├─ filter: 'all' | 'active' | 'completed'
  │
  ├─ onMounted → fetchTodos()
  ├─ handleCreate(title) → POST /api/todos → fetchTodos()
  ├─ handleToggle(item)  → PUT /api/todos/{id} → fetchTodos()
  └─ handleDelete(id)    → DELETE /api/todos/{id} → fetchTodos()
```

### TypeScript 类型

```ts
// types/todo.ts
export interface TodoItem {
  id: number
  title: string
  description: string | null
  completed: boolean
  tags: string | null
  priority: 'LOW' | 'MEDIUM' | 'HIGH'
  dueDate: string | null
  createdAt: string
  updatedAt: string
}

export interface CreateTodoRequest {
  title: string
  description?: string
  tags?: string
  priority?: 'LOW' | 'MEDIUM' | 'HIGH'
  dueDate?: string | null
}

export interface UpdateTodoRequest {
  title?: string
  description?: string
  tags?: string
  priority?: 'LOW' | 'MEDIUM' | 'HIGH'
  completed?: boolean
  dueDate?: string | null
}

export interface ApiResponse<T> {
  code: number
  message: string
  data: T
}
```

## 测试策略

| 层级 | 工具 | 覆盖 |
|------|------|------|
| 后端单元测试 | JUnit 5 + MockMvc | Controller 6 端点 + 异常路径 |
| 后端集成测试 | JUnit 5 + @SpringBootTest | Mapper CRUD + Service 逻辑 |
| 前端构建 | vue-tsc + vite build | 类型检查 + 构建通过 |

> 测试使用 H2 内存数据库替代 PostgreSQL，通过 `schema.sql` + `application-test.yml` 配置切换数据源，避免测试对真实数据库的依赖。MyBatis-Plus 的 `BaseMapper` 继承后无需额外 SQL 映射文件。

## 不包含（YAGNI）

- ❌ 分页（单用户数据量小）
- ❌ 用户认证/权限
- ❌ 全文搜索
- ❌ 拖拽排序
- ❌ 批量操作
