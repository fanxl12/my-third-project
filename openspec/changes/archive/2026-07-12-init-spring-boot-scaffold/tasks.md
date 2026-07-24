# Tasks: init-spring-boot-scaffold

## Phase 1: 项目骨架

- [x] 1.1 创建 Maven Wrapper 文件
  - Files: `backend/.mvn/wrapper/maven-wrapper.properties`, `backend/mvnw`, `backend/mvnw.cmd`
  - Verify: `cd backend && ./mvnw --version` 输出 Maven 版本信息

- [x] 1.2 创建 `pom.xml`（Spring Boot 3.4 + web + test + lombok）
  - File: `backend/pom.xml`
  - Verify: `./mvnw validate` 通过

- [x] 1.3 [RED] 写 Spring Boot 上下文加载测试（期望失败，因为无 Application 类）
  - File: `backend/src/test/java/com/fanxl/demo/DemoApplicationTests.java`
  - Verify: `./mvnw test` 编译失败

## Phase 2: 最小实现

- [x] 2.1 [GREEN] 创建 `DemoApplication.java` 入口类
  - File: `backend/src/main/java/com/fanxl/demo/DemoApplication.java`
  - Depends on: 1.3
  - Verify: `./mvnw test` 上下文加载测试通过

- [x] 2.2 [RED] 写 HealthController 测试（期望 404，因为无 Controller）
  - File: `backend/src/test/java/com/fanxl/demo/controller/HealthControllerTest.java`
  - Depends on: 2.1
  - Verify: `./mvnw test -Dtest=HealthControllerTest` 测试失败（404）

- [x] 2.3 [GREEN] 创建 `HealthController.java` 和 `ApiResponse.java`
  - Files: `backend/src/main/java/com/fanxl/demo/controller/HealthController.java`, `backend/src/main/java/com/fanxl/demo/common/ApiResponse.java`
  - Depends on: 2.2
  - Verify: `./mvnw test` 全部通过，GET /api/health 返回 200

- [x] 2.4 [GREEN] 创建 `application.yml` 配置
  - File: `backend/src/main/resources/application.yml`
  - Depends on: 2.3
  - Verify: `./mvnw spring-boot:run` 启动成功，访问 `/api/health` 返回正确 JSON

## Phase 3: 收尾

- [x] 3.1 [REFACTOR] 确认所有测试通过，代码整洁
  - Verify: `./mvnw test` 全部通过，代码格式一致

- [x] 3.2 更新 `openspec/project.md` 技术栈
  - File: `openspec/project.md`
  - Verify: project.md 中后端技术栈已填写

- [x] 3.3 启动验证：`./mvnw spring-boot:run` → 访问 `/api/health` → 返回 UP
  - Verify: 手动或用 curl 验证
