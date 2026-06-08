# GitHub Actions 集成

通过将 Claude 直接连接到你的 GitHub 工作流，自动化代码审查、Issue 分类和质量门控。两种触发模式：`@claude` 提及（人工发起）和定时/事件自动化（完全自主）。

---

## 目录

1. TL;DR
2. 两种模式
3. 配置
4. 模式 1：@claude 提及时的 PR 代码审查
5. 模式 2：推送时自动 PR 审查
6. 模式 3：Issue 分类和打标签
7. 模式 4：安全专项审查
8. 模式 5：定时仓库维护
9. 身份验证替代方案
10. 成本控制
11. 安全检查清单
12. 延伸阅读

---

## TL;DR

```YAML
# 最小可用示例——粘贴到 .github/workflows/claude.yml
name: Claude Code Review
on:
  issue_comment:
    types: [created]

jobs:
  claude:
    if: contains(github.event.comment.body, '@claude')
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      issues: write
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}

```

在任何 PR 上评论 `@claude review this PR` → Claude 读取差异对比（diff）并发布审查。

---

## 两种模式

| 模式 | 触发方式 | 使用场景 |
|-|-|-|
| **交互式** | PR/Issue 评论中的 `@claude` 提及 | 按需审查、提问、修复 |
| **自动化** | 推送、PR 创建、定时、标签 | 持续质量门控、分类 |

两者使用相同的 action——区别在于 `on:` 块以及是否包含 `if:` 条件。

---

## 配置

### 快速开始（30 秒）

在你的 Claude Code 终端中，进入任何已连接 GitHub 仓库的项目：

```Plain Text
/install-github-app

```

这将引导你完成创建 GitHub App、将 `ANTHROPIC_API_KEY` 添加到仓库 secrets，以及生成基础 `claude.yml` 工作流。

### 手动配置

1. 将 `ANTHROPIC_API_KEY` 添加到你的 GitHub 仓库 secrets
2. 创建 `.github/workflows/claude.yml`（参见下面的模式）
3. 授予工作流权限：`contents: write`、`pull-requests: write`、`issues: write`

---

## 模式 1：@claude 提及时的 PR 代码审查

人工发起。开发者评论 `@claude review this PR`，Claude 内联回复。

```YAML
# .github/workflows/claude-review.yml
name: Claude Interactive Review
on:
  issue_comment:
    types: [created, edited]
  pull_request_review_comment:
    types: [created]

jobs:
  claude:
    if: |
      contains(github.event.comment.body, '@claude') ||
      contains(github.event.review_comment.body, '@claude')
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      issues: write
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          claude_env: |
            GITHUB_TOKEN=${{ secrets.GITHUB_TOKEN }}

```

**使用示例：**

- `@claude review this PR` — 完整差异分析并提出建议
- `@claude is this change backwards compatible?` — 针对性问题
- `@claude fix the failing test in src/auth.test.ts` — Claude 开启一个跟进 PR 包含修复

---

## 模式 2：推送时自动 PR 审查

PR 一旦创建或更新就立即获得审查，无需提及。

```YAML
# .github/workflows/claude-auto-review.yml
name: Claude Auto PR Review
on:
  pull_request:
    types: [opened, synchronize]
    # 可选：仅在特定路径触发
    # paths:
    #   - 'src/**'
    #   - '!**/*.md'

jobs:
  claude-review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            审查这个 pull request。重点关注：
            - 逻辑错误和边缘情况
            - 安全问题（注入、认证、secrets）
            - 性能回退
            - 缺失的错误处理

            格式如下：
            ## 摘要
            用一段话描述变更。

            ## 发现的问题
            编号列表，严重程度（严重/主要/次要），文件:行号引用。

            ## 建议
            可选的改进建议。

            控制在 400 字以内。要直接。

```

**提示**：添加 `paths:` 避免在仅文档的 PR 上触发，或添加 `if: github.event.pull_request.draft == false` 跳过草稿 PR。

---

## 模式 3：Issue 分类和打标签

Claude 读取新 Issue，分配标签，并发布结构化分类评论。

```YAML
# .github/workflows/claude-triage.yml
name: Issue Triage
on:
  issues:
    types: [opened]

jobs:
  triage:
    runs-on: ubuntu-latest
    permissions:
      issues: write
      contents: read
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            对这个 GitHub Issue 进行分类：

            1. 从以下选项中分配一个标签：bug、enhancement、question、documentation、performance、security
            2. 分配优先级标签：priority:critical、priority:high、priority:medium、priority:low
            3. 发布评论，包含：
               - Issue 类型分类
               - 可能受影响的组件（基于 Issue 描述）
               - 对报告者的下一步建议（需要复现步骤？缺少版本信息？）

            要简洁。每点一句话。

```

---

## 模式 4：安全专项审查

专门针对涉及敏感路径（认证、支付、配置）的 PR 运行。

```YAML
# .github/workflows/claude-security.yml
name: Security Review
on:
  pull_request:
    paths:
      - 'src/auth/**'
      - 'src/payments/**'
      - '**/config/**'
      - '**/.env*'
      - '**/secrets/**'

jobs:
  security-review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            对这个 PR 进行安全专项审查。检查：

            - 注入漏洞（SQL、命令、LDAP）
            - 认证和授权绕过
            - 代码或注释中的 secrets 或凭证
            - 不安全的直接对象引用
            - 缺失的输入验证
            - 不安全的反序列化
            - OWASP Top 10 模式

            评定整体风险：低 / 中 / 高 / 严重。
            如果为高或严重，添加标签 'security-review-required'。
            列出每个发现，包含：文件:行号、漏洞类型和建议修复方案。

```

---

## 模式 5：定时仓库维护

每周健康检查——无需任何人工触发即可运行。

```YAML
# .github/workflows/claude-maintenance.yml
name: Weekly Repo Health Check
on:
  schedule:
    - cron: '0 9 * * 1'  # 每周一 UTC 上午 9 点
  workflow_dispatch:       # 也允许手动触发

jobs:
  maintenance:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      issues: write
    steps:
      - uses: actions/checkout@v4

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            执行每周仓库健康检查：

            1. 扫描 package.json（或等效文件）中的过时主要依赖
            2. 检查 src/ 中超过 30 天的 TODO/FIXME 注释
            3. 识别没有对应实现文件的测试文件
            4. 列出引用已删除或重命名文件的文档文件

            创建一个标题为「Weekly Health Check - [日期]」的 GitHub Issue 记录发现。
            如果没有需要关注的内容，发布评论「Health check passed — no issues found.」

```

---

## 身份验证替代方案

以上示例直接使用 `ANTHROPIC_API_KEY`。对于使用云提供商的团队：

**Amazon Bedrock：**

```YAML
- uses: anthropics/claude-code-action@v1
  with:
    use_bedrock: 'true'
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    AWS_REGION: us-east-1
    ANTHROPIC_MODEL: 'anthropic.claude-3-5-sonnet-20241022-v2:0'

```

**Google Vertex AI：**

```YAML
- uses: anthropics/claude-code-action@v1
  with:
    use_vertex: 'true'
  env:
    ANTHROPIC_VERTEX_PROJECT_ID: ${{ secrets.GCP_PROJECT_ID }}
    CLOUD_ML_REGION: us-east5
    ANTHROPIC_MODEL: 'claude-3-5-sonnet-v2@20241022'

```

云提供商得益于数据驻留合规，可以利用现有 IAM 策略，而无需管理单独的 API 密钥。

---

## 成本控制

自动化工作流无需人工参与即可运行——请设置明确限制。

```YAML
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    # 限制每次工作流运行的支出
    claude_args: '--max-budget-usd 0.50'
    # 分类用 Haiku，审查用 Sonnet——不要默认 Opus
    prompt: |
      ...

```

**按模式的预算参考：**

| 模式 | 模型 | 每次运行约费用 |
|-|-|-|
| PR 审查（中等 PR） | Sonnet | $0.05–0.15 |
| Issue 分类 | Haiku | $0.01–0.03 |
| 安全审查（大型 PR） | Sonnet | $0.10–0.25 |
| 定时维护 | Sonnet | $0.05–0.20 |

使用 `ccusage` 或 Anthropic Console 用量仪表盘监控实际支出。

**防止费用失控：**

- 使用 `paths:` 过滤器避免在不相关的变更上触发
- 添加 `if: github.event.pull_request.draft == false` 跳过草稿 PR
- 设置 `concurrency:` 防止同一 PR 并行运行

```YAML
jobs:
  claude-review:
    concurrency:
      group: claude-${{ github.event.pull_request.number }}
      cancel-in-progress: true

```

---

## 安全检查清单

在部署到团队仓库之前：

- [ ] `ANTHROPIC_API_KEY` 存储为 GitHub secret，绝不放在工作流 YAML 中

- [ ] 工作流权限最小化——除非需要写入，否则使用 `contents: read`

- [ ] 对于公共仓库：添加 `if: github.event.pull_request.head.repo.full_name == github.repository` 防止 fork PR 触发 API 调用

- [ ] 审查工作流公开发布的内容——Claude 的评论对所有贡献者可见

- [ ] 谨慎使用 `pull_request_target`——即使来自 fork 也以写权限运行

**Fork 安全模式（公共仓库）：**

```YAML
jobs:
  claude:
    # 仅在来自同一仓库（非 fork）的 PR 上运行
    if: github.event.pull_request.head.repo.full_name == github.repository

```

---

## 延伸阅读

- 第 9.3 节 CI/CD 集成 — 无头模式、Unix 管道、`--output-format json`
- 「生产安全」 — 自动化智能体的防护机制
- 「安全加固」 — MCP 和 webhook 安全
- [官方 action 文档](https://github.com/anthropics/claude-code-action) — 解决方案指南、迁移、云提供商
- [社区工作流蓝图](https://github.com/alirezarezvani/claude-code-github-workflow) — 8 个工作流 + 4 个自主智能体，适合进阶团队

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/Tdsxwnze1iEfyUkQ8tec0QyNnWf> ｜ 归档：2026-06-04
