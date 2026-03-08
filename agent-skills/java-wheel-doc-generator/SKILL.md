---
name: java-wheel-doc-generator
description: >
  专为 Java 后端“轮子”项目（如 Spring Boot Starter、核心中间件、公共依赖包）设计的自动分析与文档生成器。
  通过全局扫描代码和现有文档，逆向梳理项目的核心架构、执行流程、设计模式以及从基础到高级的完整配置与扩展点（精确到类和方法）。
  生成的文档既能作为开发者的思路梳理工具，也能作为高质量的对外开源/团队内部使用指南。
license: MIT
metadata:
  author: renhao
  version: 1.0
  tags: java, documentation, spring-boot-starter, architecture, design-patterns
---

**触发关键词（用户输入任意之一即可激活）：**  
写项目文档、分析这个轮子、生成starter文档、分析设计思路、完善README、写设计文档、分析扩展点等关键词。

**AI 必须严格遵循的执行流程（一步都不能跳过）：**

### 步骤 1：现有上下文收集与意图对齐（必须第一步）
1. **寻找历史文档**：全局搜索项目根目录及 `docs/` 目录下的 `README.md`、`DESIGN.md` 或其他 Markdown 文档。
2. **读取核心意图**：如果存在旧文档，仔细阅读其中的“背景”、“目的”或“快速开始”，理解作者造这个轮子的**初衷**和**核心解决的问题**。
3. **输出提示**：告知用户“已找到现有文档 XXX，将在此基础上进行深度代码分析与内容补全/重构”或“未找到现有文档，将从零开始构建架构分析文档”。

### 步骤 2：核心机制与流转全景扫描（核心大脑工作）
AI 必须对整个项目进行以下深度的结构分析，提取关键元数据，**不要停留在只看 Controller/Service，要用架构师的视角看代码**：
1. **入口追踪（Starter 机制）**：
   - 扫描 `spring.factories` 或 `org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件。
   - 追踪所有的 `@AutoConfiguration` / `@Configuration` 类。
   - 分析 `@ConditionalOn...` 注解，明确组件生效的前提条件。
2. **配置属性映射（Configuration）**：
   - 寻找所有带有 `@ConfigurationProperties` 的类。
   - 梳理出默认配置项是什么，支持哪些进阶配置。
3. **核心组件与接口梳理**：
   - 找出定义的顶层核心接口（Interfaces）和抽象类（Abstract Classes）。
   - 找出默认的核心实现类（Default Implementations）。
4. **扩展点与拦截机制（Hooks & Extensions）**：
   - 扫描自定义的 `BeanPostProcessor`、`ApplicationContextInitializer`、拦截器（Interceptors）或过滤器（Filters）。
   - 寻找标记了 `@ConditionalOnMissingBean` 的方法（这通常是留给使用者的扩展点）。

### 步骤 3：设计模式与执行链路逆向分析
1. **执行流程梳理**：从框架初始化（Init） -> 请求/任务触发（Trigger） -> 核心处理（Process） -> 结果收尾（Destroy/Return），构建一条完整的数据流转闭环链路。
2. **设计模式识别**：分析代码中使用了哪些设计模式（例如：通过 Interface+多个实现类判断是否为**策略模式**；通过 `build()` 判断是否为**建造者模式**；通过 `AbstractClass` 里的具体方法和抽象方法判断是否为**模板方法模式**；通过代理类判断是否为**责任链/代理模式**等），并记录具体应用在哪些类上。

### 步骤 4：生成高质量架构与使用文档
汇总前面的分析，生成（或覆写更新，如果原本存在 `DESIGN_DOC.md`，需检查是否与实际代码实现不同的的地方，**一切以代码实际逻辑为准**）一份高质量的 Markdown 文档（命名为 `DESIGN_DOC.md` 放于 docs 包下）。**文档必须严格包含以下结构**：

#### 1. 轮子定位与核心功能 (Overview)
- 一句话说明这是什么（如：基于 Redisson 的分布式锁 Starter）。
- 解决了什么痛点？核心 Feature 列表。

#### 2. 内部执行流转与架构设计 (Architecture & Flow)
- **启动与自动装配流程**：详细说明该依赖引入后，Spring 容器是如何把核心 Bean 注入的。
- **核心执行链路**：用文字或 Mermaid 流程图描述一次核心业务的完整流转过程（从 A 类 -> B 接口 -> C 实现）。
- **设计模式应用**：明确指出项目中使用了哪些设计模式，并在旁边注明对应的**核心类名**（让其他开发者秒懂你的代码品味）。

#### 3. 基础配置与快速使用 (Quick Start)
- 列出最简略的 `application.yml` / `application.properties` 配置示例。
- 给出基础调用的 Java 代码示例。

#### 4. 核心扩展点与高级配置 (Advanced & Extension Points) - 【重点】
*(这一部分必须向读者清晰解释“完整配置在简略配置基础上做了什么扩展”，以及“代码在什么地方可以被重写”)*
- **属性全览**：列出所有可配置属性及其默认值、作用。
- **自定义扩展指南**：详细列出用户如果想替换默认行为，需要实现**哪个类的哪个方法**。
  - *示例格式*：
    - **扩展点：自定义缓存淘汰策略**
    - **涉及接口/类**：`com.example.strategy.CacheEvictStrategy`
    - **操作方法**：实现该接口的 `doEvict()` 方法，并将其注册为 Spring Bean（框架内部由于使用了 `@ConditionalOnMissingBean`，会自动替换默认的 `DefaultCacheEvictStrategy`）。

### 额外约束（AI 必须遵守）
1. **拒绝流水账**：绝对不允许按文件的首字母顺序逐个翻译代码！必须按照“请求流转”或“架构分层”的逻辑来写。
2. **精准到方法级**：在描述扩展点时，必须给出具体的类路径和方法名，不能含糊其辞地说“实现某个接口”。
3. **发现缺陷主动提示**：如果在扫描代码时发现明显的设计缺陷（如：循环依赖风险、配置类缺少属性前缀、AutoConfiguration 未处理顺序等），在生成文档前，向用户单独输出一段【架构师建议】。
4. **输出确认**：分析完成后，先在对话框中输出大纲，询问用户：“以上梳理的逻辑是否符合你的真实设计意图？是否有需要调整的侧重点？”，确认后再写入 Markdown 文件。
