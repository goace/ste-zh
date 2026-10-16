# ste-zh：简化技术中文规范

**语言 / Language：** 简体中文 | [繁體中文](https://github.com/dualface/ste-zht)

一个面向 AI Agent 的任务汇报规范 skill。借鉴航空与工业界严谨的 **ASD-STE100**（Simplified Technical English，简化技术英语）原则，约束 Agent 用清晰、严密且无歧义的中文汇报工作结果。

Agent 的日常汇报常常过程冗长、推测含糊，或者漏掉关键的验证证据。`ste-zh` 为适用的工作汇报提供受控中文规范：**关键信息先行、实证说话、状态固定、一目了然**。

> 本 skill 只用于向用户汇报工作结果、实质进度、排查结论、方案取舍和多项工作总结。简短事实问答、普通交流、写给 agent 的材料及其草稿不适用；代码、注释、Commit 信息和写入项目的文件也不适用。完整判断顺序见 [适用范围](SKILL.md#适用范围)。

---

## 为什么需要 ste-zh？

同一个 Bug 修复任务，常见汇报与规范化汇报的差异如下。以下摘录以 [结果汇报示例](examples/result-report.md) 为准，示例中的来源事实见该文件。

**常见 AI 汇报（冗长、推测、不报实证）：**

> 「好的，我先看了一下登录相关的代码，发现问题应该是出在 token 刷新的逻辑上，大概是并发的时候两个请求同时去刷新导致的。然后我对 `auth/refresh.ts` 进行了修改，加了一个锁来实现串行化处理，另外顺手把一些日志之类的也优化了一下。测试基本都跑过了，应该没什么问题，不过 iOS 那边因为没有环境暂时没法验证。你看要不要我再把重试逻辑也一起改了？」

**`ste-zh` 规范化汇报（结论先行、实证可查、便于决策）：**

> 已完成：登录刷新逻辑的代码修改。iOS 端未验证。
>
> **原因**：两个请求同时刷新 token。后完成的刷新使先拿到的 token 失效。
>
> **证据**：
>
> - `auth/refresh.ts:42` — 刷新前不检查是否已有刷新在进行。
>
> **改动**
>
> - `auth/refresh.ts`：同一时间只发一个刷新请求。其他请求等待这次刷新的结果。
> - `auth/logger.ts`：删除 3 处重复日志。这项改动不在原范围内。
>
> **验证**
>
> - 已验证：`npm test auth` — 48 项通过，0 项失败。包含新增并发刷新测试 `auth/refresh.test.ts:88`。
> - 未验证：iOS 端 — 本次没有执行 iOS 端检查。本机没有 iOS 构建环境。
>
> **待确认**
>
> - 是否保留 `auth/logger.ts` 的改动：
>   1. 撤回（推荐）— 本次提交只含登录修复。
>   2. 保留 — 日志清理和登录修复一起提交。

---

## 核心特性

- **先判断适用范围**：加载不等于所有回复都适用。每条回复、每个内容部分分别判断；简短事实问答和普通交流不执行规则、模板或自检。
- **关键信息先行**：适用内容的首句按汇报类型写。结果写状态与完成范围，进度写当前动作，需求总结写总结范围。简短汇报只写必要句子。
- **状态固定**：工作状态与验证状态分别表述，且各自限定范围。「已完成」不自动表示「已验证」。完整定义见 [术语表第 4 节](references/terminology.md#4-状态词)。
- **如实写验证**：分清事实、推断和假设。事实必须有来源支持，推断写明依据，假设写明依据和核实方法。修复效果、运行行为和检查结果写明检查方法、结果与覆盖范围。有依据的分析可以保留，但不能冒充已验证的运行结果。进度、建议、需求摘要和待决事项不强加验证状态；其中的实际效果断言仍须写验证。资料内容证据不能代替运行验证。
- **一词一义与主动语态**：同一个概念只用同一个叫法，不同概念保留区别；消除虚化动词与歧义长句；短句以不损失原意为前提，条件、路径等完整信息优先。
- **选项化决策**：存在真实备选方案时，整理为编号选项，有充分理由时才标推荐；需要填写名称、路径或数值时直接提问。需要补齐信息时先自行查证，只有影响当前工作、且无法自行确定时才询问用户。

---

## 安装

### 推荐：使用 `skills` CLI 安装

[`skills`](https://github.com/vercel-labs/skills) 是 Vercel Labs 推出的 Agent Skill 管理工具，支持 Claude Code、Cursor、Codex 等主流环境。本 fork 的技能名称和安装目录名统一为 `ste-zh`。

全局安装（所有项目通用）：

```bash
npx skills add goace/ste-zh -g
```

安装到当前项目：

```bash
npx skills add goace/ste-zh
```

仅为指定 Agent 安装（例如 Claude Code）：

```bash
npx skills add goace/ste-zh -g -a claude-code
```

更新与卸载：

```bash
npx skills update
npx skills remove --global ste-zh
```

### 手动安装

将本仓库克隆至 Agent 的 skills 目录下，目标目录命名为 `ste-zh`。

**Codex（用户级）：**

```bash
git clone https://github.com/goace/ste-zh.git ~/.agents/skills/ste-zh
```

目录依据见 [Codex 技能文档](https://developers.openai.com/codex/skills/)。

**Claude Code（全局）：**

```bash
git clone https://github.com/goace/ste-zh.git ~/.claude/skills/ste-zh
```

其他支持 `SKILL.md` 规范的 Agent，参考各自文档将文件放置在对应的 skills 路径下。

---

## 使用方式

安装后，技能可供 Agent 发现。若希望 Codex 每个会话都调用它，将下面的内容加入全局 AGENTS.md（默认位于 `~/.codex/AGENTS.md`）：

```markdown
## 向用户汇报

每个会话调用 `ste-zh` 技能，并严格按其适用范围执行。
仅在向用户汇报工作结果、实质进度、排查结论、方案取舍和多项工作总结时应用。
简短事实问答、普通交流、写给 agent 的材料及其草稿不适用。
一条回复包含多种内容时，分别判断，只约束适用的汇报部分。
```

AGENTS.md 的读取方式见 [Codex 文档](https://developers.openai.com/codex/guides/agents-md/)。以上只提供配置示例，安装技能不会自动修改全局规则。

调用技能与应用写法是两件事：Agent 先读取技能，再按当前内容判断。即使前一条是正式汇报，下一条简单问答仍按日常方式回答。给 agent 编写的材料始终按自身要求编写，不因放在聊天中供用户审阅而改用 STE。

具体对照见 [适用范围示例](examples/scope-boundaries.md)。用户明确指定的风格、格式和更高优先级指令优先。本 skill 依靠 Agent 遵守指令，不由程序强制执行。

---

## 实战配合：与 Kander 协同

作者在使用 [Kander](https://github.com/dualface/kander)（多 Agent 看板调度工具）并行调度多个任务时，重度依赖本 skill：

- **高效扫视**：看板上多张任务卡并行流转，每张卡的汇报首句给出当前关键信息，便于查看任务进展。
- **完成与验证分开**：代码修改执行完毕、运行检查未执行时，可以写「已完成：代码修改。运行效果未验证。」验收条件来自用户要求、适用的项目要求和任务目标。必需检查未执行或失败时，不得宣称整项任务已完成。已知失败不能改写成「已完成，未验证」。
- **极简决策**：存在真实备选方案时，阻塞点格式化为编号选项，在 Agent 会话中回复数字即可快速放行；需要填写名称、路径或数值时直接提问。

---

## 设计背景与 ASD-STE100 的关系

- **关于标准**：[ASD-STE100](https://www.asd-ste100.org/)（Simplified Technical English）由欧洲航空航天与国防工业协会（ASD）维护，原本是针对英文技术维护文档制定的受控语言规范，用于消除歧义与理解偏差。
- **中文改写**：本项目提炼了 ASD-STE100 的核心原则，并结合 AI 交互特点重构为一套中文实用规则体系。条目编号与内容均为独立设计，并非原文直译。
- **独立声明**：本项目为个人开源项目，未收录 ASD-STE100 原文与受控词典，与 ASD 组织无隶属关系。ASD-STE100 与 Simplified Technical English 的权利属于 ASD。如需研读 ASD-STE100 英文原版标准，可在其官网免费申请。

---

## 核心文件

| 路径                        | 说明                                                                              |
| --------------------------- | --------------------------------------------------------------------------------- |
| `SKILL.md`                  | skill 入口：加载与使用、适用范围、25 条核心规则、5 种标准化输出模板及自检清单       |
| `references/terminology.md` | 术语字典：ASD-STE100 关键词译法、受控情态词、虚化动词替换表、固定状态词与标点规范 |
| `examples/result-report.md` | 单任务结果汇报改写示例（改写前后对比）                                            |
| `examples/task-summary.md`  | 多任务与迭代需求总结改写示例（改写前后对比）                                      |
| `examples/scope-boundaries.md` | 适用与不适用的对照，包含简单问答、agent 草稿和混合回复                         |

---

## 许可证

基于 MIT 协议开源。详见 [LICENSE](LICENSE)。

---

## 作者的其他项目

欢迎体验作者 [dualface](https://github.com/dualface) 的其他项目：

- [Kander](https://github.com/dualface/kander)：规则驱动的多 Agent 看板调度工具，内置独立审核与自动化交付门禁。
- [Ullage](https://github.com/dualface/ullage-cli)：本地守护进程与 CLI 工具，查看 Claude、ChatGPT、Grok、Cursor 等订阅的用量。
- [QuickTUI](https://quicktui.ai/)：适用于各类编码 Agent 的手机端完整终端，支持自托管直连，单台主机免费。
