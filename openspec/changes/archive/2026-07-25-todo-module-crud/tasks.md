# Tasks: todo-module-crud

## Phase 1: 后端基础设施

- [ ] **Task 1.1** pom.xml 追加依赖
  - 新增 `mybatis-plus-spring-boot3-starter`、`spring-boot-starter-validation`、`postgresql`
  - **验证**: `mvn dependency:tree` 列出新增依赖

- [ ] **Task 1.2** application.yml 新增 PostgreSQL + MyBatis-Plus 配置
  - `spring.datasource.url=jdbc:postgresql://192.168.9.2:5432/fan_demo`
  - `spring.sql.init.schema-locations=classpath:schema.sql`
  - `mybatis-plus.configuration.map-underscore-to-camel-case=true`
  - **验证**: `spring-boot:run` 启动成功，数据库连接正常

- [ ] **Task 1.3** schema.sql 建表脚本（含索引）+ DemoApplication 添加 @MapperScan
  - 创建 `schema.sql` 建表（含 tags/priority/due_date 字段 + 索引 completed/due_date/priority）
  - `DemoApplication.java` 添加 `@MapperScan("com.fanxl.demo.todo.mapper")`

## Phase 2: 后端实体 + 数据访问 (RED → GREEN)

- [ ] **Task 2.1 RED** TodoService 测试
  - 创建 `TodoServiceTest.java`（使用 @SpringBootTest + H2 内存库）
  - 测试: `createTodo`、`getAllTodos`、`getTodoById`、`updateTodo`、`deleteTodo`
  - 覆盖: title 不足10字符/空白、priority 非法值、description 超500字符、tags 超255字符、dueDate 格式非法、id 不存在 404
  - **验证**: 测试编译通过，运行失败（Service 未实现）

- [ ] **Task 2.2 GREEN** TodoItem 实体 + TodoMapper + TodoService 实现
  - `TodoItem.java`: @TableName, @TableId, @TableField 注解
  - `TodoMapper.java`: extends BaseMapper<TodoItem>
  - `TodoService.java` + `TodoServiceImpl.java`: CRUD + 异常
  - **验证**: 所有测试通过

## Phase 3: 后端 REST 接口 (RED → GREEN)

- [ ] **Task 3.1 RED** TodoController 测试
  - 创建 `TodoControllerTest.java`
  - 测试: 6 个端点 + 异常路径（title空白/不足10字符400、priority非法值400、description超长400、tags超长400、dueDate格式非法400、completed非Boolean400、不存在404、空请求体400、JSON解析失败400）
  - **验证**: 测试编译通过，运行返回 404 或 400（Controller 未实现）

- [ ] **Task 3.2 GREEN** TodoController 实现
  - `CreateTodoRequest` + `UpdateTodoRequest` DTO（含 tags/priority/dueDate）+ @Valid 校验
  - title 校验改为 @Size(min = 10, max = 100)
  - priority 校验 @Pattern(regexp = "LOW|MEDIUM|HIGH")
  - `TodoController.java`: 6 个端点
  - `GlobalExceptionHandler.java`: @ControllerAdvice 统一处理异常
  - **验证**: 所有 Controller 测试通过

## Phase 4: 前端类型 + API (RED → GREEN)

- [ ] **Task 4.1** 前端类型定义 + API 层
  - `types/todo.ts`: TodoItem / CreateTodoRequest / UpdateTodoRequest / ApiResponse
  - `api/todo.ts`: getTodos / createTodo / updateTodo / deleteTodo
  - **验证**: `pnpm build` 通过

- [ ] **Task 4.2 RED** TodoView 页面组件测试
  - 页面可挂载，路由可访问
  - **验证**: 编译通过

- [ ] **Task 4.3 GREEN** TodoView + TodoForm + TodoList + TodoItem 组件实现
  - `TodoView.vue`: 状态管理 + 数据流编排（含 tags/priority/dueDate）
  - `TodoForm.vue`: title（≥10字符前端校验）+ tags 输入 + priority 下拉选择 + dueDate 日期选择器 + 添加按钮
  - `TodoList.vue`: Tab 筛选 + 列表渲染
  - `TodoItem.vue`: 展示（含 el-tag 标签、优先级颜色标识、截止时间）+ 复选框切换 + 删除
  - `router/index.ts`: 追加 `/todos` 路由
  - **验证**: `pnpm dev` 页面可操作

## Phase 5: 最终验证

- [ ] **Task 5.1** 全链路联调
  - 同时启动 `backend:8080` + `frontend:5173`
  - 访问 `/todos` 创建/切换/删除待办，操作成功
  - **验证**: 前后端数据一致，控制台无错误

- [ ] **Task 5.2** 全部测试
  - `mvn test` 后端测试全部通过
  - `pnpm build` 前端类型检查 + 构建通过

- [ ] **Task 5.3** 更新 openspec/project.md
  - 数据库选型更新为 PostgreSQL
