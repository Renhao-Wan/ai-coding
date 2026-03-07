---
name: github-release-note-generator
description: 帮助用户编写高质量、结构化、中英双语的 GitHub Release 公告。深谙后端编程领域（尤其是 Java / Spring Boot）的专业术语，能够根据用户提供的“简短更新点”，自动进行技术上的专业扩写，解释其背后的架构意义（如：解耦、开箱即用、策略模式、前后端分离等）。
license: MIT
metadata:
  author: renhao
  version: 1.0
  tags: github, release, backend, spring-boot
---

# 技能作用
当用户输入类似“现在需要写一个 GitHub 发布公告”“帮我生成发版说明”“release note for vX.Y.Z”等话语时，自动触发此技能。技能专注于后端编程项目（如 Spring Boot、Java 相关），会扩写用户提供的更新/新增内容，使其更专业和技术化（例如，解释为什么某个变更提升了性能、安全性或可扩展性）。固定部分包括“如何升级”（How to Upgrade），通常是更新 pom.xml 中的 dependency 版本。

## 输入变量 (Input Variables)
在开始生成前，你需要确保已从用户处获取以下信息（若未提供，请主动询问）：
1. **项目名称 (`artifactId`)**：如 `multi-login-spring-security-starter`。
2. **组织名称 (`groupId`)**：如 `io.github.renhao-wan`（为默认值，需询问用户是否确认）。
3. **版本号 (`version`)**：如 `0.0.1` 或 `0.0.5`。
4. **更新或新增的内容**：用户提供的口语化或简短的更新点。
5. **发布类型**：选择是“首次发布 (Initial Release)”还是“版本更新 (Update/Enhancement)”。

## 技能执行逻辑 (Workflow)
1. **智能扩写**：仔细分析用户提供的更新点，结合后端编程语境进行专业扩写。
   - 提取每个更新点的核心功能，提炼出具有吸引力的**中英双语小标题**。
   - 补充该功能在实际业务场景中的价值（Why）。
   - 描述该功能的具体实现或修复结果（Fix/Feature）。
   - 分组到合适的部分：Features（新功能）、Improvements & Fixes（改进和修复）、Configuration Hierarchy（配置层次，如果相关）等。
   - “扩写”部分是你的核心任务。扩写的质量直接影响公告的专业度和吸引力。如果用户提供的信息不够清晰或详细，你需要主动提问以获取更多细节。
2. **双语适配**：公告的引言、主标题需支持中英双语。
3. **Markdown 严格排版**：必须完全遵循下方规定的 Markdown 模板格式，包括 Emoji 的使用。
4. **固定依赖更新**：根据用户提供的 `groupId`、`artifactId` 和 `version`，自动生成正确的 Maven 依赖 XML 代码块。

## 输出模板规范 (Output Templates)

**👉 场景一：首次正式发布 (如果是 v0.0.1 或明确说明是 Initial Release)**

```markdown
# 🎉 Initial Release v{版本号}

这是 `{项目名称}` 的第一个正式版本！
This is the first official release.

## ✨ Features (功能特性)

{基于用户的输入，扩写出3-5个核心特性，格式如下：}
*   🚀 **{英文特性标题}({中文特性标题})**：{扩写该特性的技术细节和业务价值，如：集成xxx，支持xxx，解耦xxx}。
*   🛠️ **{英文特性标题}({中文特性标题})**：{扩写说明，如：基于xxx模式重构，提升扩展性}。

## 📦 Installation (安装)

<dependency>
    <groupId>{groupId}</groupId>
    <artifactId>{artifactId}</artifactId>
    <version>{版本号}</version>
</dependency>
```

**👉 场景二：版本更新 (非首次发布，如 v0.0.5 等)**

````markdown
# 🚀 v{版本号} - {提取本次更新的核心一句话总结_中文} ({一句话总结_英文})

{对本次更新的总体背景和影响进行一段双语说明，例如：本次更新引入了全局配置支持与自定义策略，大幅提升了对前后端分离场景的适配能力。 / This release introduces global configuration support...}

## ⚙️ Improvements & Fixes (改进与修复)

{基于用户的输入，扩写每个更新点，严格按照以下格式：}
*   **{核心特性_英文标题}({中文特性标题})**：{中文特性说明，如：新增 global 配置层级，支持设置默认参数提取器}。
    *   `o Why:` {中文说明为什么要做这个更新，解决了什么痛点。如：减少冗余配置，允许统一行为}。
    *   `o Fix/Feature:` {中文说明具体做了什么改动。如：Added global configuration level...}。
*   **{核心特性_英文标题}({中文特性标题})**：{中文特性说明}。
    *   `o Why:` {中文原因说明}。
    *   `o Fix/Feature:` {中文改动说明}。

## 📦 How to Upgrade (升级)

在 `pom.xml` 中更新版本号为 {版本号}：

```xml
<dependency>
    <groupId>{groupId}</groupId>
    <artifactId>{artifactId}</artifactId>
    <version>{版本号}</version>
</dependency>
```
````
*(注意：外层不要多余的 markdown 代码块标记，保持和s上面一致的纯文本带 xml 缩进风格，或者仅使用 xml 代码块。)*

## 约束与语气 (Constraints)
- 语气需专业、极客、有亲和力。
- “扩写”部分是你的核心任务。例如，用户如果只输入“加了全局配置”，你需要扩写为“全局配置支持 (Global Configuration Support)”，并补充 Why（如“解决原生只支持表单的局限，完美支持 JSON Body 参数解析”）和 Feature。Why 和 Feature 的说明如果不够清晰，你需要主动询问用户以获取更多细节。
- XML 代码块中的标签必须严格匹配 Maven 格式。
