# 🦐 杨小虾的 Skill 编写教程

> 从零开始，学会为 OpenClaw 编写你自己的 Skill

---

## 📚 目录

1. [什么是 Skill？](#1-什么是-skill)
2. [Skill 的工作原理](#2-skill-的工作原理)
3. [Skill 的目录结构](#3-skill-的目录结构)
4. [SKILL.md 详解](#4-skillmd-详解)
5. [Bundled Resources（可选资源）](#5-bundled-resources可选资源)
6. [编写流程6步法](#6-编写流程6步法)
7. [实战练习：写一个「短信助手」Skill](#7-实战练习写一个短信助手-skill)
8. [测试与打包](#8-测试与打包)
9. [小测](#9-小测)

---

## 1. 什么是 Skill？

**Skill** 是模块化的「能力包」，用于扩展 OpenClaw 的功能。

它本质上是一个**配置文件 + 可选资源文件**，当你需要做某类特定任务时，OpenClaw 会读取对应的 Skill 来获得「专业知识」。

类比：
- OpenClaw 是一个「通才」，什么都会但不精通
- Skill 就像「专业书籍」，让 OpenClaw 在某个领域变成专家
- 你告诉 OpenClaw「帮我写一篇微信运营文案」，它就会去读 `wechat-copywriting` Skill

---

## 2. Skill 的工作原理

```
用户发送请求
     ↓
OpenClaw 判断该用哪个 Skill（根据 SKILL.md 的 description 匹配）
     ↓
加载对应 Skill 的 SKILL.md（按需加载，不是每次都全加载）
     ↓
按照 Skill 里的指引执行任务
     ↓
返回结果给用户
```

**核心机制：触发式加载**

OpenClaw 不是每次都读所有 Skill，而是根据用户的请求**智能匹配**。
`description` 写得好不好，直接决定 Skill 能不能被正确触发。

---

## 3. Skill 的目录结构

```
skill-name/              ← 文件夹名 = Skill 名（全小写，用-连接）
├── SKILL.md             ← 核心文件：元数据 + 指令（必须）
├── scripts/             ← 可选：可执行脚本（Python/Bash等）
├── references/          ← 可选：参考资料（按需加载）
└── assets/             ← 可选：资源文件（模板、图片等）
```

**最小结构只需要一个 SKILL.md**：

```
my-first-skill/
└── SKILL.md
```

---

## 4. SKILL.md 详解

SKILL.md 是 Skill 的核心，由两部分组成：

### 4.1 YAML Frontmatter（元数据）

```yaml
---
name: my-skill              # Skill 名称（英文小写+中划线）
description: 这段文字是触发依据，OpenClaw 根据它判断何时使用这个 Skill。
              写清楚：做什么 + 什么场景触发。
              示例：「当用户需要处理 Word 文档（.docx）时使用此 Skill，
              包括：创建、编辑、读取、格式转换。」
---
```

**⚠️ description 是关键！** 它决定了 Skill 何时被触发。

### 4.2 Body（正文指令）

用 Markdown 写指令，可以包含：
- 工作流程
- 代码示例
- 工具调用方法
- 参考资料链接

**原则：简洁优先**
- OpenClaw 已经很聪明，不要写太基础的步骤
- 重点写「它不知道的」和「容易错的」

---

## 5. Bundled Resources（可选资源）

### 5.1 `scripts/` - 可执行脚本

放需要反复运行的稳定逻辑：

```
my-skill/
├── SKILL.md
└── scripts/
    └── rotate_pdf.py     ← PDF 旋转脚本
```

使用时在 SKILL.md 里说明调用方式即可。

### 5.2 `references/` - 参考文档

放大量细节信息，按需读取：

```
my-skill/
├── SKILL.md
└── references/
    ├── finance.md        ← 财务相关
    ├── sales.md           ← 销售相关
    └── api-docs.md        ← API 文档
```

SKILL.md 只写核心流程，详细内容放到 references/ 里。

### 5.3 `assets/` - 资源文件

放输出时需要用到的文件（模板、图片等）：

```
my-skill/
├── SKILL.md
└── assets/
    ├── logo.png
    └── email-template.html
```

---

## 6. 编写流程 6 步法

```
Step 1: 理解需求 → 找具体例子，搞清楚 Skill 要做什么
Step 2: 规划内容 → 确定需要哪些 scripts/references/assets
Step 3: 初始化   → 用 init_skill.py 生成模板
Step 4: 编辑实现 → 写 SKILL.md 和资源文件
Step 5: 打包     → 用 package_skill.py 打包
Step 6: 测试迭代 → 实际使用，发现问题改进
```

### Step 1: 理解需求（最重要）

在动手写之前，先想清楚：

1. **这个 Skill 要解决什么问题？** 具体场景是什么？
2. **用户会怎么说？** 给出几个真实的使用例子
3. **OpenClaw 看到这个请求时，应该触发这个 Skill？** description 怎么写才能匹配？

### Step 2: 规划内容

问自己：
- 哪些步骤需要写死在 SKILL.md 里？
- 哪些是可选的、细节性的内容，放到 references/？
- 哪些是重复性的代码，放到 scripts/？
- 哪些是输出时需要的文件，放到 assets/？

### Step 3: 初始化

```bash
# OpenClaw 提供了一个初始化脚本
scripts/init_skill.py <skill-name> --path <output-path> [--resources scripts,references,assets]
```

### Step 4: 编辑实现

按照模板，填充：
- 写 `description`（反复推敲）
- 写工作流程和指令
- 编写或放入资源文件

### Step 5: 打包

```bash
scripts/package_skill.py <path/to/skill-folder>
```

打包过程会自动验证格式，验证失败会报错。

### Step 6: 测试迭代

安装后实际使用，发现问题继续改进。

---

## 7. 实战练习：写一个「短信助手」Skill

### 题目

帮老大写一个 Skill，用于处理 BOOX 墨水屏手机收不到短信的问题。

### Step 1 & 2: 需求分析和规划

**这个 Skill 要做什么：**
- 帮用户在 BOOX 文石墨水屏上解决短信/电话功能缺失的问题
- 诊断问题原因
- 提供解决方案（刷机/转发/换方案）

**需要包含的内容：**
- references/boox-p6-pro.md - 存放 P6 Pro 的技术细节和常见问题
- references/sms-forwarder-guide.md - SMS Forwarder 配置指南
- SKILL.md - 核心流程和判断逻辑

### Step 3: 初始化

假设我们已经有了 `init_skill.py`，初始化命令：

```bash
scripts/init_skill.py boox-sms-helper --path ./skills --resources references
```

### Step 4: 编辑实现

#### SKILL.md 编写

```yaml
---
name: boox-sms-helper
description: 帮助用户在 BOOX 文石墨水屏设备上解决短信和电话功能问题。
              当用户提到以下场景时使用：
              - BOOX/文石/墨水屏 收不到短信/无法发短信/没有电话功能
              - 墨水屏设备短信功能故障排查
              - 文石设备刷机/ROM/电话功能
              - P6 Pro/Spoke Pro 等文石手机设备
---
```

**正文内容：**

```markdown
# BOOX 短信助手

## 问题诊断流程

当用户报告 BOOX 设备短信/电话问题时，按以下流程诊断：

### 第一步：确认问题类型

| 症状 | 可能原因 |
|------|---------|
| 完全不能发短信/打电话 | 系统层功能被禁用 |
| 能读SIM但发不了 | SMSC号码未配置 / VoLTE未开 |
| SIM卡完全无服务 | 硬件或卡槽问题 |

### 第二步：收集信息

请用户执行以下命令并返回结果：

```bash
# 确认设备连接
adb devices

# 查看电话服务状态
adb shell dumpsys telephony.registry

# 查看短信APP列表
adb shell pm list packages | findstr sms
```

### 第三步：根据诊断结果提供方案

**方案A：系统层禁用（常见于国行文石）**
- 尝试用 ADB 开启：
  ```bash
  adb shell settings put global telephony_disabled 0
  adb reboot
  ```
- 若无效，考虑刷机

**方案B：SMSC未配置**
- 设置 → 短信中心号码 → 填入运营商SMSC
- 联通：+8613010112500

**方案C：换方案**
- 用短信转发器（另一台手机接收 → 转发到邮箱）
- 改用邮箱接收验证码

## 详细参考资料

- BOOX P6 Pro 专项 → [references/boox-p6-pro.md](references/boox-p6-pro.md)
- SMS Forwarder 配置 → [references/sms-forwarder-guide.md](references/sms-forwarder-guide.md)
```

#### references/boox-p6-pro.md 编写

```markdown
# BOOX P6 Pro 技术参考

## 设备信息
- 型号：P6 Pro（小白马）
- 系统：BOOX OS 4.1.1
- CPU：骁龙处理器

## 已知问题
- 国行版本可能默认禁用 telephony 组件
- SMS 和 Voice 服务在 ROM 层被屏蔽

## 刷机注意事项
- 需先解锁 BL 锁
- 推荐rom：XDA 论坛 BOOX P6 Pro 区

## ADB 常用命令
```bash
# 查看 telephony 服务状态
adb shell dumpsys telephony.registry

# 检查短信APP
adb shell pm list packages | findstr mms

# 设置默认短信APP
adb shell settings put secure default_sms_app com.android.mms
```
```

---

## 8. 测试与打包

### 打包

```bash
scripts/package_skill.py ./skills/boox-sms-helper
```

成功后会生成 `boox-sms-helper.skill` 文件。

### 安装

把 `.skill` 文件放到 OpenClaw 的 skills 目录即可使用。

---

## 9. 小测

学完了，来做道题检验一下 👇

---

### 测验题目

老大想让 OpenClaw 帮他处理「公众号运营」相关的工作，请你写一个 Skill。

**要求：**

1. 创建目录结构（只需骨架）
2. 写好 `name` 和 `description`
3. 写一个简单的工作流程（至少3步）
4. 设计是否需要 references/ 或 scripts/，需要哪些

---

### 参考答案（先自己做，做完再看）

<details>

**目录结构：**
```
wechat-copywriting/
├── SKILL.md
└── references/
    └── content-types.md
```

**SKILL.md：**

```yaml
---
name: wechat-copywriting
description: 辅助公众号运营和推文撰写。当用户提到：
              - 写公众号文章/推文/内容
              - 公众号选题/标题/文案
              - 微信图文消息
              - 运营方案/粉丝互动
              使用此 Skill。
---

# 公众号运营助手

## 工作流程

1. **确认账号定位**
   - 账号类型是什么？（资讯/产品/个人IP/品牌）
   - 目标用户是谁？
   - 平时发什么内容？

2. **分析需求**
   - 是写新文章还是改稿？
   - 有没有热点要追？
   - 需要什么风格？（正式/轻松/活泼/深度）

3. **撰写/优化**
   - 标题：吸引点击，≤14字
   - 开头：制造悬念或共鸣
   - 正文：逻辑清晰，适当分段
   - 结尾：引导互动（评论/转发/关注）

## 参考资料

内容类型指南 → [references/content-types.md](references/content-types.md)
```

**references/content-types.md：**
```markdown
# 公众号内容类型参考

## 常见内容类型
- 干货类：实用性强，干货满满
- 故事类：情感共鸣，人设打造
- 热点类：借势流量，快速传播
- 产品类：软广植入，场景营销
```

</details>

---

## 🎉 恭喜完成！

有问题随时问，继续练习可以尝试写其他场景的 Skill 🦐
