# AI 工具教程

> 💡
> 承接技术 IP 的工具教程，重点讲小白怎么用、适合什么开发任务。

# 推荐收录

- Cursor 教程
- Claude Code 教程
- OpenClaw 教程
- Gemini CLI 教程
- AI 编程工具选型对比

---

# 已收录教程

## Claude Code 安装教程：Mac、Windows、Linux 从 0 到跑通

Claude 官方的命令行编程助手。这篇覆盖三个系统从 0 到能跑 `claude --version` 的全流程，含每个平台最容易踩的那个坑。

| 字段 | 内容 |
|-|-|
| 作者 | 小墨同学 |
| 适合谁 | 没用过 AI CLI 的小白、想把 AI 接进自己项目目录的开发者 |
| 解决什么问题 | Claude Code 三系统怎么装、装完 command not found 怎么排查、认证哪些坑 |
| 延伸阅读 | [03.3｜AI 编程案例 → HyperFrames 实战](../03｜AI%20编程与智能体/AI%20编程案例.md) |

**关键收获**

- 通用命令 `npm install -g @anthropic-ai/claude-code` 或官方原生脚本 `curl -fsSL https://claude.ai/install.sh | bash`
- Node.js 18+ 是硬要求；Windows 用户先选好终端（WSL / Git Bash 二选一，别混用）
- 装完先 `claude --version` 验证，再进项目目录跑 `claude` 触发认证
- macOS 装完记得开新 Terminal 窗口，不然原窗口找不到命令

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/KWlzwKCPIix1rbkd1GYc9FEwnLc> ｜ 归档：2026-06-04
