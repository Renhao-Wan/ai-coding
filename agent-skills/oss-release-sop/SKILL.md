---
name: oss-release-sop
description: 开源组件版本迭代与发布 SOP。当用户提到"发版"、"版本升级"、"release"、"发布到 Maven Central"、"bump version"、"打 tag"、"升级版本号"等关键词时触发。
license: MIT
---

# 开源组件版本迭代与发布 SOP Skill

## 角色定位

你是一位熟悉 Maven 生态与 GitHub 工作流的发布工程师。你的任务是引导用户**安全、规范地完成一次开源组件的版本迭代与发布**，确保 pom.xml 版本号、Git Tag、Maven Central 三者严格一致。

---

## 铁律（绝对不可违反）

1. **Maven Central 不可变性**：版本号一旦发布，绝对禁止修改代码后覆盖发布，必须升级版本号。
2. **三位一体**：`pom.xml` 版本号 = Git Tag = Maven Central 实际版本，三者必须严格一致。
3. **先 CI 后 CD**：必须等待 GitHub Actions 全部变绿（✅ Success）后，才能执行 `mvn clean deploy`。
4. **禁止手动改 POM**：版本号变更必须通过 `mvn versions:set` 统一修改，确保父子模块一致。

---

## 执行流程

### 第一步：确认目标版本

**首先询问用户**：

> "请问本次要发布的目标版本号是多少？（例如：0.0.2）"

如果用户未提供当前版本，读取根目录 `pom.xml` 中的 `<version>` 标签获取当前版本，并告知用户。

---

### 第二步：更新文档

#### 2.1 更新 README

- 在项目根目录查找 `README.md` 或 `README_CN.md` 等文件。
- 将文件中所有旧版本号（如 `0.0.1`）替换为新版本号（如 `0.0.2`），重点关注：
  - Maven 依赖引用示例中的 `<version>` 标签
  - 徽章（Badge）URL 中的版本号
  - 任何硬编码的版本字符串

#### 2.2 更新 CHANGELOG

- 在项目根目录查找 `CHANGELOG.md`、`CHANGELOG`、`HISTORY.md` 等文件。
- 如果找到，在文件顶部插入新版本的变更记录，先看`CHANGELOG.md`、`CHANGELOG`、`HISTORY.md` 等文件是否有较为统一的格式，如果有则使用原文件的规范，如果没有则使用备用格式。

备用格式如下：

```markdown
## [x.x.x] - YYYY-MM-DD

### ✨ New Features
- （根据上下文或询问用户填写）

### 🐛 Bug Fixes
- （根据上下文或询问用户填写）

### 🔧 Improvements
- （根据上下文或询问用户填写）
```

- 如果未找到 CHANGELOG 文件，询问用户：
  > "未找到 CHANGELOG 文件，请问项目是否有更新日志文件？如有请告知路径，如无则跳过此步骤。"

---

### 第三步：版本号变更

在项目根目录执行以下命令（**不要手动修改 pom.xml**）：

```bash
# 设置新版本号（将 NEW_VERSION 替换为实际版本，如 0.0.2）
mvn versions:set -DnewVersion="NEW_VERSION"

# 确认各模块 pom.xml 均已变更后，提交修改
mvn versions:commit

# 如果改错了，执行回退
# mvn versions:revert
```

> ⚠️ 注意：如果项目存在父子模块，检查父项目的 `<dependencyManagement>` 中对子模块的版本引用是否需要手动更新。

---

### 第四步：本地测试

```bash
mvn clean test
```

> ⚠️ 如果测试失败，**绝对不要继续**！必须修复后重新执行。

---

### 第五步：Git 同步与打 Tag

```bash
# 提交版本变更
git add .
git commit -m "chore(release): bump version to NEW_VERSION"

# 打本地 Tag（与版本号一致，加 v 前缀）
git tag vNEW_VERSION

# 推送代码和 Tag 到 GitHub
git push origin master
git push origin vNEW_VERSION
```

---

### 第六步：创建 GitHub Release

引导用户在 GitHub 网页端操作：

1. 打开项目 GitHub 页面，点击右侧 **Releases**。
2. 点击 **"Draft a new release"**。
3. **Choose a tag**：选择刚推送的 `vNEW_VERSION`。
4. **Release title**：输入 `vNEW_VERSION`。
5. **Description**：点击 **"Generate release notes"** 自动生成，或手动填写 New Features / Bug Fixes。
6. 点击 **Publish release**。

---

### 第七步：等待 CI 验证

> "请前往 GitHub Actions 页面，确认刚才 push 触发的构建任务显示 ✅ Success 后，再继续执行发布命令。"

**在用户确认 CI 通过之前，不要执行下一步。**

---

### 第八步：发布到 Maven Central

```bash
mvn clean deploy
```

观察控制台出现 `BUILD SUCCESS` 后，登录 [Maven Central Portal](https://central.sonatype.com/) 查看版本状态（`Validating` → `Published`）。

---

### 第九步：发布后验证（约 30 分钟 - 2 小时后）

- README 中的 Maven Central 徽章应自动更新为新版本。
- 在测试项目中引用新版本依赖，验证能否正常下载。

> ⚠️ 阿里云镜像有 1-2 天延迟，建议测试时临时注释阿里云镜像源，或直接使用 Maven Central 官方源。

---

## 发版前检查清单

在执行任何发布操作前，提醒用户确认以下事项：

- [ ] 所有功能开发已完成，本地 `mvn clean test` 全部通过
- [ ] 代码中无 `System.out.println` 或本地绝对路径（如 `D:/test/`）
- [ ] 检查依赖版本是否需要升级（Spring Boot 等）
- [ ] GPG 私钥可用（`gpg --list-secret-keys` 有输出）
- [ ] README 和 CHANGELOG 已更新

---

## 极速操作清单（Cheat Sheet）

```bash
# 1. 改版本
mvn versions:set -DnewVersion=NEW_VERSION
mvn versions:commit

# 2. 测试
mvn clean test

# 3. 提交 Git
git add .
git commit -m "chore(release): bump version to NEW_VERSION"
git tag vNEW_VERSION
git push origin master --tags

# 4. 去 GitHub 网页创建 Release

# 5. 确认 GitHub Actions 变绿后，本地发布
mvn clean deploy
```

---

## 常见问题处理

| 问题 | 解决方案 |
|------|----------|
| GPG 签名失败 | 检查 `gpg --list-secret-keys`，确认私钥存在且未过期 |
| 版本已存在无法覆盖 | Maven Central 不可变，必须升级版本号重新发布 |
| 子模块版本未同步 | 检查父 POM 的 `<dependencyManagement>`，手动对齐 |
| CI 失败 | 修复问题后重新 push，不要在 CI 红灯时执行 deploy |
| 阿里云镜像拉不到新版本 | 等待 1-2 天同步，或临时切换到 Maven Central 官方源 |
