# 智能体工具：超越 Claude Code

Claude Code 是一个工具，所属领域自 2024 年以来已大幅扩展。数十个智能体框架、自主编程工具和多智能体系统相继问世，各有不同的权衡取舍。本页梳理该领域全貌，帮助你判断何时选用 Claude Code，何时其他工具更合适。

**本页涵盖内容**：终端编程智能体、自主编程工具、多智能体编排框架与智能体编排工具。Claude Code 自身的多智能体能力（智能体团队、事件驱动工作流、编程式使用）另有专页记录，行文中有相关链接。

**本页不涵盖内容**：基于图形界面的 AI 编程 IDE（Cursor、Windsurf、Cline），详见 「AI 生态系统 §6」。多 Claude 编排工具（Gas Town、multiclaude、Conductor 桌面应用）详见「第三方工具：多智能体编排」。

---

## 能力谱系

智能体工具处于从交互式到自主式的连续谱系中：

```Plain Text
交互式结对编程
  Claude Code, Codex CLI, Aider, Goose
        |
  Hermes Agent（交互 + 计划调度 + 消息网关）
        |
自主问题修复
  SWE-agent, Devin, claude -p 在 CI 中运行
        |
多智能体框架（自行构建）
  CrewAI, LangGraph, AutoGen/MAF

```

**交互式智能体**：你保持在循环中，审批操作，实时调整智能体。最适合日常编码、调试和需求频繁变化的探索性工作。

**自主智能体**：你指派任务，等待结果返回。最适合描述清晰、边界明确的任务：修复这个 bug、实现这个规格、审查这个 PR。任务描述的质量比工具选择对输出质量的影响更大。

**多智能体框架**：用于构建自定义智能体系统的库，本身不是编程工具。你用 LangGraph 构建智能体，而非用它写代码。

---

## 第一节：终端编程智能体

这些工具做的事和 Claude Code 一样：运行在终端中，读取代码库，编写代码，执行命令。区别在于模型支持、计费模式和具体能力。

---

### 1.1 Codex CLI（OpenAI）

OpenAI 对 Claude Code 的直接回应。2025 年 4 月发布，用 Rust 构建，以 Apache 2.0 协议开源。

| 属性 | 详情 |
|-|-|
| **GitHub** | [openai/codex](https://github.com/openai/codex) |
| **Stars** | 86,200+（2026 年 5 月） |
| **安装** | `npm install -g @openai/codex` |
| **语言** | Rust（96%） |
| **许可证** | Apache 2.0 |
| **版本** | v0.134.0（2026 年 5 月 26 日） |
| **发布次数** | 2025 年 4 月以来 800+ 次 |
| **贡献者** | 400+ |

#### Codex CLI 是什么？

一款基于 OpenAI 模型家族的终端 AI 智能体，用于编写、编辑和运行代码。其架构与 Claude Code 高度相似：描述任务，智能体读取文件、进行修改、运行测试并迭代。主要区别在于模型提供商：Codex CLI 使用 GPT-4o、o3、o4-mini 等 OpenAI 模型，而非 Claude。

ChatGPT Pro 和 Team 订阅用户可在其计划内使用 Codex CLI，对于已购买 OpenAI 订阅的团队而言是零边际成本工具。

#### Claude Code 与 Codex CLI 对比

| 方面 | Claude Code | Codex CLI |
|-|-|-|
| **模型** | 仅限 Claude 3.5/4 系列 | GPT-4o、o3、o3-mini、o4-mini 及未来 OpenAI 模型 |
| **语言** | TypeScript | Rust |
| **许可证** | 开源 | Apache 2.0 |
| **订阅** | Anthropic Claude Max（20-200 美元/月） | OpenAI ChatGPT Pro/Team（20-30 美元/月） |
| **MCP 支持** | 原生，生态系统持续扩展 | 兼容 MCP |
| **发布节奏** | 每周 | 非常频繁（13 个月内 800+ 次发布） |
| **记忆** | CLAUDE.md + 自动记忆 | AGENTS.md 惯例 |
| **Skills/Hooks** | 完整系统 | 兼容 agentskills.io 标准 |

#### 何时选择 Codex CLI

适合已订阅 ChatGPT Pro 或 Team 且希望避免第二份订阅的用户。如果你在特定任务（推理、长上下文分析）上偏好 GPT-4o 或 o3，并希望使用原生支持这些模型的终端智能体，也是合适的选择。

不适合已经在 Claude Code 工作流、CLAUDE.md 文件和 Anthropic 特定模式上有大量投入的团队。在两个智能体环境之间切换的认知成本不容忽视。

#### 快速开始

```Bash
npm install -g @openai/codex
export OPENAI_API_KEY=sk-...
codex

```

OpenAI 的 [Codex 文档](https://github.com/openai/codex/blob/main/README.md) 有详细安装说明。

---

### 1.2 Hermes Agent（前身为 OpenClaw）

截至 2026 年 5 月，GitHub 上 stars 最多的开源智能体框架。由 Nous Research 创建，该 AI 实验室以其 Hermes 系列微调模型著称。此前名为 OpenClaw，2025 年末在 Anthropic 恢复订阅支持后更名。

| 属性 | 详情 |
|-|-|
| **GitHub** | [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) |
| **Stars** | 170,000+（2026 年 5 月） |
| **安装** | `pip install hermes-agent` 或 `curl -sSL install.hermes-agent.dev | sh` |
| **语言** | Python（89%）、TypeScript（8%） |
| **许可证** | MIT |
| **版本** | v0.14.0（2026 年 5 月 16 日） |
| **发布节奏** | 每周（v0.10 4 月 16 日 → v0.14 5 月 16 日） |
| **贡献者** | 215+ |
| **创建者** | Nous Research（Teknium，@teknium1） |

#### Hermes Agent 是什么？

一款自我改进的终端智能体，支持 200+ 个 LLM 提供商，可在任何平台运行，并接入 22 个消息平台（Telegram、Discord、Slack、WhatsApp、Signal、Teams、LINE、SimpleX 等）。其核心特性是学习循环：完成任务后，Hermes 分析有效方法，提取可复用模式，并自动生成 Skills（技能模块）。每次会话都使智能体在特定工作流上略微提升。

OpenClaw 历史有两方面值得关注。其一，迁移路径清晰：`hermes-agent` 在设置时导入 OpenClaw 的记忆、Skills（技能模块）和配置，切换成本较低。其二，2026 年初的 Anthropic 计费争议专门针对 OpenClaw/Hermes 在 Claude Max 订阅上使用时未正确归因编程计费。Anthropic 现已明确将 Hermes 纳入编程计费范畴（参见「计费：编程式 vs 交互式」）。

#### Claude Code 与 Hermes Agent 对比

| 方面 | Claude Code | Hermes Agent |
|-|-|-|
| **模型** | 仅限 Claude | 200+ 提供商（OpenRouter、OpenAI、Anthropic、HuggingFace、本地） |
| **自我改进** | 每次会话全新开始 | 从重复模式自动生成 Skills（技能模块） |
| **消息接入** | 终端 + IDE | 终端 + 22 个聊天平台 |
| **定时计划** | Routines（Anthropic 云端） | 内置 cron，本地运行 |
| **计费** | 订阅或 API | 直接向 LLM 提供商付费 |
| **智能体 SDK** | Anthropic 专属 | `ctx.llm` 插件支持任意提供商 |
| **Skills（技能模块）** | SKILL.md 系统 | Skills Hub（agentskills.io）+ 自动生成 |
| **记忆** | CLAUDE.md + 自动记忆 | 跨会话持久记忆，由智能体维护 |

#### 何时选择 Hermes Agent

模型无关性是最有力的论据。如果你希望在代码生成时使用 Claude、在特定推理任务时使用 GPT-4o、在离线工作时使用本地模型（通过 Ollama），Hermes 可在单一智能体中处理所有情况，Claude Code 做不到这一点。

自我改进循环是真正的差异化特性。在同一代码库上使用 30-40 次会话后，Hermes 会积累针对项目模式的 Skills（技能模块）库。这种复利效应是静态 CLAUDE.md 文件无法达到的，尽管比较并不简单——CLAUDE.md 是人工编写且有意为之的，而 Hermes Skills（技能模块）是机器生成的。

22 个消息平台集成对于希望通过 Telegram 或 Slack 而非终端与智能体交互的团队很实用，对大多数开发者而言并非优先需求，但对某些工作流至关重要。

不适合深度投入 Anthropic 生态系统（Claude Max 订阅、Routines、Agent SDK）的用户。使用 Claude 模型运行 Hermes 会消耗编程计费额度，意味着 200 美元/月 Max 订阅的额度同时被交互式终端使用和 Hermes API 调用消耗。请综合考量。

#### 快速开始

```Bash
pip install hermes-agent

# 或一键安装
curl -sSL install.hermes-agent.dev | sh

# 从 OpenClaw 迁移
hermes import --from openclaw

# 启动
hermes

```

---

### 1.3 Aider

原版终端 AI 结对编程工具。由 Paul Gauthier 在 Claude Code 出现之前的 2023 年发布，Aider 确立了许多后续工具沿用的惯例：直接文件编辑、自动 git 提交、多文件上下文窗口。

| 属性 | 详情 |
|-|-|
| **GitHub** | [Aider-AI/aider](https://github.com/Aider-AI/aider) |
| **Stars** | 45,400+（2026 年 5 月） |
| **安装** | `pip install aider-install && aider-install` |
| **语言** | Python（80%） |
| **许可证** | Apache 2.0 |
| **创建者** | Paul Gauthier（paul-gauthier） |
| **PyPI 下载量** | 530 万+ |

#### Aider 是什么？

一款基于 Python 的编程助手，可编辑本地 git 仓库中的文件并自动提交附带描述性信息的变更。核心特点：通过 LiteLLM 支持近乎通用的模型，涵盖 GPT-4o、Claude 3.5/4、Gemini、Ollama 及数十个其他提供商。Aider 推广了"整文件"和"差异对比（diff）"编辑格式，影响了后续智能体处理文件修改的方式。

SWE-Bench 基准测试的成绩轨迹很能说明问题：Aider 在 2024-2025 年期间数月内位居 SWE-Bench Verified 榜首，后被支持更大上下文的模型和更强大的智能体超越。这一基准记录确立了其作为严肃工具而非便利封装的声誉。

#### Claude Code 与 Aider 对比

| 方面 | Claude Code | Aider |
|-|-|-|
| **模型支持** | 仅限 Claude | GPT-4o、Claude、Gemini、Ollama、50+ 提供商 |
| **Git 集成** | 原生（读取 .git，运行 git） | 深度集成（自动提交、提交信息、blame 上下文） |
| **架构** | Anthropic 专有 | 开源，底层使用 LiteLLM |
| **文件编辑** | 基于工具（编辑工具、写入工具） | 整文件或差异对比（diff）格式发送给模型 |
| **网页搜索** | 通过 MCP | 非原生（需要插件） |
| **智能体循环** | 完整（多轮，工具使用） | 完整（architect 模式下自动接受变更） |
| **发布节奏** | 每周 | 每月（最近：v0.86.0，2025 年 8 月） |

最后发布日期（2025 年 8 月）值得注意。Aider 仍在维护且功能正常，但相对于 Claude Code 和 Hermes，发布节奏有所放缓。这本身不是警示信号，但如果你需要最新特性，值得留意。

#### 何时选择 Aider

最佳场景：你需要在成熟、经过实战检验的工具中支持多模型，且不想承担 Hermes 的运维开销。Aider 比 Hermes 配置更简单，占用资源更少，有数年积累的社区文档。

也适合对 git 规范有严格要求、希望每次 AI 变更都明确提交并附清晰信息的团队。Aider 的自动提交行为比 Claude Code 更激进（Claude Code 默认在提交前询问）。

#### 快速开始

```Bash
pip install aider-install && aider-install

# 使用 Claude
export ANTHROPIC_API_KEY=sk-ant-...
aider --model claude-sonnet-4-6

# 使用 GPT-4o
export OPENAI_API_KEY=sk-...
aider

```

完整模型列表和配置选项参见 [aider.chat](https://aider.chat)。

---

### 1.4 Goose（AAIF/Block）

通用型智能体，不仅限于编程工具。最初由 Block（前身为 Square）构建，后转移至 Linux 基金会旗下的 AAIF（智能体 AI 基础设施基金会）以实现长期治理中立。

**完整介绍见 「AI 生态系统 §11.1：Goose」。**

快速数据：45,900+ stars（2026 年 5 月），Rust（63%）+ TypeScript（30%），Apache 2.0，每日活跃开发，368+ 贡献者。与 Claude Code 的核心区别：提供商无关（Claude、GPT、Gemini、Ollama、15+ 提供商），基于 recipe 的可复用工作流，以及异构子智能体团队（每个子智能体可运行不同模型）。

---

## 第二节：自主编程智能体

这些工具在你不旁观的情况下运行。你给出任务描述（GitHub issue、规格说明、bug 报告），它们产出一个 pull request。交互模式与终端智能体根本不同：迭代性更少，更像向同事分配工作。

---

### 2.1 Devin（Cognition）

第一款商业化完全自主软件工程师。闭源、云端托管、企业定价。

| 属性 | 详情 |
|-|-|
| **网站** | [devin.ai](https://devin.ai) |
| **类型** | 云端 SaaS，专有 |
| **定价** | Core：20 美元/月（按 ACU 付费）；Team：500 美元/月（250 ACU）；Enterprise：定制 |
| **发布时间** | 2024 年 |
| **估值** | 250 亿美元（2026 年 4 月融资） |
| **重要收购** | Windsurf AI 原生 IDE（2025 年 7 月） |
| **企业客户** | 高盛、微软、Palantir、花旗、戴尔 |

#### Devin 是什么？

一款在云端 Linux 虚拟机中运行、配备专属 shell、代码编辑器和浏览器的自主软件工程师。Devin 规划方案、编写代码、运行测试、读取错误信息，并迭代直至任务完成或遇到阻碍。主要交互界面是 Slack：你发送消息"修复 issue #342"，Devin 完成后开 PR。

计费单位为 ACU（智能体计算单元），1 ACU 约对应 15 分钟智能体工作时长。复杂功能可能消耗 10-20 ACU，简单 bug 修复可能用 1-3 ACU。

#### Claude Code 与 Devin 对比

| 方面 | Claude Code | Devin |
|-|-|-|
| **执行环境** | 你的本地机器 | 云端 Linux 虚拟机（沙盒） |
| **交互模式** | 交互式（你在旁观察） | 异步（分配后等待结果） |
| **状态** | 会话范围内 | 任务全程持久化 |
| **定价** | 订阅制（20-200 美元/月） | 按任务 ACU 计费（约 0.07-0.15 美元/ACU） |
| **谁来驱动** | 你（结对编程） | 智能体（自主运行，你审查） |
| **任务描述** | 对话式，迭代式 | 前置（规格越清晰，输出越好） |
| **浏览器访问** | 通过 MCP（Playwright） | 内置，原生支持 |
| **代码审查集成** | 你在 IDE 中审查 | Devin 开 PR，你在 GitHub 上审查 |

#### 何时选择 Devin

Devin 最适合任务描述清晰、边界明确、无需持续判断的场景。重构特定模块、实现已有文档的 API 端点、修复已知根本原因的回归：这些是 Devin 的任务。设计新系统架构、调试晦涩的生产问题、编写依赖代码库隐性上下文的代码：这些需要更多交互式循环。

500 美元/月的 Team 计划（250 ACU）成本不低。在这个价位上，你付费购买的是异步价值：开发者不必等待智能体输出而被阻塞、智能体并行处理多个任务、无需上下文切换。如果你的瓶颈是开发者注意力而非原始吞吐量，Devin 值得计算投入产出比。如果你想保持在循环中并交互式迭代，200 美元/月的 Claude Code 提供更高的性价比。

Windsurf 收购（2025 年 7 月）表明 Cognition 正向完整开发者环境迈进，而不仅仅是后台智能体。关注将交互式编程（Windsurf IDE）与自主任务执行（Devin）融合在同一产品中的工作流。

---

### 2.2 SWE-agent（普林斯顿）

专为从 issue 描述单独解决 GitHub issues 而设计的学术型智能体。NeurIPS 2024 论文，由普林斯顿 NLP 组和斯坦福合作发布。

| 属性 | 详情 |
|-|-|
| **GitHub** | [SWE-agent/SWE-agent](https://github.com/SWE-agent/SWE-agent) |
| **Stars** | 19,300+（2026 年 5 月） |
| **论文** | NeurIPS 2024 |
| **许可证** | MIT |
| **语言** | Python（95%） |
| **版本** | v1.1.0（2025 年 5 月） |
| **维护者** | 普林斯顿 NLP 组 + 斯坦福 |

#### SWE-agent 是什么？

一条智能体流水线：接收 GitHub issue URL 和模型，尝试复现 bug、编写修复方案并生成补丁。其架构使用智能体-计算机接口（ACI）层，将终端、文件编辑和测试运行抽象为一组一致的命令，无论底层环境如何。这一 ACI 设计是主要学术贡献：它表明智能体性能与环境暴露信息的质量高度相关，而非仅与模型的原始能力相关。

SWE-agent + Claude 3.7 在 SWE-Bench Full（开放权重）上保持最先进水平。这一基准是关键背景：SWE-Bench 衡量智能体端到端解决真实 GitHub issues 的百分比，SWE-agent 正是以该基准为优化目标而设计的。

#### 何时选择 SWE-agent

主要用于学术和研究场景。如果你想对不同模型在真实 GitHub issues 上的表现进行系统评估，SWE-agent 是正确工具，因为它具备生产工具所欠缺的可复现性基础设施（轨迹记录、评估框架、配置 YAML）。

对于生产级批量 issue 解决，Devin 的云端沙盒和更好的错误恢复使其更具实用性。SWE-agent 需要你自行设置环境并手动处理失败。

研究价值是真实的：构建智能体系统的团队可使用 SWE-agent 的轨迹数据（由 issue 解决运行生成）来微调模型。Nous Research 的 SWE-agent-LM-32b（开放权重，SWE-Bench 开放模型最先进水平）就是在 SWE-agent 生成的轨迹上训练的。

```Bash
pip install swe-agent

# 在 GitHub issue 上运行
sweagent run \
  --agent.model.name=claude-sonnet-4-6 \
  --env.repo.github_url=https://github.com/org/repo \
  --problem_statement.github_url=https://github.com/org/repo/issues/123

```

---

### 2.3 无头模式下的 Claude Code

Claude Code 自身的自主模式：`claude -p "任务"` 以非交互方式运行单条指令后退出。结合 CI/CD，它成为一个自主智能体，可由 GitHub 事件触发、通过 Routines 按计划运行，或通过 Agent SDK 以编程方式处理任务。

**自 2026 年 6 月 15 日起，这属于编程计费范畴。** 参见「计费：编程式 vs 交互式」了解额度限制和超额费率。

常用模式：

```Bash
# 单任务，完成后退出
claude -p "为 src/auth.ts 编写测试，目标覆盖率 80%"

# GitHub Actions：由 issue 标签触发
# 完整模式见 workflows/event-driven-agents.md

# Agent SDK：带工具的编程式使用
# 见 ai-ecosystem.md §14（Claude 托管智能体）

```

交叉引用：

- **事件驱动模式**：「workflows/event-driven-agents.md」
- **智能体团队**：「workflows/agent-teams.md」
- **托管智能体（云端）**：「ai-ecosystem.md §14」

---

## 第三节：多智能体框架

这些不是编程工具，而是从零开始构建自定义多智能体应用的库：营销流水线、研究自动化、文档处理、客服机器人。如果你正在构建内部含有 AI 智能体的产品，而非作为开发者希望智能体为你写代码，才应使用它们。

与 Claude Code 的关系：Claude（模型）可以是这些框架所构建智能体的 LLM 之一。这些框架本身并不与 Claude Code 竞争，就像 Express.js 不与浏览器竞争一样。

---

### 3.1 CrewAI

基于角色的多智能体编排工具。对于希望按职能（研究员、撰稿人、编辑）定义智能体并让其在结构化任务上协作的团队，是主流选择。

| 属性 | 详情 |
|-|-|
| **GitHub** | [crewAIInc/crewAI](https://github.com/crewAIInc/crewAI) |
| **Stars** | 52,300+（2026 年 5 月） |
| **语言** | Python（99%） |
| **许可证** | MIT |
| **版本** | v1.14.5（2026 年 5 月 18 日） |
| **执行次数** | 报告显示超过 20 亿次智能体任务执行 |
| **下载量** | 2700 万+ |
| **企业客户** | 150+ |

#### CrewAI 是什么？

你定义具有角色、目标和背景故事的智能体（"团队"），定义任务并分配给智能体。CrewAI 处理路由：顺序（A 完成后 B 开始）、并行（A 和 B 同时运行）或层级（管理者智能体委派给专家）。每个智能体可使用工具，包括 MCP 服务器和网页搜索，支持多个 LLM 提供商（Claude、GPT、Gemini、Ollama）。

它区别于 LangChain（通常被拿来对比的老框架）之处在于：完全不依赖 LangChain。是独立的 Python 库。

#### 何时使用 CrewAI

适合能够以人类角色描述工作流的团队。如果你能说出"我希望有一个研究员收集信息、一个撰稿人起草、一个编辑润色"，CrewAI 负责编排和智能体间的通信。你编写智能体定义，而非编排代码。

当工作流有复杂条件分支、需要跨故障持久执行，或需要精细控制状态在智能体间传递时，避免使用 CrewAI。LangGraph 更适合这些场景。

```Python
from crewai import Agent, Task, Crew, Process

researcher = Agent(
    role="Technical Researcher",
    goal="Find accurate technical information",
    backstory="Expert at synthesizing documentation and research papers",
    llm="claude-sonnet-4-6"
)

writer = Agent(
    role="Technical Writer",
    goal="Write clear, accurate documentation",
    backstory="Experienced at translating technical concepts",
    llm="claude-sonnet-4-6"
)

task = Task(
    description="Research and document the new auth API endpoints",
    expected_output="Markdown documentation with examples",
    agent=writer,
    context=[research_task]  # researcher 的输出喂给 writer
)

crew = Crew(agents=[researcher, writer], tasks=[task], process=Process.sequential)
result = crew.kickoff()

```

---

### 3.2 LangGraph

LangChain 出品的基于图的智能体编排工具。比 CrewAI 更底层，更灵活，更适合复杂的有状态工作流。

| 属性 | 详情 |
|-|-|
| **GitHub** | [langchain-ai/langgraph](https://github.com/langchain-ai/langgraph) |
| **Stars** | 33,100+（2026 年 5 月） |
| **语言** | Python（99%）+ JS 版本可用 |
| **许可证** | MIT |
| **版本** | v1.2.2（2026 年 5 月 26 日） |
| **生产用户** | Klarna、Replit、Elastic |

#### LangGraph 是什么？

一个将工作流建模为有向图的智能体构建框架，图中有节点（智能体步骤）和边（转换）。核心原语：状态（跨所有节点持久化的类型化字典）、条件边（基于状态的分支）和持久化（检查点机制使中断的工作流从最后一个检查点恢复，而非从头开始）。人机协作是一等模式：可在任意节点暂停执行，等待人类决策后再继续。

LangGraph 不自带智能体，你定义工作流逻辑并插入任意 LLM。框架确保状态转换可预测、故障可恢复，工作流可逐步调试。

#### 何时使用 LangGraph

适合智能体需要在故障中存活、基于运行时条件分支，或在特定决策点需要人工审批的场景。示例：当智能体检测到安全相关变更时上报给人类的代码审查流水线；在每个耗时步骤后设置检查点的数据处理工作流（重启时不重新处理已完成阶段）；遇到模糊信源时暂停等待人工指导的多步骤研究智能体。

学习曲线比 CrewAI 陡。工作流复杂度足够高时才值得投入。

```Python
from langgraph.graph import StateGraph, END
from typing import TypedDict

class AgentState(TypedDict):
    messages: list
    task_complete: bool

graph = StateGraph(AgentState)
graph.add_node("agent", call_agent)
graph.add_node("tools", call_tools)
graph.add_conditional_edges("agent", should_continue, {"continue": "tools", "end": END})
graph.add_edge("tools", "agent")
graph.set_entry_point("agent")

app = graph.compile(checkpointer=MemorySaver())  # 持久执行

```

LangSmith（LangChain 的可观测性产品）原生集成，用于调试和追踪智能体运行。

---

### 3.3 AutoGen / Microsoft Agent Framework

微软的多智能体框架，正处于从原始 AutoGen 库（2025 年 9 月起进入维护模式）向 Microsoft Agent Framework（MAF）的过渡期，MAF 将 AutoGen 和 Semantic Kernel 合并为一个 SDK。

| 属性 | 详情（MAF） |
|-|-|
| **GitHub** | [microsoft/agent-framework](https://github.com/microsoft/agent-framework) |
| **Stars** | 10,800+（MAF，活跃） |
| **旧版 GitHub** | [microsoft/autogen](https://github.com/microsoft/autogen)（58,400 stars，2025 年 9 月起维护模式） |
| **语言** | Python + C# + TypeScript |
| **许可证** | MIT |
| **版本** | python-1.6.0（2026 年 5 月 22 日） |
| **生产版本** | v1.0（2026 年 4 月） |

#### Microsoft Agent Framework 是什么？

MAF 是 AutoGen（Python，对话式多智能体）和 Semantic Kernel（C# + Python，函数调用抽象）的合并。结果是一个跨运行时框架：Python 智能体可与 .NET 智能体协调，全部由同一消息层支撑。它实现了 A2A（智能体间）协议（微软对智能体互操作性的贡献），并支持 MCP。

AutoGen 的 star 数量（58,400）反映了其历史声誉。AutoGen 开创了"可对话智能体"模式——智能体在结构化对话循环中相互交谈。即便实现随时间演进，这一模式仍是框架中占主导地位的思维模型。

#### 何时使用 MAF

非常适合微软生态系统团队：.NET + Python 混合开发、Azure 部署、已有 Semantic Kernel 的企业环境。跨运行时特性是真实的：Python 智能体可调用以 .NET Semantic Kernel 函数实现的工具。

对于没有现有 .NET 投入的团队吸引力较弱。如果你是纯 Python 栈，CrewAI 或 LangGraph 有更大的社区和更多教程。

---

### 3.4 Anthropic Agent SDK

Anthropic 自家的以编程方式构建多智能体系统的框架，有别于 Claude Code。详见 「AI 生态系统 §14：Claude 托管智能体」。

操作性区别：Claude Code 是你作为开发者使用的成品；Agent SDK 是你用来构建内部含有 Claude 的产品的库。Agent SDK 通过 Messages API 处理工具使用、上下文管理和多智能体协调。它同样属于编程计费范畴（见上方计费交叉引用）。

---

## 第四节：智能体编排工具

位于智能体框架之上的工具，管理智能体在规模化场景下的部署、路由和运维。不要与多 Claude 编排工具（Gas Town、multiclaude）混淆，后者在「第三方工具」中介绍。

---

### 4.1 Conductor（Gemini CLI 方法论）

一种开发方法论，而非产品。"Conductor"最初是 Gemini CLI 的扩展，强制执行"上下文 → 规格 → 计划 → 实现"工作流：在编写任何代码之前，智能体先创建并提交规格文档，然后是计划文档，再按两者进行实现。

| 属性 | 详情 |
|-|-|
| **GitHub** | [gemini-cli-extensions/conductor](https://github.com/gemini-cli-extensions/conductor) |
| **Stars** | 3,600+（2026 年 5 月） |
| **许可证** | Apache 2.0 |

该方法论已通过社区仓库移植到 Claude Code：[lackeyjb/claude-conductor](https://github.com/lackeyjb/claude-conductor)、[ryanmac/code-conductor](https://github.com/ryanmac/code-conductor) 和 wshobson/agents 插件市场。这些仓库自身没有显著吸引力，但模式本身（先写规格再写代码，提交文档）直接对应 Claude Code 的「规格优先开发工作流」。

---

### 4.2 Conductor（Microsoft CLI）

与 Gemini 版本完全独立的项目。一款以 YAML 为核心的 CLI，用于确定性多智能体工作流，路由逻辑是静态配置，而非 LLM 决策。

| 属性 | 详情 |
|-|-|
| **GitHub** | [microsoft/conductor](https://github.com/microsoft/conductor) |
| **Stars** | 158（2026 年 5 月，全新项目） |
| **许可证** | MIT |
| **发布时间** | 2026 年 5 月 14 日（微软开源博客） |

核心理念：在 YAML 中定义智能体工作流（哪些智能体顺序运行、哪些并行、每个使用什么模型、各阶段之间传递什么），以确定性方式执行。编排循环中无 LLM，只有智能体步骤中有。支持 GitHub Copilot SDK 和 Anthropic Agent SDK 作为提供商。仍处于极早期阶段（158 stars，撰写时刚发布数天），但由微软开源团队支持。

---

### 4.3 Hermes Control Room

社区模板，作者为 Shann（@shannhk，里斯本），用于在 VPS 上管理 Hermes 智能体集群。非 Nous Research 项目。

| 属性 | 详情 |
|-|-|
| **GitHub** | [shannhk/hermes-agent-control-room](https://github.com/shannhk/hermes-agent-control-room) |
| **Stars** | 474（2026 年 5 月） |
| **时间** | 12 天（截至 2026 年 5 月 27 日） |
| **类型** | 模板/文档，非可执行软件 |

概念：包含治理文档、已部署智能体注册表、常见操作 runbook 和 8 个内置 Hermes Skills（技能模块）的文件夹结构，Skills（技能模块）涵盖 VPS 配置、任务路由、备份、安全审计和 cron 规划。智能体共享基于文件系统的任务总线（每个专业领域有 inbox/working/outbox/archive）。编排者读取 control room 文档了解智能体能力，通过总线路由任务，并综合结果。

对于运行 3 个以上 Hermes 智能体的任何人，这一模式都是合理的。但该仓库太新（7 次提交），不建议作为生产依赖。关注 v1.0 版本的运维强化。

---

## 第五节：决策框架

### 完整对比矩阵

| 工具 | 开源 | Stars | 模型支持 | 模式 | 语言 | 费用 |
|-|-|-|-|-|-|-|
| **Claude Code** | 是（TS） | 112K | 仅限 Claude | 交互式 + 无头模式 | TypeScript | 20-200 美元/月 |
| **Codex CLI** | 是 | 86K | GPT-4o、o3、o4-mini | 交互式 + 无头模式 | Rust | 含在 ChatGPT Pro/Team 中 |
| **Hermes Agent** | 是（MIT） | 170K | 200+ 提供商 | 交互式 + cron + 消息 | Python | 按 LLM 调用付费 |
| **Aider** | 是 | 45K | 50+ 提供商 | 交互式 | Python | 按 LLM 调用付费 |
| **Goose** | 是 | 46K | 15+ 提供商 | 交互式 + 子智能体 | Rust | 按 LLM 调用付费 |
| **Devin** | 否 | N/A | 专有 | 完全自主 | 专有 | 20-500 美元/月 |
| **SWE-agent** | 是（MIT） | 19K | 任意（Claude、GPT...） | 自主（issue → PR） | Python | 按 LLM 调用付费 |
| **CrewAI** | 是（MIT） | 52K | 50+ 提供商 | 框架（自行构建） | Python | 框架免费 |
| **LangGraph** | 是（MIT） | 33K | 任意 | 框架 | Python/JS | 框架免费 |
| **AutoGen/MAF** | 是（MIT） | 58K/11K | 任意 | 框架 | Python/C#/TS | 框架免费 |

### 情境工具选择指南

| 情境 | 推荐 |
|-|-|
| 日常编码，已订阅 Claude Max | Claude Code |
| 日常编码，已订阅 ChatGPT Pro | Codex CLI |
| 日常编码，希望用任意模型 | Hermes Agent 或 Aider |
| 日常编码，通用型智能体 | Goose |
| 分配任务，等待 PR 结果 | Devin（500 美元/月）或 CI 中的 `claude -p` |
| 自主修复 GitHub issues，研究/基准测试 | SWE-agent |
| 编排多个 Claude Code 实例 | Gas Town、multiclaude、Ruflo（见「第三方工具」） |
| 构建基于角色的多智能体产品 | CrewAI |
| 构建有状态、可恢复的工作流 | LangGraph |
| 在 .NET + Python 的微软技术栈中构建 | AutoGen/MAF |
| Anthropic 生态系统，云端托管智能体 | Anthropic Agent SDK（见 「ai-ecosystem.md §14」） |
| 在 VPS 上管理 Hermes 智能体集群 | Hermes Control Room 模式 |

### 模型锁定问题

在 Claude Code、Codex CLI、Hermes、Aider 和 Goose 之间做选择时，最核心的澄清问题是：工具是否需要与唯一一个模型提供商绑定，还是支持多个？

如果你致力于 Claude 和 Anthropic 生态系统（订阅、Routines、Agent SDK、CLAUDE.md 工具链），Claude Code 毫无疑问是正确选择。集成原生且来自 Anthropic 的功能迭代速度很快。

如果你需要模型灵活性（本地模型处理敏感代码、更便宜的模型处理常规任务、特定模型用于基准测试），Hermes Agent 支持最广泛的范围且自动化程度最高。Aider 和 Goose 是体量更小的简单替代方案。

如果你的团队以 OpenAI 为主且已为 ChatGPT Pro 付费，Codex CLI 无需额外增量成本。

### 自主性与控制的权衡

更高的自主性意味着智能体可以在你不旁观的情况下完成更多工作，但也意味着在模糊任务上更容易偏离方向。合适的自主性级别取决于任务的描述清晰程度，而非工具的"能力强弱"。

Claude Code 无头模式（`claude -p`）和 SWE-agent 给你受控的自主性：你设置任务，智能体运行，你审查输出。Devin 给你最大化的自主性，配备云端沙盒：智能体拥有完整的 Linux 环境，可以采取你未预料到的行动。权力越大，合并前所需的审查越多。

交互式智能体（Claude Code 终端、Hermes、Aider、Goose）给你实时控制权。你旁观智能体思考，在它走偏时及时纠正，并批准破坏性操作。对于需求在会话中途变化的探索性工作，交互式方式尽管看起来更费力，实际上往往比自主方式更快。

---

## 交叉引用

- **多 Claude 编排**（Gas Town、multiclaude、Ruflo、Conductor 桌面应用）：「第三方工具：多智能体编排」
- **Goose 深度解析**：「AI 生态系统 §11.1」
- **使用 Anthropic SDK 构建自定义智能体**：「AI 生态系统 §14」
- **Claude Code 自身的智能体团队模式**：「workflows/agent-teams.md」
- **事件驱动自主模式**：「workflows/event-driven-agents.md」
- **编程计费（Hermes、Codex CLI、第三方框架）**：《Claude Code：从入门到大师》：计费分类
- **智能体框架工程（理论框架）**：「core/agent-harness.md」
- **编程智能体对比矩阵**（23 款工具，11 项标准）：[coding-agents-matrix.dev](https://coding-agents-matrix.dev)

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/Slh5wuMijitdlmkr9tEcOEkOnC2> ｜ 归档：2026-06-04
