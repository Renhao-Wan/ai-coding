---
name: apache-license
description: 为项目生成完整的 Apache 2.0 开源协议文件套装，包含 LICENSE 英文原文、LICENSE-CN.md 中文说明、以及中英双语 NOTICE 文件
tags:
  - license
  - apache
  - open-source
  - 开源协议
license: MIT
---

# Apache 2.0 许可证文件生成器

## 职责

当用户需要为项目添加 Apache 2.0 开源协议时，自动生成以下三个文件：

1. `LICENSE` — Apache 2.0 英文官方原文（不可修改内容）
2. `LICENSE-CN.md` — 非官方中文说明（注明法律效力以英文原文为准）
3. `NOTICE` — 中英双语版权声明

## 触发关键词

- 添加 Apache 协议 / 添加开源协议
- 生成 LICENSE 文件
- apache license / apache 2.0
- 开源协议文件

## 执行步骤

### 第一步：收集信息

在生成文件前，向用户确认以下信息：
- **项目名称**（用于 NOTICE 文件）
- **版权年份**（默认当前年份）
- **版权所有者姓名或组织名**

### 第二步：生成 LICENSE

内容为 Apache 2.0 官方英文原文，**末尾附上版权声明块**，格式如下：

```
Copyright [年份] [版权所有者]

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

> LICENSE 正文为 Apache 官方标准文本，不得修改任何条款内容。

### 第三步：生成 LICENSE-CN.md

- 文件顶部必须有免责声明，注明本文件为非官方中文说明，法律效力以 `LICENSE` 英文原文为准
- 涵盖以下章节：授权与条件、再分发要求、专利授权、商标、免责声明、责任限制
- 文末附上英文原文链接和官方许可证地址

### 第四步：生成 NOTICE

格式为中英双语，结构如下：

```
[项目名称]
Copyright [年份] [版权所有者]

本产品基于 Apache License, Version 2.0 授权发布。
This product is licensed under the Apache License, Version 2.0.

您可以在以下地址获取许可证副本：
You may obtain a copy of the License at:
http://www.apache.org/licenses/LICENSE-2.0
```

## 规则与约束

- `LICENSE` 文件内容必须与 Apache 官方原文完全一致，不允许翻译或修改条款
- `LICENSE-CN.md` 仅作参考说明，必须在文件顶部注明非官方性质
- `NOTICE` 中英双语完全合法，Apache 2.0 不限制语言
- 三个文件均放置于项目根目录
- 生成完成后提示用户将文件加入 git 追踪并提交

## 输出示例

生成完成后告知用户：

```
已生成以下文件：
- LICENSE          （Apache 2.0 英文官方原文）
- LICENSE-CN.md    （非官方中文说明）
- NOTICE           （中英双语版权声明）

建议执行：
git add LICENSE LICENSE-CN.md NOTICE
git commit -m "chore: 添加 Apache 2.0 开源协议文件"
```
