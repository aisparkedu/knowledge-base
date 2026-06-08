# reMarkable 2 AI 黑客指南

> **最后验证**：2026 年 2 月

reMarkable 2 是一款具备完整 root 访问权限的 Linux 电子墨水平板。其零干扰的设计理念使其成为出色的思考工具，但原生集成极为简约。本页涵盖所有利用 AI 增强它的方法——从最简单到最技术性。

## 目录

1. remarkable-mcp：游戏规则改变者
2. Ghostwriter：Vision-LLM 接口
3. reMarkable → Obsidian 同步
4. OCR + AI 自定义流水线
5. SSH 访问与社区工具
6. 未充分利用的原生功能
7. 官方 API 与开发者门户
8. Zapier 自动化
9. Read-it-later：网页 → reMarkable
10. 会议记录 → AI 摘要
11. Zotero → reMarkable（研究用途）
12. 屏幕共享作为 AI 辅助白板
13. 自定义应用与趣味黑客
14. 待构建的 AI 增强工作流
15. 从哪里开始

---

## 1. remarkable-mcp：游戏规则改变者

**ROI：最大化 | 难度：中等 | 连接方式：USB SSH（无需云端）**

Sam Morrow 创建了一个 **MCP 服务器**，将 reMarkable 直接连接到 Claude Code、VS Code Copilot 以及所有兼容 MCP 的 AI 助手。

| 属性 | 详情 |
|-|-|
| **仓库** | https://github.com/SamMorrowDrums/remarkable-mcp |
| **博客** | https://sam-morrow.com/blog/building-an-mcp-server-for-remarkable |
| **连接方式** | USB SSH——无需云端，无需订阅 |
| **语言** | Python（FastMCP） |

### 功能

- **原生提取输入文字**（Type Folio / 虚拟键盘）——即时，无需 OCR
- **手写 OCR**——通过 Google Cloud Vision（每月 1000 次免费请求）
- **智能搜索**——在你的整个文库中搜索
- **文本提取**——从 PDF 和 EPUB 及其注释中提取
- **完整文档遍历**

### 为什么排名第一

你可以问 Claude"我在 1 月 15 日会议上关于 X 记了什么？"——它会在你的手写笔记中搜索。reMarkable 成为一个**可查询的第二大脑**。

### 技术栈

```Plain Text
FastMCP + rmscene（原生 .rm 解析）+ PyMuPDF（PDF）
+ Google Cloud Vision（OCR）+ Paramiko（SSH）

```

### 快速安装

```Bash
# 1. 在 reMarkable 上启用 SSH
# 设置 → 帮助 → 版权与许可证 → IP + root 密码

# 2. 克隆仓库
git clone https://github.com/SamMorrowDrums/remarkable-mcp
cd remarkable-mcp && pip install -e .

# 3. 添加到 Claude Code
# 在 ~/.claude.json 中或通过 "claude mcp add"

```

### Claude Code 配置

```JSON
{
  "mcpServers": {
    "remarkable": {
      "command": "python",
      "args": ["-m", "remarkable_mcp"],
      "env": {
        "REMARKABLE_HOST": "10.11.99.1",
        "REMARKABLE_PASSWORD": "<root-password>"
      }
    }
  }
}

```

---

## 2. Ghostwriter：Vision-LLM 接口

**ROI：实验性 | 难度：低（复制一个 Rust 二进制文件）**

| 属性 | 详情 |
|-|-|
| **仓库** | https://github.com/awwaiid/ghostwriter |
| **模型** | GPT-4o Vision |
| **HN 讨论** | https://news.ycombinator.com/item?id=42979986 |

### 概念

你在 reMarkable 上手写一个提示词。Vision-LLM（GPT-4o）读取你的笔迹和图画，**直接在平板上**回复。

### 安装

```Bash
# 1. 下载已编译的 Rust 二进制文件
scp ghostwriter root@10.11.99.1:/home/root/

# 2. SSH + 启动
ssh root@10.11.99.1
chmod +x ghostwriter && ./ghostwriter

```

### 支持的交互

- 手写识别
- 草图分析（线框图、原理图）
- 小型图标语言
- 手势

### 实际用例

画一个架构图，写上"优化这个"——LLM 视觉分析后回复。这是一个引人入胜的原型，探索通过手写笔进行人机 AI 交互。

**客观局限性**：reMarkable 的原生绘图应用极为简约（回复中无法自由放置文本）。

---

## 3. reMarkable → Obsidian 同步

**ROI：使用 Obsidian 时很高 | 难度：低至中等**

### 选项 A：Scrybble（最完整）

| 属性 | 详情 |
|-|-|
| **网站** | scrybble.ink |
| **Obsidian 插件** | 社区插件（vault 设置） |
| **托管** | 自托管或 Scrybble 服务器 |
| **讨论** | https://forum.obsidian.md/t/scrybble-sync-plugin/103194 |

**功能：**

- 将笔记本、PDF、ePub 同步到 Obsidian vault
- 将 PDF/ePub 高亮提取为 Markdown
- 将输入文字提取为 Markdown
- 在 vault 中将完整笔记本渲染为 PDF
- 带标签的按页组织

**用例**：学术研究、会议记录，背后有 Obsidian 搜索支持。

### 选项 B：自定义云同步插件

- **演示**：https://www.youtube.com/watch?v=EsRdi8J9Cnc
- "remarkable insert" 命令 → 从 reMarkable 云端拉取文件
- `rm/` 文件夹中的 PDF → 嵌入到 Obsidian 笔记
- 在平板上修改时自动重新获取

---

## 4. OCR + AI 自定义流水线

**ROI：定制工作流时很高 | 难度：中至高**

### rmirror + Claude API（推荐模式）

来源：https://news.ycombinator.com/item?id=47110872（2026 年 2 月）

**概念**：macOS 后台智能体，功能如下：

1. 从 reMarkable 同步笔记本
2. 通过 Claude API 进行 OCR（对手写体的上下文理解优于 Tesseract）
3. 将转录的笔记作为可搜索页面推送到 Notion

**为什么 Claude 优于 Tesseract 做 OCR**：Claude 理解上下文，纠正写得不好的词语，自动组织列表和表格。

### 自制流水线

```Plain Text
reMarkable → SSH/USB
  → 提取 .rm 文件
  → rmscene 解析（Type Folio 时提取原生文字）
  → Claude Vision API（页面截图用于手写识别）
  → 结构化文本 + 自动标签
  → 通过 API 推送到 Notion/Obsidian/GitHub

```

### 解析工具

| 工具 | 用途 | 链接 |
|-|-|-|
| **rmscene** | 原生解析 .rm 文件 | https://github.com/ricklupton/rmscene |
| **rmc** | 将 .rm 转换为 SVG/PNG | https://github.com/ricklupton/rmc |
| **rmapi** | Go 语言云 API 接口 | https://github.com/juruen/rmapi |

---

## 5. SSH 访问与社区工具

**其他一切的必要基础**

### 启用 SSH

```Bash
# 通过平板界面：
# 设置 → 帮助 → 版权与许可证
# → 显示：IP + root 密码

# USB（直接连接）
ssh root@10.11.99.1

# WiFi（启用后）
rm-ssh-over-wlan on
# 或通过 "Simply Customize It"

```

### 核心工具

| 工具 | 用途 | 链接 |
|-|-|-|
| **RMHacks/xovi** | rM1/2/Paper Pro 的 mod 框架 | https://www.nilorea.net/2025/08/11/latest-rmhacks-with-xovi-for-remarkable-1-2-paper-pro/ |
| **Simply Customize It** | 切换功能的 GUI（WLAN SSH 等） | 第三方应用 |
| **ReMy** | 通过 SSH 浏览/预览/导出文档的 GUI（无需云端） | https://github.com/bordaigorl/remy |
| **rmirro** | 平板 ↔ 本地文件夹的 PDF 双向同步 | https://github.com/hersle/rmirro |
| **reStream** | 在 Mac/PC 上流式传输 reMarkable 屏幕 | https://github.com/rien/reStream |
| **KOReader** | 替代阅读器（更多格式，可定制） | 通过 SSH |
| **reGitable** | 通过 git 自动备份 | awesome-reMarkable |

### 自定义模板

```Bash
# 创建 SVG 模板 → 通过 SSH 复制
scp my-template.svg root@10.11.99.1:/usr/share/remarkable/templates/

# 编辑 templates.json 进行注册
ssh root@10.11.99.1 'vi /usr/share/remarkable/templates/templates.json'

```

**生成工具**：ReCalendar.me、Remarkable Grid Generator、Remarkably Planner Builder

---

## 6. 未充分利用的原生功能

**难度：零 | 含在 Connect 中（约 6 欧元/月）**

| 功能 | 用途 |
|-|-|
| **手写转文字** | 选中 → 转换 → 复制/粘贴到任意应用 |
| **云同步** | Google Drive、Dropbox、OneDrive |
| **发送到 Slack** | 直接将会议记录分享到频道 |
| **手写搜索**（AI 测试版） | 在历史手写笔记中搜索 |
| **屏幕共享** | 在 PC 上共享屏幕（演示、会议） |
| **发送到邮件** | 以 PDF 或 PNG 发送 |

**使用技巧**：手写转文字对孤立词语效果良好，但对密集的连笔句子效果较差。使用印刷体书写以获得更好的转换效果。

---

## 7. 官方 API 与开发者门户

| 资源 | 链接 |
|-|-|
| **开发者门户** | https://developer.remarkable.com |
| **云 API 文档** | https://github.com/splitbrain/ReMarkableAPI |
| **社区指南** | https://remarkable.guide/ |
| **rmfakecloud** | 自托管云（无需 Connect 订阅） |

**操作系统**：Linux（Codex），完整 SSH root 访问，GPL 合规。交叉编译工具链允许部署自定义原生应用。

**rmfakecloud**：reMarkable 云的开源替代方案，用于自托管同步，无需 Connect 订阅。

---

## 8. Zapier 自动化

**ROI：中等 | 难度：低 | 无需代码**

**机制**：reMarkable → 邮件（my@remarkable.com）→ Zapier 拦截 → 自动操作

### 可用目的地

Google Drive、Asana、ClickUp、Trello、Slack、WordPress、Evernote、Notion

### 免费计划

- 每月 100 个任务
- 2 步 Zap
- 每 15 分钟检查一次

### 实际工作流

```Plain Text
会议记录 → PDF 自动上传到 Google Drive
草图 → 文件发送到 Slack 频道
行动项 → 在 Asana/ClickUp 创建任务

```

**来源**：https://myremarkable.substack.com/p/integrating-remarkable

---

## 9. Read-it-later：网页 → reMarkable

**ROI：阅读场景很高 | 难度：几乎为零**

| 工具 | 说明 |
|-|-|
| **Chrome 扩展 "Read on reMarkable"** | 将任意网页保存为 EPUB/PDF 发送到平板（去除广告） |
| **Goosepaper** | RSS + 新闻 + 每日 Wikipedia → 格式化为电子墨水 |
| **remarkable_news** | 当日新闻/漫画/图片作为屏保 |
| **Instapaper 变通方案** | 以 EPUB 下载文章 → 通过桌面应用导入 |

**PDF 选项**：右键点击扩展 → "Read on reMarkable as PDF"（可调整边距以便注释）。

---

## 10. 会议记录 → AI 摘要

**ROI：很高 | 难度：极低**

### 手动工作流（无 MCP）

```Plain Text
1. 会议期间手写笔记
2. 通过 reMarkable 移动应用截图（或云同步）
3. 上传图片到 Claude/ChatGPT
4. 提示词："总结这些笔记，提取带截止日期和负责人的行动项"

```

### MCP 工作流（安装 remarkable-mcp 后）

```Plain Text
Claude，总结我今天会议的笔记
→ Claude 通过 SSH 获取文件
→ 必要时进行 OCR
→ 直接输出结构化摘要

```

**MCP 优势**：跳过截图和上传步骤。即使没有 Connect 订阅也能工作。

### 推荐模板

Paper Pro Move 会议笔记本：60 次会议，每次 5 个相互关联的页面（概览 + 笔记 + 行动项 + 跟进）。

---

## 11. Zotero → reMarkable（研究用途）

**ROI：阅读论文时很高 | 难度：中等**

| 工具 | 用途 |
|-|-|
| **Zotero2reMarkable Bridge** | 从 Zotero 同步 PDF，支持高亮 |
| **KOReader + Toltec + Zotero 插件** | 更好的双栏 PDF 阅读，双向同步 |
| **sync_zotero_remarkable** | 更轻量的替代方案 |

**客观局限性**：reMarkable 是封闭系统。Zotero 集成需要变通方案。不如原生支持 Zotero 的 Android 电子阅读器流畅。可用但有摩擦。

---

## 12. 屏幕共享作为 AI 辅助白板

**ROI：演示/会议引导 | 难度：零（Connect 原生功能）**

- **屏幕共享**：你的实时书写出现在外部屏幕/虚拟会议上
- **激光笔**：触控笔靠近屏幕顶部 → 激活激光笔
- **组合工作流**：屏幕共享 + 同事实时将你的笔记发送给 ChatGPT = 增强白板

**价格**：含在 Connect 中（美国约 30 美元/年，欧盟约 6 欧元/月）。

---

## 13. 自定义应用与趣味黑客

| 应用/黑客 | 说明 |
|-|-|
| **Ephemeris** | 从你的日历生成的每日议程（Python） |
| **Remarcal** | 同步 Google/Outlook/Apple 日历到 reMarkable |
| **reMarkable keywriter** | 零干扰键盘笔记应用 |
| **remarkable-wikipedia** | 离线 Wikipedia 阅读器 |
| **whiteboard-hypercard** | 实时协作，共享绘图 |
| **NetSurf** | 极简网页浏览器（通过 SSH） |
| **pdf2remarkable** | 从命令行上传 PDF 到云端 |
| **send-to-remarkable** | 通过邮件上传文档（类似发送到 Kindle） |
| **libreMarkable** | 开发原生应用的框架 |
| **oxide/remux/draft** | 多任务启动器 |
| **latex-yearly-planner** | LaTeX 生成的年度计划器 |

**完整目录**：https://github.com/reHackable/awesome-reMarkable

---

## 14. 待构建的 AI 增强工作流

利用现有积木可以实现但尚未打包的工作流。

### A. AI 分析日记

```Plain Text
每晚 → 在 reMarkable 上写一页反思
→ remarkable-mcp + Claude → 每周分析模式、情绪、决策
→ 输出：带连接图的 Obsidian 洞察

```

### B. 辅助收件箱处理

```Plain Text
在 reMarkable 上阅读和注释的论文/文章
→ Claude Vision OCR → 结构化摘要
→ 自动标签 + 分类到 Obsidian/Notion

```

### C. 草图转代码

```Plain Text
在 reMarkable 上画 UI 线框图
→ 截图 → Claude Vision → HTML/React 代码

```

### D. 自动闪卡

```Plain Text
课堂/阅读笔记在 reMarkable 上
→ remarkable-mcp → Claude 提取关键概念
→ 自动生成 Anki 闪卡

```

### E. 每日站会自动化

```Plain Text
每天早晨写 TODO（自定义模板）
→ OCR → 自动格式化发送到 Slack/邮件
→ 当天结束：勾选完成项，发送差异对比

```

### F. 头脑风暴捕获 → 思维导图

```Plain Text
自由涂鸦想法
→ Claude Vision 分析空间布局 + 文字
→ 生成结构化思维导图（Mermaid/Markmap）

```

---

## 15. 从哪里开始

### 第一阶段——本周末（2 小时）

1. 通过设置 → 帮助 → 版权与许可证启用 SSH
2. 安装 **remarkable-mcp** 并连接到 Claude Code
3. 测试：让 Claude 在你的笔记中搜索

### 第二阶段——下周

1. 如果你使用 Obsidian → 安装 Scrybble
2. 测试 **Ghostwriter**（10 分钟安装，保证有趣）

### 第三阶段——想深入时

1. 搭建带 Claude Vision API 的自定义 OCR 流水线
2. 探索 rmfakecloud 以摆脱 Connect 订阅

---

## 来源

**GitHub 项目**

- https://github.com/SamMorrowDrums/remarkable-mcp（MCP 服务器，2025 年 11 月）
- https://github.com/awwaiid/ghostwriter（Vision-LLM 接口）
- https://github.com/reHackable/awesome-reMarkable（社区目录）
- https://github.com/hersle/rmirro（无云同步）
- https://github.com/bordaigorl/remy（SSH GUI）
- https://github.com/rien/reStream（屏幕流式传输）
- https://github.com/splitbrain/ReMarkableAPI（云 API 文档）
- https://github.com/ricklupton/rmscene（原生 .rm 解析）

**文章与讨论**

- https://sam-morrow.com/blog/building-an-mcp-server-for-remarkable
- https://news.ycombinator.com/item?id=47110872（rmirror + Claude OCR，2026 年 2 月）
- https://news.ycombinator.com/item?id=42979986（Ghostwriter HN）
- https://news.ycombinator.com/item?id=46099997（Hacking reMarkable 2，HN 2025）
- https://sgt.hootr.club/blog/hacking-on-the-remarkable-2/（SSH 黑客指南）
- https://myremarkable.substack.com/p/integrating-remarkable（Zapier 集成）

**Obsidian**

- https://forum.obsidian.md/t/scrybble-sync-plugin/103194
- https://www.youtube.com/watch?v=EsRdi8J9Cnc（云同步演示）

**官方资源**

- https://developer.remarkable.com（开发者门户、SDK、API）
- https://remarkable.guide/（社区指南）

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/Y8dWwWD7Liey1Hkiqivck6r7nJg> ｜ 归档：2026-06-04
