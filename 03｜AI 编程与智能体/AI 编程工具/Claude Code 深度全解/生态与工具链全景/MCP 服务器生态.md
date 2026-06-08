# MCP 服务器生态

**最后更新**：2026 年 5 月 • **下次审阅**：2026 年 6 月

本指南涵盖 Anthropic 官方服务器之外经过验证的社区 MCP 服务器。所有列出的服务器均已针对生产就绪性、维护活跃度和安全性进行评估。

> **不确定该用 MCP 服务器还是 CLI 工具？** 请参阅 「MCP 与 CLI 决策指南」，其中包含完整的权衡分析、决策矩阵和场景化建议。

## 目录

- 官方服务器与社区服务器
- 评估框架
- 生态系统演进
- 经验证的社区服务器

  - 浏览器自动化
  - DevOps 与基础设施
  - 安全与代码分析
  - 代码搜索与分析
  - 文档与知识
  - 项目管理
  - 编排
- 生产部署
- 每月监测方法论
- 排除的服务器

---

## 官方服务器与社区服务器

| 类型 | 示例 | 特征 | 适用场景 |
|-|-|-|-|
| **官方** | filesystem、memory、brave-search、github | Anthropic 维护，稳定性有保障 | 默认首选，核心功能 |
| **社区** | Playwright、Semgrep、Kubernetes | 由组织/个人维护，可达生产级 | 专项需求、生态集成 |

**核心区别**：官方服务器有 Anthropic SLA 背书，社区服务器需要逐一评估。

---

## 评估框架

所有社区服务器均按以下标准评估：

| 标准 | 阈值 | 说明 |
|-|-|-|
| **GitHub Stars** | ≥50 | 最低社区认可度 |
| **近期发布** | <3 个月 | 积极维护 |
| **文档** | README + 示例 + 配置 | 降低采用门槛 |
| **测试/CI** | ✅ 自动化 | 保证稳定性 |
| **使用场景** | 官方服务器未覆盖 | 避免重复 |
| **许可证** | 必须开源 | 可持续性与可审计性 |

**质量评分维度**：

- 维护性（10 分）：发布频率、Issue 响应时间
- 文档（10 分）：README 完整性、示例、故障排查
- 测试（10 分）：测试覆盖率、CI/CD 自动化
- 性能（10 分）：响应时间、资源效率
- 采用度（10 分）：社区使用量、生产部署情况

**总分**：`/50` → 归一化为 `/10` 作为最终评级。

---

## 生态系统演进

**重大进展（2026 年 1 月）**：

### Linux 基金会标准化

MCP 通过 Linux 基金会治理下的 **Agentic AI Foundation** 成为官方标准。

- **公告**：[YouTube - Linux Foundation](https://www.youtube.com/watch?v=btNbIY7KYwg)
- **影响**：企业采用加速，长期稳定性得到保障

### 高级 MCP 工具调用

Anthropic 为 MCP 上下文管理部署优化措施：

- **延迟加载**：工具按需加载，而非预先全部加载
- **基于搜索的工具**：在大型工具集中高效发现工具
- **公告**：Josh Twist LinkedIn

### MCPB 包格式

标准化的一键安装包格式（替代运行时依赖管理）。

- **讨论**：[Reddit - r/ClaudeAI](https://www.reddit.com/r/ClaudeAI/comments/1qkzdh0/mcp_server_installs_are_nondeterministic_heres/)
- **优势**：安装结果确定性，降低配置复杂度

### MCP Apps（交互式工作工具）

Claude 现在通过 MCP Apps 规范支持交互式工具：

- **示例**：Slack 草稿撰写、Figma 图表、Asana 时间线
- **公告**：[Smol.ai Newsletter](https://news.smol.ai/issues/26-01-26-mcp-apps)
- **深度解析**：参见 「guide/architecture.md:656」

### IDE 集成

**Visual Studio 2026** 原生集成 Azure MCP Server、GitHub Copilot Chat 与 MCP 客户端。

- **公告**：[Microsoft DevBlogs](https://devblogs.microsoft.com/visualstudio/azure-mcp-server-now-built-in-with-visual-studio-2026-a-new-era-for-agentic-workflows/)

---

## 版本控制（官方服务器）

这些基础 MCP 服务器为所有开发工作流提供版本控制自动化能力。均为 **Anthropic 官方服务器**，稳定性有保障。

### Git MCP（Anthropic）

**Anthropic 官方服务器**，通过模型上下文协议实现 Git 仓库交互。提供对 Git 操作的编程化访问，具备结构化输出和跨平台安全性。

**仓库**：[modelcontextprotocol/servers/git](https://github.com/modelcontextprotocol/servers/tree/main/src/git)**许可证**：MIT**状态**：早期开发阶段（API 可能变更）**Stars**：77,908+（父仓库）

**使用场景**：

- **自动化提交工作流**：AI 生成提交信息、暂存变更、执行提交
- **日志分析**：按日期、作者、分支过滤提交，输出结构化结果
- **分支管理**：创建特性分支、切换分支、按 SHA 过滤
- **Token 高效的差异对比**：控制上下文行数，专注于代码审查
- **多仓库自动化**：在 Monorepo 中管理多个仓库

#### 核心功能

| 工具 | 描述 | 参数 |
|-|-|-|
| `git_status` | 工作区状态（已暂存、未暂存、未跟踪） | - |
| `git_log` | 带高级过滤的提交历史 | `max_count`、`skip`、`start_timestamp`、`end_timestamp`、`author` |
| `git_diff` | 提交/分支间的差异 | `target`、`source`、`context_lines` |
| `git_diff_unstaged` | 未暂存的变更 | `context_lines` |
| `git_diff_staged` | 已暂存的变更 | `context_lines` |
| `git_commit` | 创建提交 | `message` |
| `git_add` | 暂存文件/模式 | `files` |
| `git_reset` | 取消暂存文件 | `files` |
| `git_branch` | 列出/过滤分支 | `contains`、`not_contains` |
| `git_create_branch` | 创建新分支 | `name` |
| `git_checkout` | 切换分支/提交 | `ref` |
| `git_show` | 查看提交详情 | `revision` |

**高级过滤**（`git_log`）：

- **ISO 8601 日期**：`2024-01-15T14:30:25`
- **相对日期**：`2 weeks ago`、`yesterday`、`last month`
- **绝对日期**：`2024-01-15`、`Jan 15 2024`
- **作者过滤**：`--author="John Doe"`

#### 配置方法

**安装（3 种方式）**：

```Bash
# 方式 1：UV（推荐）——一行命令
uvx mcp-server-git --repository /path/to/repo

# 方式 2：pip + Python 模块
pip install mcp-server-git
python -m mcp_server_git

# 方式 3：Docker（沙箱隔离）
docker run -v /path/to/repo:/repo ghcr.io/modelcontextprotocol/mcp-server-git

```

**Claude Code 配置**（`~/.claude.json`）：

```JSON
{
  "mcpServers": {
    "git": {
      "command": "uvx",
      "args": ["mcp-server-git", "--repository", "/Users/you/projects/myrepo"]
    }
  }
}

```

**多仓库支持**：

```JSON
{
  "mcpServers": {
    "git-main": {
      "command": "uvx",
      "args": ["mcp-server-git", "--repository", "/path/to/main-repo"]
    },
    "git-docs": {
      "command": "uvx",
      "args": ["mcp-server-git", "--repository", "/path/to/docs-repo"]
    }
  }
}

```

#### IDE 集成

**支持一键安装的平台**：

- **Claude Desktop**（macOS/Windows/Linux）
- **VS Code**（稳定版 + Insiders）
- **Zed**
- **Zencoder**

集成链接详见 [官方 README](https://github.com/modelcontextprotocol/servers/tree/main/src/git#quickstart)。

#### 质量评分

**8.5/10** ⭐⭐⭐⭐⭐

| 标准 | 得分 | 说明 |
|-|-|-|
| 维护性 | 10/10 | Anthropic 背书，积极开发中 |
| 文档 | 9/10 | README 详尽、有示例，但含早期开发警告 |
| 测试 | 8/10 | 自动化 CI，覆盖率持续提升 |
| 性能 | 8/10 | 快速（<100ms），结构化输出减少 Token 消耗 |
| 采用度 | 8/10 | 官方服务器，77K+ Stars，广泛的 IDE 支持 |

#### 局限性与解决方案

| 局限性 | 解决方案 |
|-|-|
| **早期开发阶段**（API 可能变更） | 生产环境固定版本，持续关注发布动态 |
| **不支持交互式变基**（`-i` 参数） | 使用 Bash 工具执行 `git rebase -i` |
| **不支持 reflog** | 使用 Bash 工具执行 `git reflog` |
| **不支持 git bisect** | 使用 Bash 工具执行 `git bisect` |
| **每个实例对应单一仓库** | 配置多个 MCP 服务器实例 |

#### 决策矩阵：Git MCP vs GitHub MCP vs Bash 工具

**各工具适用场景**：

| 操作 | Git MCP | GitHub MCP | Bash 工具 | 说明 |
|-|-|-|-|-|
| **本地提交** | ✅ 最优 | ❌ | ⚠️ 可用 | 结构化输出，跨平台安全 |
| **分支管理** | ✅ 最优 | ❌ | ⚠️ 可用 | `git_branch` 过滤，SHA 包含/排除 |
| **差异/日志分析** | ✅ 最优 | ❌ | ⚠️ 可用 | `context_lines` 控制，Token 高效 |
| **暂存文件** | ✅ 最优 | ❌ | ⚠️ 可用 | 模式匹配（`git_add`），更安全 |
| **创建 PR** | ❌ | ✅ 最优 | ⚠️ gh CLI | GitHub API，支持标签、指派、审阅者 |
| **Issue 管理** | ❌ | ✅ 最优 | ⚠️ gh CLI | GitHub 专属操作 |
| **CI/CD 状态检查** | ❌ | ✅ 最优 | ⚠️ gh CLI | GitHub Actions 集成 |
| **交互式变基** | ❌ | ❌ | ✅ 最优 | Git MCP 不支持 `-i` 参数 |
| **Reflog 恢复** | ❌ | ❌ | ✅ 最优 | 高级 Git 操作 |
| **Git Bisect 调试** | ❌ | ❌ | ✅ 最优 | 复杂调试工作流 |
| **多工具流水线** | ✅ | ✅ | ❌ | MCP 服务器可与其他 MCP 工具组合 |

**决策树**：

```Plain Text
是否为 GitHub 专属操作（PR、Issue、Actions）？
├─ 是 → 使用 GitHub MCP
└─ 否 → 是否为核心 Git 操作（commit、branch、diff、log）？
    ├─ 是 → 使用 Git MCP（结构化、安全、Token 高效）
    └─ 否 → 是否为高级 Git 功能（rebase -i、reflog、bisect）？
        ├─ 是 → 使用 Bash 工具（灵活性）
        └─ 否 → 默认使用 Git MCP（更安全、结构化）

```

**工作流示例**：

| 工作流 | 工具链 | 说明 |
|-|-|-|
| **功能开发** | Git MCP（`git_create_branch` + `git_commit`）→ GitHub MCP（PR） | 原子化、结构化、全生命周期覆盖 |
| **提交历史分析** | Git MCP（`git_log`，`start_timestamp: "2 weeks ago"`） | Token 高效过滤，支持相对日期 |
| **代码审查准备** | Git MCP（`git_diff`，`context_lines: 3`） | 聚焦上下文，减少 Token 消耗 |
| **整理提交（变基）** | Bash 工具（`git rebase -i HEAD~5`） | Git MCP 不支持交互模式 |
| **恢复丢失的提交** | Bash 工具（`git reflog`） | Git MCP 未暴露 reflog |
| **Bisect 排查 Bug** | Bash 工具（`git bisect start/good/bad`） | Git MCP 不支持 bisect 工作流 |
| **自动化发布流程** | Git MCP（commit + tag）→ GitHub MCP（创建 Release） | 全自动化，结构化 |

#### 参考资源

- **GitHub**：https://github.com/modelcontextprotocol/servers/tree/main/src/git
- **父仓库**：https://github.com/modelcontextprotocol/servers（77,908+ Stars）
- **MCP Inspector**：支持实时调试测试
- **Docker Hub**：`ghcr.io/modelcontextprotocol/mcp-server-git`

---

## 经验证的社区服务器

### 浏览器自动化

#### Playwright MCP（Microsoft）

**Microsoft 官方服务器**，专为 LLM 优化的浏览器自动化工具。使用无障碍树（accessibility tree）而非截图，降低 Token 消耗。

**使用场景**：AI 编程智能体在浏览器中验证自身工作（E2E 测试、Bug 验证）。

**核心功能**：

| 功能 | 详情 |
|-|-|
| 浏览器自动化 | 导航、点击、填写、悬停（Playwright API） |
| 内容提取 | 通过无障碍树提取结构化数据 |
| 截图 | 全页面 + 指定元素截图 |
| JavaScript 执行 | 在页面上下文中运行代码 |
| 会话管理 | 持久化浏览器状态 |
| 支持的浏览器 | Chromium、Firefox、WebKit |

**配置方法**：

```Bash
# 安装
npm install @microsoft/playwright-mcp
# 或
npx @microsoft/playwright-mcp

```

**Claude Code 配置**（`~/.claude.json`）：

```JSON
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["--yes", "@microsoft/playwright-mcp"]
    }
  }
}

```

**使用示例**：

```Plain Text
用户："导航到 example.com，用邮箱 test@example.com 登录，然后截图"

Claude：[执行 playwright_navigate → playwright_type → playwright_click → playwright_screenshot]

结果：截图 + 无障碍树信息已纳入上下文

```

**质量评分**：**8.8/10** ⭐⭐⭐⭐⭐

| 维度 | 得分 | 说明 |
|-|-|-|
| 维护性 | 9/10 | 每两周发布，Microsoft 团队活跃 |
| 文档 | 9/10 | README 完整，有示例和 Playwright Live 视频 |
| 测试 | 10/10 | 完整测试套件，CI/CD 自动化 |
| 性能 | 8/10 | 快速快照（~200ms），内存高效 |
| 采用度 | 8/10 | 2890+ 次使用（Smithery.ai 统计） |

**局限性与解决方案**：

| 局限性 | 解决方案 |
|-|-|
| 单浏览器会话 | 使用 session ID 持久化状态 |
| 不支持跨域 iframe 访问 | 限制为同源内容 |
| 截图大小限制（最大 4K） | 对大型页面使用元素快照 |

**备选方案**：

| 服务器 | 优势 | 劣势 |
|-|-|-|
| **Playwright MCP** | 无障碍树，LLM 原生 | 不支持视觉模型 |
| Browserbase MCP | 云端，支持隐身模式 | API 费用，延迟较高 |
| Puppeteer MCP | 轻量，仅限 JS | 结构化数据较少 |

**参考资源**：

- **GitHub**：https://github.com/microsoft/playwright-mcp
- **发布记录**：https://github.com/microsoft/playwright-mcp/releases
- **Playwright Live 演示**：https://youtu.be/CNzg1aPwrKI

---

#### Browserbase MCP

**Browserbase 官方服务器**，提供云端浏览器自动化，内置 Stagehand AI 智能体用于自主任务执行。

**使用场景**：需要隐身模式、代理支持或自主执行的复杂 Web 交互（网页抓取、表单填写、数据提取）。

**核心功能**：

| 功能 | 详情 |
|-|-|
| 浏览器控制 | 通过 Browserbase 云端运行 Chromium |
| Stagehand 智能体 | 自主任务执行（例如"预订航班"） |
| 数据提取 | CSS 选择器 + 基于 Schema 的结构化提取 |
| 反检测 | 隐身模式、代理支持、IP 轮换 |
| 多模型支持 | OpenAI、Claude、Gemini、自定义 LLM |

**配置方法**：

```Bash
npm install @browserbasehq/mcp-server-browserbase

```

**配置**：

```JSON
{
  "mcpServers": {
    "browserbase": {
      "command": "npx",
      "args": ["@browserbasehq/mcp-server-browserbase"],
      "env": {
        "BROWSERBASE_API_KEY": "YOUR_KEY",
        "BROWSERBASE_PROJECT_ID": "YOUR_PROJECT_ID",
        "GEMINI_API_KEY": "YOUR_GEMINI_KEY"
      }
    }
  }
}

```

**质量评分**：**7.6/10** ⭐⭐⭐⭐

**费用**：Freemium（按 API 用量付费），约 $0.10/会话

**局限性**：

| 局限性 | 解决方案 |
|-|-|
| 云端延迟（~500ms） | 批量操作，缓存结果 |
| API 费用 | 仅用于高价值提取任务 |
| Stagehand 能力限制 | 回退到手动 `playwright_*` 工具 |

**参考资源**：

- **GitHub**：https://github.com/browserbase/mcp-server-browserbase
- **官方文档**：https://www.browserbase.com

---

#### Chrome DevTools MCP

**Anthropic 官方服务器**，集成 Chrome DevTools Protocol，通过 Chrome 原生 DevTools API 提供调试与检查能力。

**使用场景**：调试 Web 应用、检查运行时状态、监控网络请求、分析性能。与 Playwright MCP（测试）互补，专注于面向开发的调试能力。

**核心功能**：

| 功能 | 详情 |
|-|-|
| 控制台访问 | 读取浏览器控制台日志、错误、警告 |
| 网络监控 | 检查 HTTP 请求、响应、请求头 |
| DOM 检查 | 查询 DOM 结构、元素属性 |
| JavaScript 执行 | 在页面上下文中执行任意 JS |
| 性能分析 | CPU 分析、内存快照 |

**配置方法**：

```Bash
npm install @modelcontextprotocol/server-chrome-devtools

```

**配置**：

```JSON
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-chrome-devtools"]
    }
  }
}

```

**使用场景对比**：

| 场景 | 使用 Chrome DevTools MCP | 使用 Playwright MCP |
|-|-|-|
| 调试运行时错误 | ✅ 控制台日志、堆栈跟踪 | ❌ 错误可见性有限 |
| 检查网络请求 | ✅ 完整请求/响应详情 | ⚠️ 仅基本导航 |
| 测试用户交互 | ❌ 非为测试设计 | ✅ 点击、输入、导航 |
| 性能分析 | ✅ CPU/内存分析 | ❌ 无分析工具 |
| 自动化工作流 | ❌ 专注手动调试 | ✅ E2E 测试自动化 |

**局限性**：

- 需要 Chrome 浏览器以 DevTools Protocol 模式启动
- 需手动配置（以 `--remote-debugging-port` 启动 Chrome）
- 不适合自动化测试（请使用 Playwright）
- 启用性能分析时存在性能开销

**参考资源**：

- **npm**：https://www.npmjs.com/package/@modelcontextprotocol/server-chrome-devtools
- **Chrome DevTools Protocol**：https://chromedevtools.github.io/devtools-protocol/

---

### DevOps 与基础设施

#### Kubernetes MCP（Red Hat）

**官方 Containers Community 服务器**（Red Hat 背书），支持用自然语言管理 Kubernetes/OpenShift。

**使用场景**：DevOps/SRE 工程师通过 Claude 查询/配置集群（"自然语言版 kubectl"）。

**核心功能**：

| 功能 | 详情 |
|-|-|
| 资源 CRUD | 创建、读取、更新、删除任意 K8s 资源 |
| Pod 操作 | 日志、事件、exec、指标（top） |
| Deployment 管理 | 扩缩容、滚动更新、状态查询 |
| 配置管理 | 查看/更新 ConfigMaps、Secrets |
| CRD 支持 | 自定义资源定义 |
| 多集群 | 切换 kubeconfig 上下文 |
| OpenShift 支持 | 原生 OpenShift 资源 |

**配置方法**：

```Bash
# Docker
docker run -it --rm \
  --mount type=bind,src=$HOME/.kube/config,dst=/home/mcp/.kube/config \
  ghcr.io/containers/kubernetes-mcp-server

# 原生（Go 二进制）
go install github.com/containers/kubernetes-mcp-server@latest
kubernetes-mcp-server

```

**Claude Desktop 配置**：

```JSON
{
  "mcpServers": {
    "kubernetes": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "--mount",
        "type=bind,src=/home/user/.kube/config,dst=/home/mcp/.kube/config",
        "ghcr.io/containers/kubernetes-mcp-server"
      ]
    }
  }
}

```

**使用示例**：

```Plain Text
用户："显示 production 命名空间中内存使用超过 500Mi 的所有 Pod"
Claude：[对 Pod + 指标执行 list_resources]
结果：含内存统计的 Pod 列表

用户："将 backend deployment 扩容到 5 个副本"
Claude：[执行 patch_resource]
结果：Deployment 已扩容

```

**质量评分**：**8.4/10** ⭐⭐⭐⭐

**安全性**：RBAC 强制执行，kubeconfig 鉴权，无权限提升

**局限性**：

| 局限性 | 解决方案 |
|-|-|
| 需要 kubeconfig 访问权限 | 使用 ServiceAccount + RBAC 确保安全 |
| 节点 Shell 访问受限 | 使用 `kubectl exec` 进行调试 |
| CRD 发现有延迟 | 为 AI 上下文预先记录 CRD |

**参考资源**：

- **GitHub**：https://github.com/containers/kubernetes-mcp-server
- **Red Hat 文档**：https://developers.redhat.com/articles/2025/09/25/kubernetes-mcp-server-ai-powered-cluster-management

---

#### Vercel MCP

**社区服务器**，支持 Vercel 平台（部署、项目、环境变量、团队）。

**使用场景**：AI 助手生成 Next.js 代码，创建 Vercel 项目，配置环境变量，触发部署——在 IDE 内完成完整 CI/CD 闭环。

**核心功能**：

| 功能 | 详情 |
|-|-|
| 部署 | 列出、查看详情、创建、监控状态 |
| 项目 | 列出、创建、更新设置 |
| 环境变量 | 获取、设置、管理 Secrets |
| 团队 | 列出、创建、管理 |
| 域名 | 列出、配置、DNS 管理 |
| Functions | 监控 Vercel Functions、日志 |

**配置方法**：

```Bash
git clone https://github.com/nganiet/mcp-vercel
cd vercel-mcp
npm install

```

**配置**：

```JSON
{
  "mcpServers": {
    "vercel": {
      "command": "npm",
      "args": ["start"],
      "env": {
        "VERCEL_API_TOKEN": "YOUR_VERCEL_TOKEN"
      }
    }
  }
}

```

**质量评分**：**7.6/10** ⭐⭐⭐⭐

**说明**：Vercel 也有官方 MCP 服务器，本社区版本提供更全面的 API 覆盖。

**参考资源**：

- **GitHub**：https://github.com/nganiet/mcp-vercel
- **Vercel 文档**：https://vercel.com/docs/mcp/deploy-mcp-servers-to-vercel
- **Vercel 官方 MCP**：https://vercel.com/docs/mcp/vercel-mcp

#### Sentry MCP

**Sentry 官方服务器**，用于错误监控与可观测性。形成诊断闭环：Sentry 告警触发 → Claude 读取 Issue 和堆栈跟踪 → 诊断根因 → 提出或编写修复方案。

**仓库**：[getsentry/sentry-mcp](https://github.com/getsentry/sentry-mcp)**许可证**：MIT**维护方**：Sentry（官方）

**使用场景**：生产环境触发 Sentry 告警，工程师问 Claude："SEN-4521 是什么原因？"。Claude 读取完整堆栈跟踪，追踪代码库中的回归路径，起草修复方案——全程无需离开 IDE。可观测性闭环在 Claude Code 内完成。

**核心功能**：

| 工具 | 描述 |
|-|-|
| `list_issues` | 使用 Sentry 查询语法获取未解决的 Issue（`is:unresolved level:error`） |
| `get_issue` | 完整 Issue 详情——堆栈跟踪、受影响用户、首次/最近出现时间戳 |
| `get_event` | 通过 ID 获取特定事件，适用于时间范围调查 |
| `search_events` | 带字段过滤的原始事件全文搜索 |
| `list_projects` | 列出 Sentry 组织中的项目 |

**配置方法**：

```Bash
# 通过 npx（推荐——请对照官方文档确认包名）
npx -y @sentry/mcp-server

# Claude Code 一键添加
claude mcp add sentry -- npx -y @sentry/mcp-server

```

**Claude Code 配置**（`~/.claude/settings.json`）：

```JSON
{
  "mcpServers": {
    "sentry": {
      "command": "npx",
      "args": ["-y", "@sentry/mcp-server"],
      "env": {
        "SENTRY_AUTH_TOKEN": "your_auth_token",
        "SENTRY_ORG": "your-org-slug"
      }
    }
  }
}

```

> 认证 Token：[sentry.io/settings/account/api/auth-tokens/](https://sentry.io/settings/account/api/auth-tokens/)——所需权限：`project:read`、`event:read`、`org:read`

**使用示例**：

```Plain Text
用户："SEN-4521 是什么原因？昨天部署后一直在触发。"

Claude：
  [list_issues: query="is:unresolved level:error project:api-service"]
  [get_issue: issue_id="4521"]

结果：UserController.getProfile() 第 142 行 NullPointerException。
  在昨天 14:32 UTC 的提交 a3f8c2 中引入——Profile 重构时移除了 null 检查。
  修复方案：在第 142 行恢复 Optional.ofNullable。
  正在创建 PR。

```

**查询语法**（影响调用效果的关键——最常见的调用失败来源）：

```Plain Text
is:unresolved                         # 仅未解决的 Issue
is:unresolved level:error             # 仅错误（排除警告、info）
is:unresolved has:user                # 已识别用户的 Issue
is:unresolved times_seen:>100         # 高频 Issue
project:api-service is:unresolved     # 限定单个项目
assigned:me is:unresolved             # 分配给自己的 Issue
!has:assignee is:unresolved           # 未分配的 Issue

```

> **参考文件**：本仓库 `examples/skills/mcp-integration-reference/references/sentry-mcp.md`——包含完整参数文档、注意事项、分页模式及精选降噪列表。可将其复制到 CLAUDE.md includes 或项目 Skills 中。

**质量评分**：**8.5/10** ⭐⭐⭐⭐⭐

| 维度 | 得分 | 说明 |
|-|-|-|
| 维护性 | 10/10 | Sentry 官方服务器，企业级背书 |
| 文档 | 8/10 | README 良好，Sentry 文档涵盖边界情况 |
| 测试 | 8/10 | 存在 CI，TypeScript 类型安全 |
| 性能 | 8/10 | API 受限（~200–400ms），大型组织需分页 |
| 采用度 | 9/10 | Sentry 是事实标准错误监控平台（100K+ 组织） |

**局限性与解决方案**：

| 局限性 | 解决方案 |
|-|-|
| `organization_slug` ≠ 显示名称 | 从 URL 读取 Slug：`sentry.io/organizations/<slug>/` |
| 大型组织中 `search_events` 超时 | 搜索事件时始终指定 `project_slug` |
| 每次调用最多返回 100 个 Issue | 使用基于游标的分页进行完整扫描 |
| 默认只读 | 解决/指派操作需要额外的 Token 权限 |
| 事件保留 90 天 | 默认 Sentry 方案下 90 天前的事件不可用 |

**适用场景对比**：

| 工具 | 最适合 | 不推荐的情况 |
|-|-|-|
| **Sentry MCP** | 错误诊断闭环：告警 → 堆栈跟踪 → 修复 | 纯告警（直接用 Webhook 或 PagerDuty） |
| **Datadog MCP** | APM、分布式追踪、指标仪表盘 | 仅处理错误——对该场景过于复杂 |
| **Bash + Sentry CLI** | 批量操作、脚本化数据导出 | 交互式调试会话 |

**参考资源**：

- **GitHub**：https://github.com/getsentry/sentry-mcp
- **Sentry MCP 文档**：https://docs.sentry.io/product/sentry-mcp/
- **参考文件**：`examples/skills/mcp-integration-reference/references/sentry-mcp.md`
- **Auth Token 配置**：https://sentry.io/settings/account/api/auth-tokens/

---

### 安全与代码分析

#### Semgrep MCP

**Semgrep 官方服务器**，用于漏洞扫描（SAST、Secrets、供应链）。内置自定义规则引擎。

**使用场景**：Claude Code 生成代码，Semgrep 自动扫描安全问题，提出修复方案（"默认安全"）。

**核心功能**：

| 功能 | 详情 |
|-|-|
| 快速扫描 | 对代码片段进行快速安全检查 |
| 全量扫描 | 使用 p/ci 规则集进行全面 SAST |
| 自定义规则 | 使用用户提供的 Semgrep 规则扫描 |
| AST 生成 | 抽象语法树分析 |
| 规则集支持 | 预置规则集（OWASP、CWE 等） |
| 语言覆盖 | Python、JS/TS、Java、Go、C#、Rust、PHP 等 |

**配置方法**：

```Bash
# 通过 uvx（推荐）
uvx semgrep-mcp

# 或 pip
pip install semgrep-mcp

```

**Claude Code 配置**：

```Bash
claude mcp add semgrep -- uvx semgrep-mcp

```

**Cursor 配置**（`~/.cursor/mcp.json`）：

```JSON
{
  "mcpServers": {
    "semgrep": {
      "command": "uvx",
      "args": ["semgrep-mcp"],
      "env": {
        "SEMGREP_APP_TOKEN": "your_token"
      }
    }
  }
}

```

**使用示例**：

```Plain Text
用户："扫描这段 Python 代码中的 SQL 注入漏洞"

代码：
  def search(query):
      return db.execute(f"SELECT * FROM users WHERE name = '{query}'")

Claude：[使用 security_check 工具]

结果：[存在漏洞] 第 2 行检测到 SQL 注入。
  修复方案：使用参数化查询：
  return db.execute("SELECT * FROM users WHERE name = ?", [query])

```

**质量评分**：**9.0/10** ⭐⭐⭐⭐⭐

| 维度 | 得分 | 说明 |
|-|-|-|
| 维护性 | 10/10 | 官方维护，频繁发布 |
| 文档 | 9/10 | 文档完整，有示例 |
| 测试 | 10/10 | 广泛测试覆盖 |
| 性能 | 7/10 | 良好，复杂度相关（每次扫描约 500ms） |
| 采用度 | 9/10 | 企业标准（5000+ 家公司） |

**备选方案**：

| 服务器 | 优势 | 劣势 |
|-|-|-|
| **Semgrep** | 全面 SAST，自定义规则 | 大型代码库较慢 |
| GitGuardian | 专注 Secrets，速度快 | SAST 覆盖有限 |
| SonarQube | 企业级，报告详细 | 较重，配置复杂 |

**参考资源**：

- **GitHub**：https://github.com/semgrep/mcp
- **官方文档**：https://semgrep.dev/docs/mcp
- **规则注册表**：https://semgrep.dev/r
- **定价**：https://semgrep.dev/pricing（MCP 有免费层）

---

### 代码搜索与分析

#### Grepai MCP

**社区服务器**，通过本地 Ollama Embeddings 提供语义代码搜索和调用图分析。按意图（"支付流程"、"认证逻辑"）而非精确模式搜索代码，并追踪函数调用关系。

**仓库**：[yoanbernabeu/grepai](https://github.com/yoanbernabeu/grepai)**许可证**：MIT**状态**：积极开发中**隐私保护**：完全本地运行（Ollama + nomic-embed-text），数据不离开本机

**使用场景**：开发者需要理解陌生代码库 → grepai 通过自然语言描述找到相关代码并映射函数依赖，无需读取完整文件。

**核心功能**：

| 功能 | 详情 |
|-|-|
| `grepai_search` | 自然语言语义搜索（例如"错误处理中间件"） |
| `grepai_trace_callers` | 查找调用指定符号的所有函数 |
| `grepai_trace_callees` | 查找指定符号调用的所有函数 |
| `grepai_trace_graph` | 完整调用图（调用者 + 被调用者），支持配置深度 |
| `grepai_index_status` | 健康检查：已索引文件数、分块数、配置信息 |

**Token 效率对比**：

| 工作流 | Token 数 | 评价 |
|-|-|-|
| Grep + 读取文件（暴力方式） | ~15K | 噪音大，大量无关上下文 |
| grepai 搜索 + 追踪 | ~4K | 精准，只返回相关结果 |
| grepai 单独使用（无后续） | ~2-3K | 快速发现 |

**配置方法**：

```Bash
# 安装 grepai
curl -sSL https://raw.githubusercontent.com/yoanbernabeu/grepai/main/install.sh | sh

# 安装 Ollama + Embedding 模型
brew install ollama
ollama pull nomic-embed-text

# 在项目中初始化
cd /path/to/project
grepai init  # 选择：ollama, nomic-embed-text, gob

# 索引代码库
grepai index

# 可选：监听文件变更（自动重新索引）
grepai watch

```

**Claude Code 配置**：

```Bash
claude mcp add grepai -- grepai mcp

```

**.mcp.json（项目级作用域）**：

```JSON
{
  "mcpServers": {
    "grepai": {
      "command": "grepai",
      "args": ["mcp"]
    }
  }
}

```

**使用示例**：

```Plain Text
用户："在这个代码库中找到认证流程"

Claude：[执行 grepai_search query="authentication flow" limit=5]

结果：3 个相关文件及行号和相似度分数
  - src/auth/middleware.ts:12-45 (0.89)
  - src/routes/login.ts:8-32 (0.85)
  - src/utils/jwt.ts:1-28 (0.78)

用户："哪些函数调用了 validateToken？"

Claude：[执行 grepai_trace_callers symbol="validateToken"]

结果：调用图显示 3 个文件中的 4 个调用方
  - authMiddleware → validateToken
  - refreshHandler → validateToken
  - wsAuthGuard → validateToken
  - testHelper → validateToken

```

**质量评分**：**7.8/10** ⭐⭐⭐⭐

| 维度 | 得分 | 说明 |
|-|-|-|
| 维护性 | 8/10 | 积极开发，维护者响应及时 |
| 文档 | 7/10 | README 良好，有 MCP 集成文档 |
| 测试 | 7/10 | 有 CI，覆盖率持续提升 |
| 性能 | 8/10 | 本地 Embeddings 速度快（~2s 搜索），无网络延迟 |
| 采用率 | 9/10 | 社区不断壮大，已在 Claude Code 生产环境中使用 |

**限制与解决方案**：

| 限制 | 解决方案 |
|-|-|
| 需要本地运行 Ollama | `brew services start ollama`（开机自启） |
| 索引可能过期 | 使用 `grepai watch` 自动重建索引 |
| 不适合精确模式匹配 | 使用原生 Grep 工具处理正则表达式 |
| 需下载嵌入模型（约 270MB） | 一次性执行 `ollama pull nomic-embed-text` |

**替代方案**：

| 服务器 | 优点 | 缺点 |
|-|-|-|
| **Grepai** | 本地私有，支持语义搜索和调用图谱 | 需要配置 Ollama |
| 原生 Grep | 即时响应，支持精确模式 | 不具备语义理解 |
| GitHub 代码搜索 | 基于云端，支持跨仓库 | 需要 GitHub，不支持调用图谱 |

**交叉参考**：详细用法、提示策略及与其他 MCP 服务器的集成方式，请参阅 「ultimate-guide.md — MCP 服务器：Grepai」。

**相关资源**：

- **GitHub**：https://github.com/yoanbernabeu/grepai
- **Ollama**：https://ollama.com
- **嵌入模型**：nomic-embed-text（nomic-ai）

---

### 文档与知识

#### Context7 MCP

用于实时获取库文档（LangChain、Anthropic SDK 等）的**官方 Upstash 服务器**，可消除 API 幻觉问题。

**使用场景**：Claude Code 需要调用某个库的 API → Context7 提供最新文档及示例代码。

**核心功能**：

| 功能 | 详情 |
|-|-|
| 库搜索 | 检索 500+ 个库的文档 |
| 代码示例 | 各语言专属示例（Python、TS 等） |
| API 参考 | 详细的函数签名与参数说明 |
| 版本筛选 | 获取特定库版本的文档 |
| 智能排序 | AI 按相关性和项目使用频率排序 |

**安装方式**：

```Bash
# 本地运行
npx -y @upstash/context7-mcp --api-key YOUR_API_KEY

```

**Claude Code 配置（本地）**：

```Bash
claude mcp add context7 -- npx -y @upstash/context7-mcp --api-key YOUR_API_KEY

```

**Claude Code 配置（远程/HTTP）**：

```Bash
claude mcp add --transport http --header "CONTEXT7_API_KEY: YOUR_API_KEY" \
  context7 https://mcp.context7.com/mcp

```

**使用示例**：

```Plain Text
用户："告诉我如何用 Python SDK 调用 Claude 的流式 API"

Claude：[使用 context7 搜索]

结果：官方 Python SDK 文档 + 流式输出示例代码

```

**质量评分**：**8.2/10** ⭐⭐⭐⭐

**限制**：

| 限制 | 解决方案 |
|-|-|
| 库覆盖范围有限 | 冷门库退回至网页搜索 |
| 版本延迟（1-2 天） | 前沿内容直接查官方仓库 |
| 存在幻觉风险（较低） | 与官方文档交叉验证 |

**替代方案**：

| 服务器 | 优点 | 缺点 |
|-|-|-|
| **Context7** | 实时更新，版本精准 | 需要 API Key |
| 网页搜索 | 覆盖全面，免费 | 速度慢，存在幻觉风险 |
| 静态 RAG | 速度快，本地运行 | 内容过期，不支持版本 |

**相关资源**：

- **GitHub**：https://github.com/upstash/context7
- **官网**：https://context7.com
- **LobeHub 注册表**：https://lobehub.com/mcp/upstash-context7

**ctx7 CLI 配套工具**：Context7 还附带了一个 CLI（`npx ctx7`），用于从终端进行技能发现和 MCP 配置。`ctx7 skills suggest` 可自动检测项目依赖并推荐匹配技能；`ctx7 setup --claude` 会运行向导，自动配置 MCP 或 CLI+Skills 模式。完整工作流程详见《Claude Code：从入门到大师》第 5.5 节。

---

### 项目管理

#### Linear MCP

用于 Linear（项目管理 SaaS）的**社区服务器**，支持 GraphQL API，可进行问题管理、项目、团队、评论等操作。

**使用场景**：Claude Code 自动创建工单、更新状态、在 Linear 中关联问题（打通开发与项目管理之间的闭环）。

**核心功能**：

| 功能 | 详情 |
|-|-|
| 问题管理 | 列出、获取、创建、更新、删除、搜索 |
| 项目 | 列出、创建、更新、分配 |
| 团队与成员 | 团队管理、成员分配 |
| 评论 | 添加、列出，支持位置追踪 |
| 周期 | Sprint/周期管理 |
| Webhooks | 订阅 Linear 事件（可选） |

**安装方式**：

```Bash
# NPM 或 uvx
npm install mcp-linear
# 或
uvx mcp-linear

```

**Claude Code 配置**：

```Bash
claude mcp add linear -- npx -y mcp-linear --api-key YOUR_LINEAR_API_KEY

```

**使用示例**：

```Plain Text
用户："在 Linear 中为我刚发现的 CSS 布局问题创建一个 Bug 工单"

Claude：[使用 linear.issues.create，附带团队标识、标题、描述]

结果：工单创建成功，返回问题 ID

用户："将工单 SOFT-123 的状态更新为'进行中'"

Claude：[使用 linear.issues.update]

结果：状态已更改

```

**质量评分**：**7.6/10** ⭐⭐⭐⭐

**备注**：由社区维护（非 Linear 官方），但活跃且文档完善。

**限制**：

| 限制 | 解决方案 |
|-|-|
| 超时问题（1 小时后恢复） | 实现心跳检测，检查防火墙配置 |
| 字段 65KB 上限 | 评论内容自动分块 |
| GraphQL 复杂度限制 | 自动拆分复杂查询 |

**替代方案**：

| 服务器 | 优点 | 缺点 |
|-|-|-|
| **Linear MCP** | 现代 GraphQL，适合初创团队 | 社区维护 |
| Jira MCP | 企业级，支持复杂工作流 | 体量较重，API 较旧 |
| GitHub Issues | 内置，免费 | 项目管理功能有限 |

**相关资源**：

- **GitHub**：https://github.com/tacticlaunch/mcp-linear
- **Linear API**：https://developers.linear.app
- **文档**：https://jan.ai/docs/desktop/mcp-examples/productivity/linear

---

### 编排

#### MCP-Compose

用于以 Docker Compose 风格管理多个 MCP 服务器的**社区工具**，支持声明式 YAML 配置和多传输协议（STDIO/HTTP/SSE）。

**使用场景**：开发者需要管理 5 个以上 MCP 服务器时，Docker Compose 风格的配置大幅简化生命周期管理。

**核心功能**：

| 功能 | 详情 |
|-|-|
| YAML 配置 | Docker Compose 风格的服务器定义 |
| 多传输协议 | 支持 STDIO、HTTP、SSE、TCP |
| 容器运行时 | 支持 Docker、Podman 及原生进程 |
| 网络管理 | 自动创建 Docker 网络 |
| 健康监控 | 连接池与会话管理 |
| HTTP 代理 | 统一单一 HTTP 端点 |
| 热重载 | 无需重启即可更新配置 |

**安装方式**：

```Bash
git clone https://github.com/phildougherty/mcp-compose
cd mcp-compose
cargo build --release

```

**配置**（`mcp-compose.yaml`）：

```YAML
version: "1.0"
mcpServers:
  filesystem:
    command: npx
    args:
      - "@modelcontextprotocol/server-filesystem"
      - "/tmp"
    transport: stdio

  memory:
    command: npx
    args:
      - "@modelcontextprotocol/server-memory"
    transport: stdio
    env:
      DEBUG: "true"

  postgres:
    image: postgres:15
    transport: tcp
    port: 5432
    env:
      POSTGRES_PASSWORD: secret

proxy:
  port: 3000
  listen: "127.0.0.1"

```

**生成 Claude Desktop 配置**：

```Bash
./mcp-compose create-config --type claude --output ~/.claude.json

```

**启动服务器**：

```Bash
./mcp-compose up
# 统一 HTTP 代理运行于 http://localhost:3000

```

**质量评分**：**7.4/10** ⭐⭐⭐⭐

**限制**：

| 限制 | 解决方案 |
|-|-|
| 需要 Cargo 编译 | 使用预编译二进制（如有） |
| YAML 学习曲线 | 提供常见配置模板 |
| 调试较复杂 | 使用 `mcp-compose logs` 排查问题 |

**相关资源**：

- **GitHub**：https://github.com/phildougherty/mcp-compose
- **Docker Compose 文档**：https://docs.docker.com/compose/
- **MCP 协议规范**：https://modelcontextprotocol.io

---

#### Packmind

**社区工具**，用于将工程规范作为 AI 上下文跨多个智能体和仓库分发。提供一个 MCP 服务器，支持直接在 Claude Code（或任何支持 MCP 的智能体）中创建和管理 Playbook 规范。

**使用场景**：工程团队维护一份统一的 Playbook；Packmind MCP 服务器允许 Claude Code 在会话过程中直接提出新规范或更新现有规范，无需离开编辑器。

**核心功能**：

| 功能 | 详情 |
|-|-|
| 规范创建 | 通过 MCP 工具创建/更新 Playbook 条目 |
| 多智能体输出 | 从单一来源生成 CLAUDE.md、.cursor/rules、Copilot 说明 |
| 知识导入 | 通过各自的 MCP 服务器从 GitHub、Slack、Jira、GitLab、Confluence、Notion 拉取上下文 |
| 自托管 | 支持 Docker/Kubernetes，CLI 采用 Apache-2.0 授权 |

**相关资源**：

- **GitHub**：https://github.com/PackmindHub/packmind
- **示例用例**：https://github.com/PackmindHub/demo-use-case-skills

> **交叉参考**：完整工具评估详见 「third-party-tools.md — 工程规范分发」。

---

## 生产部署

### 安全检查清单

- [ ] **API Keys** 存储在 `.env` 中，不写入配置文件

- [ ] **RBAC/权限** 已审查（尤其是 Kubernetes、Semgrep）

- [ ] **速率限制** 已了解（Linear GraphQL 复杂度、Vercel API）

- [ ] **已实现 API 故障的回退机制**

- [ ] **所有 MCP 服务器已启用监控和日志记录**

### 错误处理与可靠性

MCP 工具可能因多种原因失败，如何向 Claude 传达失败信息至关重要。该协议提供了专用机制：工具响应中的 `isError` 标志。

**isError 标志**

工具调用失败时，在响应中设置 `isError: true`，而不是抛出异常或返回虚假的成功结果。这样 Claude 就能知道调用失败，并决定下一步操作：重试、换其他方式，或将问题告知用户。

```JSON
{
  "content": [
    {
      "type": "text",
      "text": "Database connection refused: ECONNREFUSED 127.0.0.1:5432"
    }
  ],
  "isError": true
}

```

不设置 `isError: true` 时，Claude 可能将错误消息误认为数据，并在错误状态下继续执行。设置后，Claude 能理解该步骤失败，从而推理如何恢复。

**错误分类：四个类别**

不同类型的失败需要不同的恢复策略。围绕以下四个类别构建错误消息，有助于 Claude 选择正确的恢复动作：

| 类别 | 触发时机 | Claude 的预期响应 | 示例 |
|-|-|-|-|
| **暂时性** | 临时不可用、网络抖动、速率限制 | 延迟后重试 | `503 Service Unavailable`、超时 |
| **验证错误** | 输入有误——类型错误、缺少字段、格式错误 | 修正输入后立即重试 | `invalid date format: expected ISO8601` |
| **业务错误** | 输入正确但操作违反领域规则 | 上报或跳过 | `cannot delete: record has active dependencies` |
| **权限错误** | 调用方无授权 | 停止并向用户说明 | `403 Forbidden: insufficient scope` |

**实现模式**

在错误消息中包含类别信息，让 Claude 无需猜测即可采取行动：

```Python
def call_tool(params):
    try:
        result = execute(params)
        return {"content": [{"type": "text", "text": result}], "isError": False}
    except NetworkError as e:
        return {
            "content": [{"type": "text", "text": f"[transient] {e}. Retry in a few seconds."}],
            "isError": True
        }
    except ValidationError as e:
        return {
            "content": [{"type": "text", "text": f"[validation] {e}. Check the input format."}],
            "isError": True
        }
    except PermissionError as e:
        return {
            "content": [{"type": "text", "text": f"[permission] {e}. Cannot proceed without elevated access."}],
            "isError": True
        }
    except BusinessError as e:
        return {
            "content": [{"type": "text", "text": f"[business] {e}. Operation not permitted by domain rules."}],
            "isError": True
        }

```

只有暂时性错误才适合自动重试。验证错误应修正输入后重试，而不是盲目重试。业务错误和权限错误应停止执行并反馈给用户，而不是循环重试。

### 工具描述设计模式

工具描述是 MCP 服务器中影响最大的部分。Claude 依赖它来决定调用哪个工具——描述模糊或重叠会导致路由错误，这比其他任何设计失误都更可靠地触发问题。

**核心问题：描述重叠导致路由错误**

两个描述听起来相似的工具会产生歧义。Claude 会选择其中一个，但往往不一致，因为它在凭描述猜测哪个适用。

```JSON
// 差——描述模糊，Claude 只能猜
{ "name": "analyze_content", "description": "Analyzes content" }
{ "name": "analyze_document", "description": "Analyzes document content" }

// 好——每个描述明确界定特定输入类型
{ "name": "analyze_content", "description": "Analyzes raw text strings or inline content (not files). Use for clipboard content, API responses, or text passed directly as a string." }
{ "name": "analyze_document", "description": "Analyzes content from a file path or URL. Use when the content lives on disk or at a remote endpoint, not when you already have the text in memory." }

```

检验标准：只看描述，你能否明确知道什么情况下**不该**使用这个工具？如果不能，就补充边界说明。

**描述的结构**

一个好的工具描述由三部分组成，按顺序排列：

1. **它做什么** — 一句话，用现在时、动词开头
2. **它接受什么** — 关键输入类型或约束（文件路径还是字符串、单个还是批量）
3. **什么时候用它而不是类似工具** — 明确说明决策边界

```JSON
{
  "name": "search_codebase",
  "description": "Searches source code files by regex pattern across the repository. Use for finding symbol definitions, function calls, and string literals in code. Not for searching documentation, configs, or prose — use search_docs for those."
}

```

**防止路由错误的命名规范**

| 模式 | 示例对比 | 原因 |
|-|-|-|
| 动词区分意图 | `get_user` vs `search_users` | 按已知 ID 获取 vs 按条件查找 |
| 名词区分输入类型 | `analyze_file` vs `analyze_text` | 磁盘上的路径 vs 内联字符串 |
| 范围后缀 | `list_tickets` vs `list_project_tickets` | 全局 vs 限定范围 |
| 操作粒度 | `create_record` vs `bulk_create_records` | 单条 vs 批量 |

避免将同义词用作工具名——`fetch`、`get`、`retrieve` 对 Claude 来说意思相同。每个语义操作只选一个动词系列。

**需避免的反模式**

- **缺乏范围的通用动词**：`process`、`handle`、`manage` 对 Claude 来说毫无信息量
- **缺少边界**："搜索数据库"——哪个数据库？全表？某个特定表？
- **会改变语义的布尔标志**：行为完全不同的工具应拆分为两个
- **描述超过 3 句话**：如果需要更多，说明工具职责过多

**input_examples 作为补充**

当 schema 不足以表达哪些参数组合有效或常见时，可添加 `input_examples`（Anthropic API 自 2026 年 2 月起支持）。它们为 Claude 展示具体用法，尤其适合可选参数：

```JSON
{
  "name": "create_ticket",
  "input_examples": [
    { "title": "Login page 500 error", "priority": "critical", "assignee": "oncall" },
    { "title": "Add dark mode toggle", "priority": "low" },
    { "title": "Update API docs for v2.1" }
  ]
}

```

示例能传达描述无法表达的内容：`assignee` 只在紧急情况下设置，`priority` 对于常规任务可以省略。

---

## 高级 MCP 工具设计

除基本错误分类外，三个设计决策会显著影响 Claude 在生产环境中使用 MCP 工具的效果：错误响应语义、资源与工具的区别，以及工具命名。

---

### isRetryable：应用层约定

MCP 规范的错误响应 schema 中不包含 `isRetryable` 字段。然而，在 `structuredContent` 中嵌入重试指导已成为调用易失败外部服务的工具的一种实用模式。

```JSON
{
    "isError": true,
    "content": [
        {
            "type": "text",
            "text": "Database query timed out after 30s. Retrying with the same parameters is likely to succeed."
        }
    ],
    "structuredContent": {
        "error": {
            "code": "DATABASE_TIMEOUT",
            "message": "Query timed out",
            "isRetryable": true,
            "retryAfterMs": 5000,
            "suggestedAction": "Retry the same query after 5 seconds"
        }
    }
}

```

不可重试的错误：

```JSON
{
    "isError": true,
    "content": [
        {
            "type": "text",
            "text": "Record not found. No record with ID 'usr_99999' exists in the database."
        }
    ],
    "structuredContent": {
        "error": {
            "code": "RECORD_NOT_FOUND",
            "message": "No record found for ID: usr_99999",
            "isRetryable": false,
            "suggestedAction": "Verify the ID is correct before retrying"
        }
    }
}

```

`isRetryable` 标志不是 Claude 从 MCP 规范中原生读取的字段，而是由你的编排层读取，由它决定是重试还是上报。该模式之所以有效，是因为 `structuredContent` 是机器可读的，你的代码可以在 Claude 处理之前先行检查。

---

### isError: false + 空内容 vs isError: true

这两种响应形式的语义完全不同，混淆它们会导致难以调试的静默失败。

| 响应 | 含义 |
|-|-|
| `isError: false` + 非空内容 | 工具执行成功，返回结果 |
| `isError: false` + 空内容 | 工具执行成功，未找到任何结果（合法的空状态） |
| `isError: true` | 工具失败：操作无法完成 |

```JSON
// 搜索无结果：不是错误
{
    "isError": false,
    "content": [
        {
            "type": "text",
            "text": "No documents found matching query: 'quarterly report Q5 2024'"
        }
    ]
}

// 搜索执行失败：是错误
{
    "isError": true,
    "content": [
        {
            "type": "text",
            "text": "Search service unavailable. Could not execute query."
        }
    ]
}

```

搜索返回零结果时，Claude 应告知用户并尝试不同的关键词。搜索执行失败时，Claude 应报告工具故障，编排层应考虑重试或上报。这两条路径应当分开，而只有正确的错误语义才能让它们正确分开。

---

### MCP 资源与工具

资源和工具服务于不同目的，由不同的参与方控制。混淆两者会导致工具无法被索引，而资源无法接受参数。

| 维度 | 资源 | 工具 |
|-|-|-|
| 访问控制方 | 应用程序（预定义，非模型发起） | 模型（在对话过程中按需调用） |
| 参数 | 无（通过 URI 读取） | 完整参数 schema |
| 使用场景 | 只读数据目录：配置文件、参考数据、文档 | 参数化操作：搜索、计算、写入、API 调用 |
| 发现方式 | 启动时列出，可浏览 | 在系统提示中描述，按需调用 |
| 副作用 | 无（按惯例只读） | 允许 |
| 示例 | 公司政策文档 | `search_policy_documents(query, date_range)` |

```Python
# 资源：静态参考数据，由应用程序控制
@server.list_resources()
async def list_resources():
    return [
        Resource(
            uri="config://database/schema",
            name="Database Schema",
            description="Current production database schema",
            mimeType="application/json"
        ),
        Resource(
            uri="docs://api/reference",
            name="API Reference",
            description="Internal API documentation",
            mimeType="text/markdown"
        )
    ]

@server.read_resource()
async def read_resource(uri: str) -> str:
    if uri == "config://database/schema":
        return json.dumps(get_current_schema())
    elif uri == "docs://api/reference":
        return read_file("docs/api-reference.md")
    raise ValueError(f"Unknown resource: {uri}")

# 工具：参数化操作，由模型控制
@server.list_tools()
async def list_tools():
    return [
        Tool(
            name="search_database",
            description="Search the database with a structured query",
            inputSchema={
                "type": "object",
                "properties": {
                    "query": {"type": "string"},
                    "table": {"type": "string"},
                    "limit": {"type": "integer", "default": 10}
                },
                "required": ["query", "table"]
            }
        )
    ]

```

**ResourceLink 桥接：** 当工具返回资源引用（而非内联内容）时，使用 ResourceLink：

```JSON
{
    "isError": false,
    "content": [
        {
            "type": "resource",
            "resource": {
                "uri": "docs://reports/Q3-2024",
                "mimeType": "application/pdf",
                "text": "Q3 2024 Financial Report (use read_resource to access full content)"
            }
        }
    ]
}

```

---

### 工具命名与系统提示冲突

工具名称若与系统提示中的关键词相同，会导致 Claude 将该工具与无关指令关联起来。名为 `process` 的工具会被 Claude 与系统提示中所有出现"process"一词的地方关联，产生不可预测的触发模式。

工具命名规则：

- 使用具体的复合名称：`search_customer_records` 而非 `search`
- 避免系统提示中常见的通用动词：`run`、`process`、`execute`、`handle`、`manage`
- 使用下划线，不用驼峰命名或连字符（MCP 约定）
- 工具较多时加领域前缀：`crm_get_contact`、`crm_update_contact`、`crm_search`

```Python
# 差：通用名称与系统提示关键词冲突
tools = ["search", "process", "run", "execute", "get", "update"]

# 好：带领域前缀的具体复合名称
tools = [
    "crm_search_contacts",
    "crm_get_contact_by_id",
    "crm_update_contact_status",
    "billing_create_invoice",
    "billing_get_invoice_status"
]

```

---

### 任务范围工具配置

向每次智能体调用提供所有可用工具既低效，又增加了只读阶段发生意外写操作的风险。任务范围工具配置根据当前任务阶段限制可用工具。

```Python
TOOL_PROFILES = {
    "exploration": [
        "search_documents",
        "get_document_by_id",
        "list_categories",
        "read_config"
    ],
    "analysis": [
        "search_documents",
        "get_document_by_id",
        "calculate_metrics",
        "compare_versions"
    ],
    "execution": [
        "create_document",
        "update_document",
        "delete_document",
        "send_notification"
    ]
}

def get_tools_for_phase(phase: str) -> list[str]:
    return TOOL_PROFILES.get(phase, TOOL_PROFILES["exploration"])

```

探索配置是只读的，执行配置增加了写操作。由于工具在分析阶段根本不存在，Claude 无法意外调用 `delete_document`。

对于不同用户角色可访问不同工具的多角色系统，应在角色层面进行范围控制，而不是在调用后过滤：

```Python
ROLE_TOOL_ACCESS = {
    "viewer": ["search_documents", "get_document_by_id"],
    "editor": ["search_documents", "get_document_by_id", "create_document", "update_document"],
    "admin": ["*"]  # 所有工具
}

def get_tools_for_role(role: str, all_tools: list) -> list:
    if role == "admin":
        return all_tools
    allowed = ROLE_TOOL_ACCESS.get(role, [])
    return [t for t in all_tools if t.name in allowed]

```

范围访问对 `verify_fact` 工具模式尤其有价值：只需验证单个断言的子智能体可以只获得 `verify_fact`，从而降低延迟（上下文中需要描述的工具更少）和风险（范围内没有写工具）。

---

### 快速入门套件

**最小可行方案（核心）**：

1. **Playwright MCP** — E2E 测试、网页验证
2. **Semgrep MCP** — 安全优先编码

**重要补充**：

1. **Context7 MCP** — API 参考准确性
2. **Linear MCP**（可选）— 问题追踪集成

**DevOps/SRE 套件**：

1. **Kubernetes MCP** — 集群管理
2. **Vercel MCP** — Next.js 部署自动化

**复杂环境**：

1. **MCP-Compose** — 多服务器编排
2. **Browserbase MCP** — 重度网页自动化（付费）

### 安装示例

```Bash
# Playwright（浏览器测试）
npm install @microsoft/playwright-mcp

# Semgrep（安全）
uvx semgrep-mcp

# Context7（文档）
npx -y @upstash/context7-mcp --api-key YOUR_API_KEY

# Linear（项目管理）
npm install mcp-linear

```

### 性能指标

| 指标 | 中位数 | 范围 | 备注 |
|-|-|-|-|
| **响应时间** | 约 200ms | 100–500ms | 取决于云端（Browserbase 约 500ms） |
| **Token 开销** | 约 200–500 Token | 结构化输出较少 | 可访问性树 vs 截图 |
| **配置时间** | 约 5 分钟 | 2–10 分钟 | Cargo 编译（MCP-Compose）约 10 分钟 |

---

## 月度跟踪方法论

本节记录维护本指南并进行月度生态更新的流程。

### 监控来源

**官方来源**：

- [Anthropic MCP GitHub](https://github.com/modelcontextprotocol/servers)
- [Anthropic 博客](https://www.anthropic.com/news)
- [MCP 协议规范](https://modelcontextprotocol.io)

**社区来源**：

- [GitHub 话题：mcp-servers](https://github.com/topics/mcp-servers)（7260+ 个服务器）
- [Awesome MCP Servers](https://github.com/punkpeye/awesome-mcp-servers)（75.5k 星）
- [MCP 注册表](https://github.blog/ai-and-ml/generative-ai/how-to-find-install-and-manage-mcp-servers-with-the-github-mcp-registry/)

**讨论社区**：

- [Reddit r/ClaudeAI](https://www.reddit.com/r/ClaudeAI/)
- [Reddit r/mcp](https://www.reddit.com/r/mcp/)
- [X/Twitter #MCPServer](https://twitter.com/search?q=#MCPServer)

**技术文章**：

- [Blog Skyvia](https://blog.skyvia.com/best-mcp-servers/)
- [Builder.io 博客](https://www.builder.io/blog/best-mcp-servers-2026)
- [Cyberpress](https://cyberpress.org/best-mcp-servers/)

### 月度审查清单

- [ ] **官方服务器**：检查 Anthropic GitHub 的新版本发布

- [ ] **社区服务器**：查看 GitHub 话题中的热门服务器（≥50 星，发布时间 <3 个月）

- [ ] **生态变化**：关注 Anthropic 博客的协议更新

- [ ] **服务器健康**：重新评估现有服务器（版本发布、问题、维护情况）

- [ ] **安全**：检查已披露的漏洞（GitHub 安全公告）

- [ ] **废弃情况**：识别已归档或无人维护的服务器

- [ ] **更新指南**：添加新验证的服务器，移除已废弃的服务器

### 评估模板

对每个候选服务器：

1. **基础验证**：

   - GitHub 星数 ≥50？
   - 最近发布时间 <3 个月？
   - 文档完整（README + 示例 + 配置）？
   - 是否有测试/CI？
2. **质量评分**（参见评估框架）：

   - 维护活跃度：`/10`
   - 文档质量：`/10`
   - 测试覆盖：`/10`
   - 性能：`/10`
   - 采用率：`/10`
   - **总分**：`/50` → 归一化为 `/10`
3. **使用场景分析**：

   - 填补了什么空白？
   - 是否已被官方服务器覆盖？
   - 有哪些替代方案？
4. **决策**：

   - **收录**（评分 ≥8）：在指南中添加完整章节
   - **观察**（评分 6–7）：添加至「观察列表」，下月重新评估
   - **拒绝**（评分 <6）：在已排除服务器中记录原因

### 集成工作流程

新增服务器时：

1. 在对应类别下创建章节（浏览器自动化、DevOps 等）
2. 包含以下内容：

   - 使用场景描述
   - 核心功能表格
   - 安装说明
   - 配置示例
   - 质量评分
   - 限制与解决方案
   - 替代方案对比
   - 资源（GitHub、文档、教程）
3. 若与最小可行方案相关，更新快速入门套件
4. 若涉及安全关键内容，更新生产部署清单

---

## 为 Claude 记录 MCP：参考文件模式

将 MCP 服务器集成到技能中时，Claude 必须自行摸索查询语法、必填参数组合和奇怪的行为。对于简单的 MCP 来说这没问题，但对于面向生产环境的工具（可观测性工具、项目管理 API、日志聚合器），这很快就会出问题。Claude 会猜参数格式，收到晦涩的错误，换种方式再试，最终把预算都浪费在无效重试上。

来自 Packmind 工程团队（Apache 2.0 开源）的解决方案：在技能目录下添加 `references/<mcp-name>.md` 文件，并在技能的第一步读取它，然后再进行任何 MCP 调用。

### 参考文件包含的内容

Claude 无法可靠推断的三类内容：

**1. 与工具名称不符的参数语义**

例如，使用 Datadog 查询语法（而非正则表达式）的 Datadog "search" 工具；或需要 `organization_slug`（URL slug，而非显示名称）的 Sentry 工具。这些不是 MCP 的 Bug，只是不够直观。

**2. 已知错误模式及其触发原因**

例如："在 DDSQL 的 `GROUP BY` 中使用 `SELECT` 别名会导致晦涩错误，请改用完整表达式。"这将一个 10 分钟的调试过程变成了零秒的查阅。

**3. 覆盖 80% 场景的可用查询示例**

提供覆盖最常见查询的可复制粘贴示例，让 Claude 直接修改复用，而不是从头构建。

### 文件结构

```Plain Text
.claude/skills/my-mcp-skill/
├── SKILL.md                    # 主技能文件
└── references/
    └── <mcp-name>.md           # MCP 参考文件（此模式）

```

SKILL.md 在第一步读取参考文件：

```Markdown
## 第一步：读取 MCP 参考文件

在做任何其他事情之前，先读取 `references/<mcp-name>.md`。
其中包含该 MCP 的查询语法和已知注意事项。
不得跳过此步骤。

```

### 为何有效

参考文件不是写给人看的文档，而是结构化的上下文注入。其中的每条信息都能降低 Claude 发出错误 MCP 调用的概率。做得好的话，它能消除因语法错误引起的重试循环，使技能可靠到足以无人值守地定时运行。

这个模式适用于任何行为不够直观的 MCP：Datadog、Sentry、PagerDuty、Linear、Jira、Mixpanel、Posthog。如果 MCP 有查询语言、分页细节或名称不直观的必填参数，参考文件在第一次运行时就能证明其价值。

### 开箱即用模板

演示此模式的完整模板技能（含 Sentry 示例）位于：

本仓库的 `examples/skills/mcp-integration-reference/`

模板包含：

- 具有 5 步结构的 `SKILL.md`（读取参考、收集范围、获取、分析、报告）
- 包含完整参数文档、注意事项、查询示例和噪音排除列表的 `references/sentry-mcp.md`
- 适配任意 MCP 服务器的操作说明

> 灵感来源于 [Packmind 开源仓库](https://github.com/packmind/packmind)（Apache 2.0，Cédric Teyton）的 Datadog MCP 参考文件。完整致谢见 「Credits」。

---

## 已排除服务器

经过评估但未纳入验证列表的服务器：

| 服务器 | 排除原因 | 来源 | 评估日期 |
|-|-|-|-|
| **X/Twitter MCP** | API 不稳定，认证问题频发，维护不一致 | [Cursor 论坛](https://forum.cursor.com/t/linear-mcp-commonly-errors-out-and-requires-turning-off-then-on/148816) | 2026 年 1 月 |
| **Vector Search MCP** | 星数 <50，文档不完整 | [LobeHub](https://lobehub.com/mcp/hugoduncan-mcp-vector-search) | 2026 年 1 月 |
| **GitHub MCP** | 已归档，迁移至官方 Go SDK | [GitHub 变更日志](https://github.blog/changelog/2025-12-10-the-github-mcp-server-adds-support-for-tool-specific-configuration-and-more/) | 2026 年 1 月 |
| **Jira MCP（sooperset）** | 近期无版本发布（最后更新：2025 年 6 月），稳定性不如 Linear | [GitHub 发布页](https://github.com/sooperset/mcp-atlassian/releases) | 2026 年 1 月 |

---

## 统计与洞察

### 按类别分布

| 类别 | 服务器数量 | 使用场景 |
|-|-|-|
| **浏览器自动化** | 3（Playwright、Browserbase、Chrome DevTools） | 测试、调试、数据提取 |
| **DevOps/基础设施** | 3（Vercel、Kubernetes、Sentry） | 部署、集群管理、可观测性 |
| **安全/代码分析** | 1（Semgrep） | 漏洞扫描、安全编码 |
| **代码搜索/分析** | 1（Grepai） | 语义搜索、调用图谱分析 |
| **文档/知识** | 1（Context7） | API 参考、代码示例 |
| **项目管理** | 1（Linear） | 问题追踪、Sprint 规划 |
| **编排** | 1（MCP-Compose） | 多服务器管理 |

### 维护方类型

- **官方服务器**（6 个）：Playwright（微软）、Browserbase、Semgrep、Context7、Kubernetes（红帽）、Chrome DevTools（Anthropic）
- **社区服务器**（4 个）：Linear、Vercel、MCP-Compose、Grepai（设计良好，积极维护）

---

**最后更新**：2026 年 5 月**下次审查**：2026 年 6 月**维护方**：Claude Code 《Claude Code：从入门到大师》团队

---

*返回 「主指南」 | 「README」*

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/X6iXwqTstif65GkyYg4cK9ztnsf> ｜ 归档：2026-06-04
