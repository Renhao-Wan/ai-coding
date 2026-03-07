---
name: spring-boot-unit-test-skill
description: >
  专为 Spring Boot 项目（单体或微服务）自动生成并执行 JUnit 5 + Mockito 单元测试。（Spring Boot 业务接口单元测试生成器）
  针对业务接口（@Service 的 public 业务方法）生成正常逻辑 + 失败逻辑测试。  
  支持增量生成（已有测试的接口跳过）、按 Service 层逻辑划分测试类、单次不超过 15 个业务接口、分批计划、生成后询问是否执行测试、仅测试模式、标准化测试报告。
license: MIT
metadata:
  author: renhao
  version: 1.0
  tags: github, release, backend, spring-boot
---

**触发关键词（用户输入任意之一即可激活）：**  
生成测试、写测试代码、创建单元测试、生成 JUnit 测试、运行测试、执行测试、测试报告、只测试、全部测试、测试 UserService 等

**AI 必须严格遵循的执行流程（一步都不能跳过）：**

### 步骤 1：意图判断（必须第一步）
分析用户本次输入，确定模式：
- **生成模式**：用户提到「生成」「写」「创建」「生成测试代码」等 → 需要先生成测试代码（可能后续执行）
- **仅测试模式**：用户只提到「运行」「执行」「测试」「跑测试」「生成报告」等 → 跳过生成，直接执行测试
- **指定模式**：用户明确说「只生成 XXX」「只测试 UserService」「测试 login 方法」→ 仅处理指定内容

如果用户未明确，默认为「生成 + 测试」模式。

### 步骤 2：项目扫描与业务接口统计（必须完成）
1. 检测构建工具（优先找 pom.xml → Maven；否则 build.gradle → Gradle）。
2. 扫描 `pom.xml` 或 `build.gradle`，检查是否包含 `spring-boot-starter-test` 依赖。若未包含，必须先询问用户是否需要自动添加依赖。
3. 扫描 `src/main/java` 下所有带 `@Service` 注解的类。
4. 提取「业务接口」定义：每个类的 **public 方法**（排除 getter/setter、toString、equals、hashCode 以及明显内部方法和非业务方法）。
5. 检查 `src/test/java` 中对应测试类（包名镜像，例如 `com.example.service.UserService` → `com.example.service.UserServiceTest`）。
   - 若测试类不存在 → 该业务接口全部需要生成。
   - 若测试类存在 → 解析所有 `@Test` 方法名，判断哪些业务方法已有测试覆盖（匹配 `testXXX`、`shouldXXX`、`XXXSuccess`、`XXXFail` 等命名）。
   - 注意识别**多模块项目**。如果根目录下没有 src/main/java，请自动寻找包含 @Service 的子模块，并在该子模块对应的 src/test/java 中生成测试。
6. 统计**需要生成测试的业务接口数量**（每个方法算 1 个）。
   - 如果用户指定了某些类/方法，只统计指定范围。
   - 如果总数 **≤ 5** → 直接进入生成。
   - 如果总数 **> 5** → 先输出分批计划，例如：
     ```
     【分批计划】
     总共 8 个业务接口需要生成测试
     批次 1（5 个）：UserService.login、UserService.register、OrderService.createOrder...
     批次 2（3 个）：ProductService.getList...
     请回复「继续生成批次1」或「全部自动分批」开始。
     ```
     每批生成完成后询问是否继续下一批。

### 步骤 3：生成测试代码（仅生成模式）
- **包名规则**：与被测类完全一致的包路径（src/test/java/...）
- **测试类划分规则**（严格遵守）：
  - 优先按 **Service 层逻辑** 分组（例如 UserService 相关的所有 Service 方法统一放到 `UserServiceTest.java`）
  - **绝不允许** 把多个无关业务接口的测试写在同一个测试类里
- **每个业务接口必须包含**：
  - 至少 1 个正常逻辑测试（happy path）
  - 至少 1 个失败逻辑测试（异常、参数无效、空指针、业务异常等）
  - 使用 `@Mock` + `@InjectMocks`（Service）
  - 使用 JUnit 5 `@Test`、`@DisplayName`、`@ParameterizedTest`（合适时）
  - Mockito `when(...).thenReturn(...)` / `thenThrow(...)`
  - 断言使用 `assertThat`（AssertJ，已包含在 spring-boot-starter-test 中）
  - 严禁使用 @SpringBootTest，必须使用 `@ExtendWith(MockitoExtension.class)`，配合 `@InjectMocks (被测类)` 和 `@Mock (依赖类)`，保证测试秒级运行且无需加载 Spring 上下文。
  - 详细中文注释（说明测试场景）
- 生成完一批后，**实际写入文件** 到 `src/test/java` 对应位置。
- 每批生成完成后，总结「已生成 X 个测试类，Y 个测试方法」并询问是否继续下一批或「全部生成完成，是否开始执行测试？」。

### 步骤 4：测试执行（生成模式完成后或仅测试模式）
- 用户回复「开始测试」「执行」「跑」「yes」后才开始。
- 按测试类顺序依次运行（避免并发冲突）：
  - Maven 项目：`mvn test -Dtest=XXXTest -DfailIfNoTests=false`
  - Gradle 项目：`./gradlew test --tests XXXTest`
- 记录每个测试类的通过/失败/跳过数量。
- 如果在执行测试时遇到编译错误（如缺少依赖、找不到类）或测试运行失败，则跳过这次测试，记录在测试报告中，继续下一个类的测试。
- 支持用户指定「只测试 UserServiceTest」或「只测试 login 方法」。

### 步骤 5：生成测试报告（每次测试完成后必做）
- 报告文件名规范（避免混淆）：
  - `test-report-YYYYMMDD-HHMMSS.md`（例如 `test-report-20250307102235.md`）
  - 若为分批：`test-report-20250307102235-batch-1.md`
  - 存放位置：项目根目录下的 `test-reports/` 文件夹（不存在则自动创建）
- 报告内容（保持简洁）：
  ```
  # Spring Boot 单元测试报告
  生成时间：2025-03-07 22:35
  模式：生成+测试 / 仅测试
  测试范围：UserService、OrderService（共 12 个业务接口）
  
  通过：11/12
  失败：1
  跳过：0
  
  详细结果：
  - UserServiceTest → 8 passed, 0 failed
    • loginSuccess
    • loginFail_InvalidPassword
  - OrderServiceTest → 3 passed, 1 failed
    • createOrderFail_StockInsufficient ← 失败原因：...
  
  测试报告文件已保存至：test-reports/test-report-20250307102235.md
  ```
- 生成报告后，输出「测试完成！报告已生成，是否需要我帮你打开或分析失败用例？」

### 额外约束（AI 必须遵守）
- 永远不把所有测试放在一个类里。
- 生成代码必须可直接编译运行（import 正确、注解完整）。
- 每次响应都要清晰告知当前进度（「正在扫描...」「第 1/3 批生成完成」「测试执行中...」）。
- 如果项目不是 Spring Boot 或缺少 spring-boot-starter-test，主动提醒用户。
- 用户随时可以说「停止」「下一批」「只测试 XXX」来中断或切换。

**Skill 结束标志**：当报告生成后，自动结束本次 Skill 执行，并回复「已完成测试，生成的测试报告为test-report-XXXXXXXXX」
