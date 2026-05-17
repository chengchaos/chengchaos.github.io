---
title: 提升 Claude Code 使用效率
key: 2026-04-16
tags: Claude Code
---

如何才能做到呢? 

<!--more-->

## 0x01 CLAUDE.md 

CLAUDE.md 会在每一轮对话中加载.

**每一轮, 绝不例外**

整个加载体系层级:

| 文件 | 作用域 | 用途 |
| ---- | ----- | ---- |
| `~/.cluade/CLAUDE.md` | 全局 | 你的代码风格偏好 |
| `./CLAUDE.md` | 项目级 | 架构决策, 规范约定 |
| `.claude/rules/*.md` | 模块化规则 | 按功能拆分的规则文件 |
| `CLAUDE.local.md` | 私有 | 不纳入版本控制的个人笔记 |

有 40,000 个字符的额度. 把架构决策写进去. 文件命名规范, 测试模式. "绝对不能这样做" 的规范. 模型每一轮都会读取它们.

**如果你只能做一件事, 就做这一件好了.**

## 0x02 Subagent 

**共享 Prompt Cache (并行几乎 0 成本)**

当 Claude Code fork 一个 Subagent 时, 他会创建一个与父上下文 **字节级一致** 的副本. API 会缓存这个副本. 因此, 驻派 5 个 Agent 去处理代码库的不同部分, 其成本几乎与 1 个 Agent 顺序执行相同.

再读一遍.

**5 个 Agent, 和  1 个 Agent 成本相同, 因为它们都命中了 Prompt Cache.**

有 3 种 Subagent 执行模型:

| 模型 | 特性 | 使用场景 |
| ---- | ---- | ---- |
| fork | 继承父上下文, Cache 优化 | 并行任务(推荐) |
| teammate | 在 tmux/iTerm 中打开独立 | 多任务协作 |
| worktree | 独立的 git worktree , 每个 Agent 一个隔离分支 | 超大规模并行 |

我们可以告诉哦 Claude Code 同时启动 5 个 Agent: 一个做安全审计, 一个冲过认证模块, 一个写测试, 一个更新文档, 一个修 Bug -- 全部同时进行, 全部共享 Cache.

**这套架构就是 为并行而生的, 单线程使用, 简直暴殄天物**

## 0x03 权限系统

**设计初衷是让你配置他, 而不是让你点按钮**

一个 **5 级设置层级体系**:

$$policy > flag > local > project > user$$

在 `~/.claude/settings.json` 中, 可以用 Glob 模式设置永久允许的操作:

```json
{
    "permissions": {
        "allow": [
            "Bash(npm *)",
            "BAsh(git *)",
            "Edit(src/**)",
            "Write(src/**)"
        ]
    }
}

```

权限模式有三种:

| 模式 | 行为 | 风险 |
| ---- | ---- | ---- |
| bypass | 完全跳过权限检查 | 危险但最快 |
| allowEdits | 自动允许对工作目录的文件编辑 | 适中 |
| auto (新增) | 对每个操作运行 LLM 分析器判断 | 推荐, 平衡安全与效率 |

`auto` 模式有独立的 allow/deny 列表可以配置. 源码显示它会并行竞速多个解析器: 用户点击, Hook 分类器, Bridge, 先响应着胜出.

## 0x04 有 5 种压缩策略

**Context 压力是真实存在的问题**.

当对话过长时, 源码中有 5 中不同的对话压缩方式:

| # | 策略 | 机制 | 
| ---- | ---- | ---- |
| 1 | microcompact | 基于时间清除旧的工具结果 |
| 2 | context collance | 摘要一段对话 | 
| 3 | session memory | 将关键上下文提取到文件 | 
| 4 | full compact | 摘要整个历史 |
| 5 | PTL truncation | 丢弃最旧的消息组 |

着意味着:

- **主动使用 /compact**. 不要等系统自动压缩, 丢失珍贵的 Context.
- 默认上下文窗口是 200k Tokens, 可以使用 1m 模型后缀切换到 1M Tokens.
- 长时间会话积累 "Session Memory" -- 恢复会话比新开始更好.
- 大型工具结果会存储到磁盘, 只向模型发送 8kb 的预览.

**把 `/compact` 当做游戏里使用存档点 -- 保留重要的, 清除无用的 -- 继续前进**

## 0x05 Hook 系统才是真重的扩展 API

**(25+ 生命周期事件)**

着几乎是无人知晓的 Power User 功能.

源码揭示了 25+ 个可以 Hook 的生命周期事件:

| 类别 | 事件 |
| ---- | ---- |
| 工具执行 | `PreToolUse`, `PostToolUse` |
| 提示词 | `UserPromptSubmit`, `ModelResponse` |
| 会话 | `SessionStart`, `SessionEnd` |
| 文件 | `FileChanged`, `FileSaved` |
| Agent | `AgentStart`, `AgentEnd` |

**Hook 有 5 种类型**:

| 类型 | 作用 |
| ---- | ---- |
| command | 运行  shell 命令 |
| prompt | 通过 LLM 注入上下文 |
| agent | 运行完整的 AGent 验证循环 |
| http | 调用 Webhook |
| function | 运行 JavaScript |

**实际用例**:

- 每次写入文件前自动运行 Linting
- 每次编辑后自动运行测试
- 自动将相关文档注入每个提示词
- 任务完成时发送 Slack 通知
- 代码发布前验证安全模式是否被遵循

`UserPromptSubmit` Hook 尤为强大, 我们可以向发送的每条消息注入 `additionalContext`.

**这就是在 Claude Code 之上构建自定义开发环境的方法 -- 不是靠更好的提示词, 而是 Hook 到系统本身**.

## 0x06 Session 是持久化的

**可以随时恢复 (别再从头开始了)**

每个对话都以 JSONL 格式保存在:

`~/.claude/projects/{hash}/{sessionId}.jsonl`

源码支持以下恢复方式:

| 命令 | 作用 |
| ---- | ---- |
| `--continue` | 恢复上一个 Session |
| `--resume` | 选择一个特定的过去 Session |
| `--fork-session` | 从过去的对话分支出去 (我最爱这个) |

Session Memory 提取在压缩过程中保留关键上下文: 任务规格, 文件列表, 工作流状态, 错误和学习总结.

## 0x07 工具系统运行 60+ 工具, 具备只能批处理

工具调用分为 2 类:

- 并发: 文件读取, 搜索等
- 串行: 编辑, 写入, bash 等

可以连接 MCP 服务器来添加更多工具. **延迟加载**机制使工具只在需要时才加载, `ToolSearch` 用于延迟发现 Agent 尚不知道的工具.

## 0x08 流式架构(中断成本极低)

整个处理流水线使用异步生成器 (Async Generator), 逐个事件-yield. 按 ESC 键可以干净得中止当前流, 而不会丢失之前的上下文.


# 实践

## 0x01 原则：大语言模型（大多数）是无状态的

大语言模型（LLM）是无状态函数。它们的权重在用于推理时已被冻结，因此它们不会随时间推移而学习。模型对你代码库的唯一了解就是你输入的 Token。

同样，Claude Code 等编程 AI 智能体 工具套件通常需要你显式管理智能体的记忆。CLAUDE.md（或 AGENTS.md）是默认情况下会进入你与智能体进行的每一次对话的唯一文件。

这有三个重要的含义：

1. 编程 AI 智能体 在每次会话开始时对你的代码库一无所知。
1. 每次开始会话时，必须告知 AI 智能体 关于你代码库的重要信息。
1. CLAUDE.md 是实现这一目标的首选方法。

## 0x02 CLAUDE.md 引导 Claude 熟悉你的代码库

由于 Claude 在每次会话开始时对你的代码库一无所知，你应该使用 CLAUDE.md 来引导 Claude 进入你的代码库。从宏观层面来看，这意味着它应该涵盖：

- **WHAT** ：告诉 Claude 相关技术、你的技术栈、项目结构。给 Claude 一张代码库地图。这对单一代码仓库尤为重要！告诉 Claude 哪些是应用程序，哪些是共享包，以及所有内容的用途，以便它知道去哪里查找东西
- **WHY** ：告诉 Claude 项目的目的以及仓库中各项内容的作用。项目不同部分的目的和功能是什么？
- **HOW** ：告诉 Claude 它应该如何在这个项目上工作。例如，你使用的是 bun 还是 node？需要包含所有能让 Claude 开展实质性工作的信息。如何验证修改？怎样运行测试、类型检查和编译流程？

但实现方式至关重要！不要试图把 Claude 可能需要运行的每条命令都塞进你的 CLAUDE.md 文件中——这样只会适得其反。

## 0x03 渐进式信息呈现

在大型项目中，编写一个涵盖你想让 Claude 知道的所有内容的简洁 CLAUDE.md 文件可能具有挑战性。

为了解决这个问题，我们可以利用渐进式披露原则，确保 Claude 仅在需要时才看到特定于任务或项目的指令。

我们建议将特定于任务的指令保存在项目某处具有自描述名称的单独 Markdown 文件中，而不是将关于构建项目、运行测试、代码约定或其他重要上下文的所有不同指令都包含在 CLAUDE.md 文件中。

例如：

```txt
agent_docs/
  |- building_the_project.md
  |- running_tests.md 
  |- code_conventions.md
  |- service_architecture.md
  |- database_schema.md
  |- service_communication_patterns.md
```

然后，在你的 CLAUDE.md 文件中，你可以包含这些文件的列表以及对每个文件的简要描述，并指示 Claude 决定哪些文件（如果有）是相关的，并在开始工作之前阅读它们。或者，要求 Claude 先向你展示它想阅读的文件以供批准，然后再阅读。

首选指针而非副本。如果可能，不要在这些文件中包含代码片段——它们很快就会过时。相反，包含 file:line 引用以将 Claude 指向权威上下文。

从概念上讲，这与 Claude Skills 的预期工作方式非常相似，尽管技能更侧重于工具使用而非指令。

## 0x04 Claude 是（不是）一个昂贵的代码检查器

我们看到人们放入 CLAUDE.md 文件中最常见的内容之一是代码风格指南。永远不要派 大语言模型 去做 Linter 的工作。与传统的 Linter（代码检查工具）和格式化程序相比，大语言模型 相对昂贵且极其缓慢。我们认为你应该尽可能使用确定性工具。

代码风格指南将不可避免地向你的上下文窗口添加一堆指令和大多不相关的代码片段，从而降低你的大语言模型的性能和指令遵循能力，并消耗你的上下文窗口。

大语言模型是上下文学习者（in-context learners）！如果你的代码遵循一套特定的风格指南或模式，你会发现只要对你的代码库进行几次搜索（或者一份好的研究文档！），你的 AI 智能体 应该倾向于遵循现有的代码模式和约定，而无需被告知。

如果你对此非常执着，你甚至可以考虑设置一个 Claude Code Stop Hook，运行你的格式化程序和 Linter，并向 Claude 展示错误以供其修复。不要让 Claude 自己去发现格式问题。

加分项：使用可以自动修复问题的 Linter（我们喜欢 Biome），并仔细调整关于什么可以安全自动修复的规则，以实现最大（安全）覆盖率。

你还可以创建一个 Slash Command（斜杠命令），其中包含你的代码指南，并将 Claude 指向版本控制中的更改，或你的 git status 或类似内容。这样，你可以分开处理实现和格式化。结果是你会在两方面都看到更好的成效。

## Appendix

[如何编写一份优秀的 CLAUDE.md](https://zhuanlan.zhihu.com/p/1978792365482337485)