# Design: init-spring-boot-scaffold

## 概述

搭建 Spring Boot 3.4 后端骨架，Maven 构建（含 wrapper），JDK 21，单模块结构，最小可用。

## 目录结构

```
backend/
├── .mvn/wrapper/
│   ├── maven-wrapper.properties    # Maven Wrapper 配置
│   └── maven-wrapper.jar
├── mvnw                            # Maven Wrapper (Unix)
├── mvnw.cmd                        # Maven Wrapper (Windows)
├── pom.xml                         # Maven POM
└── src/
    ├── main/
    │   ├── java/com/fanxl/demo/
    │   │   ├── DemoApplication.java          # Spring Boot 入口
    │   │   ├── controller/
    │   │   │   └── HealthController.java     # /api/health 端点
    │   │   └── common/
    │   │       └── ApiResponse.java          # 统一响应体
    │   └── resources/
    │       └── application.yml               # 配置
    └── test/
        └── java/com/fanxl/demo/
            ├── DemoApplicationTests.java     # 上下文加载测试
            └── controller/
                └── HealthControllerTest.java  # API 测试
```

## 技术方案

### Maven POM 核心依赖

| 依赖 | 版本 | 用途 |
|------|------|------|
| `spring-boot-starter-parent` | 3.4.x | 父 POM，统一版本管理 |
| `spring-boot-starter-web` | — | REST API（嵌入式 Tomcat） |
| `spring-boot-starter-test` | — | 测试（JUnit 5 + MockMvc） |
| `lombok` | — | 减少样板代码 |

### 关键决策

1. **Maven Wrapper** — 使用 `mvnw` 而非系统 `mvn`，团队成员无需预装 Maven，保证构建一致性
2. **`application.yml`** — 使用 YAML 格式（非 properties），层级清晰，后续多环境配置更方便
3. **`ApiResponse<T>`** — 统一响应体设计，方便前端统一处理，也方便后续加 Swagger 文档
4. **包名 `com.fanxl.demo`** — 占位包名，后续业务明确后可重构

### API 设计

```
GET /api/health
Response 200:
{
  "code": 200,
  "message": "success",
  "data": {
    "status": "UP",
    "timestamp": "2026-07-12T..."
  }
}
```

### 配置（application.yml）

```yaml
server:
  port: 8080

spring:
  application:
    name: demo
```

## 不做什么

- ❌ 不引入数据库（用户选择了 D：先不接数据库）
- ❌ 不引入 Spring Security（无认证需求）
- ❌ 不引入 Swagger/OpenAPI（业务确定后再加）
- ❌ 不引入 MyBatis/JPA（无数据库）
