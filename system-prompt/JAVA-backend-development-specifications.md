# 后端全局 AGENTS 配置

## 概述
主要针对后端开发。代理负责自动化代码生成、优化和维护，确保代码符合最佳实践。所有代理行为均需遵循以下规范，以保证代码的可读性、可维护性和性能优化。配置采用 YAML 或 JSON 格式（视具体实现而定），但本 MD 文件提供人类可读的描述和示例。

必须优先考虑JAVA后端技术栈：Spring Boot、MyBatis Plus、Lombok 等。代码规范强调简洁、高效，避免冗余。

## 通用规范
### 1. 命名与代码风格 (Naming & Style)
- **类与接口**：使用 `PascalCase`（大驼峰）。接口通常不需要 `I` 前缀（如 `UserService` 而不是 `IUserService`），实现类通常带有 `Impl` 后缀（如 `UserServiceImpl`）。
- **方法与变量**：使用 `camelCase`（小驼峰）。
- **常量**：使用 `UPPER_SNAKE_CASE`（全大写加下划线），且必须声明为 `public/private static final`。
- **包名**：全小写，单词间不使用下划线，以点分隔（如 `com.example.project.util`）。
- **禁止使用魔法值**：代码中不能出现未定义的魔法数字或字符串，必须提取为常量或枚举（Enum）。

### 2. Java 语言特性最佳实践 (Java Best Practices)
- **集合处理**：优先使用 Java 17+ Stream API 处理集合转换和过滤。返回空集合时，使用 `Collections.emptyList()`，绝对不要返回 `null`。
- **空值处理**：使用 `Optional` 作为方法的返回值（当返回值可能为空时），**禁止**将 `Optional` 作为方法参数或类的字段。
- **不可变性**：方法参数尽可能视为不可变；不需要修改的局部变量优先考虑使用 `final`。

### 3. Spring 生态规范 (Spring Ecosystem Standards)
- **依赖注入 (DI)**：**严禁使用 `@Autowired` 字段注入**。必须使用**构造器注入**。如果使用了 Lombok，请使用 `private final` 修饰依赖字段，并在类上添加 `@RequiredArgsConstructor` 注解。
- **组件扫描与 Bean 作用域**：正确使用 `@Component`、`@Service`、`@Configuration` 等注解。必须保证绝大多数 Bean 是无状态的（Stateless），以确保在默认的 Singleton 作用域下线程安全。
- **配置管理**：优先使用 `@ConfigurationProperties` 绑定配置项，避免在代码中散落大量 `@Value` 注解。

### 4. 模块与架构适配 (Web App vs Maven Library)
- **如果是 Web 应用开发**：
  - **Controller 规范**：使用 `@RestController`。遵循 RESTful 风格，路径使用名词的复数形式（如 `/api/v1/users`）。明确指定 HTTP 方法（`@GetMapping`, `@PostMapping` 等）。
  - **分层架构**：严格遵守 Controller -> Service -> Repository/DAO 三层架构。Controller 层只负责参数校验和请求路由，业务逻辑必须在 Service 层。
  - **内外隔离**：禁止将数据库实体（Entity）直接返回给前端，必须转换为 DTO（Data Transfer Object）或 VO（View Object）。
- **如果是 Maven 依赖/共享库开发**：
  - **最小依赖原则**：不引入不必要的第三方依赖（如尽量不依赖重量级 Web 框架，除非是 Web 专用组件）。
  - **自动配置**：如果是 Spring Boot Starter，规范提供 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`（Spring Boot 2.7+）和 `@AutoConfiguration` 类。且采用三层架构。
  - **扩展性**：对外暴露的 API 应面向接口编程，内部实现类降低可见性（包级私有）。

### 5. 异常处理 (Exception Handling)
- **避免过度捕获**：不要笼统地捕获 `Exception` 或 `Throwable`，除非是在顶层的全局异常处理器中。捕获异常后如果无法处理，必须抛出。
- **自定义异常**：业务逻辑错误应抛出自定义的 `RuntimeException`，而不是使用通用异常。
- **Web 全局异常**：如果是 Web 项目，必须假设存在且配合 `@RestControllerAdvice` 和 `@ExceptionHandler` 进行全局异常拦截，并返回统一格式的 JSON 结构。

### 6. 日志规范 (Logging)
- **禁止使用 `System.out` 或 `System.err`**。
- **日志框架**：必须使用 SLF4J 接口（如 Lombok 的 `@Slf4j`）。
- **占位符**：输出日志时使用参数化日志占位符（如 `log.info("Processing user id: {}", userId);`），禁止直接使用字符串拼接（`+`）。
- **异常日志**：打印异常时必须传入异常对象本身（如 `log.error("Failed to process", e);`），不要只打印 `e.getMessage()` 从而丢失堆栈信息。

### 7. 注释与文档 (Documentation)
- **类级别**：每个公共类、接口都必须有 Javadoc，简要说明其职责和设计意图。
- **方法级别**：所有 public 方法（尤其是接口方法和库的对外 API）必须有 Javadoc，包含 `@param` 和 `@return` 说明。
- **行内注释**：代码即文档。只在极其复杂的算法或不符合直觉的业务逻辑处使用行内注释 `//`，不要用中文直译代码（例如 `// 设置名字 user.setName(name);` 是废话，坚决不要）。
- **文档规范**：README.md 必须包含项目启动指南、依赖列表和规范链接。

### 8. 性能与并发安全 (Performance & Concurrency)
- **线程池**：禁止在代码中使用 `new Thread()`。必须使用 Spring 提供的 `@Async`（配合自定义线程池）或通过 `ThreadPoolExecutor` 创建全局共享线程池。
- **资源释放**：处理文件、流、网络连接等资源时，必须使用 `try-with-resources` 语法确保资源安全关闭。
- **ThreadLocal**：使用 `ThreadLocal` 时，必须保证在 `finally` 块中调用 `remove()` 方法，防止内存泄漏。

### AI 执行要求
在接收到我的具体编码需求后，你必须：
1. 先思考适用的是 Web 场景还是基础库场景，或者是其它场景。
2. 给出符合上述规范的 Java 代码。
3. 代码应当直接可用，包含必要的 import 语句。
4. 如果使用了未提及的技术选型（如具体的 JSON 库、特定的工具类），优先使用 Spring 生态自带的（如 Jackson, Spring 工具类）。

## MyBatis Plus 特定规范
MyBatis Plus 是核心 ORM 框架，任何涉及 MyBatis-Plus 查询构造、Wrapper、Specification 的问题，**必须先读取 ./docs/specification/mybatis-plus-specification.md 的完整内容**，然后再思考和回复和进行下一步。

## Spring Boot 特定规范
- **Controller 层**：
  - 使用 @RestController，返回统一响应对象（e.g., ResponseEntity 或自定义 Result 类）。
  - 参数校验使用 @Valid 或 @Validated。
  - 示例：
    ```java
    @PostMapping("/users")
    public Result<User> createUser(@Valid @RequestBody UserDTO dto) {
        // 业务逻辑
    }
    ```

- **Service 层**：
  - 业务逻辑核心层，使用 @Service 注解。
  - 事务管理使用 @Transactional(readOnly = true) 对于读操作。
  - 避免 Service 层直接抛出异常，一律包装为业务异常。

- **Repository 层**：
  - 继承 BaseMapper<User> 接口。
  - 自定义方法使用注解 @Select, @Update 等。
  - 禁止在 Repository 层使用 @Transactional。

## 附加补充
- **多线程处理**：使用 ThreadPoolTaskExecutor，避免直接 new Thread。
- **缓存**：集成 Redis，使用 @Cacheable 对于高频查询。

如果需要更新配置，请通过 PR 提交，并确保所有代理在生成代码时自动应用这些规范。
