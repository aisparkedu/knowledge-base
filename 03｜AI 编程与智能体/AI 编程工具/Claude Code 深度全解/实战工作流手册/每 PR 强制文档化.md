# 每 PR 强制文档化

一个三层强制执行模式，确保每个 PR 在编写时即完成文档化，而非在发布时补文档。

---

## 问题所在

单一的 `CHANGELOG.md` 文件在活跃团队中会失控。三个都在修改同一文件的开放功能分支，意味着每次合并都会产生冲突。有人解决冲突时丢掉了一行，发布说明在发布前就已经错了。

更深层的问题是时机。「在发布时记录」听起来合理，直到你盯着三周前合并的 PR #840，试图还原用户侧的变更内容。提交信息写的是 `fix session handling`。开发者在另一个时区。上下文已经消失。

没有 CI 门控的强制执行意味着 changelog 变成了催债工作。每次发布前有人要追着所有人，在时间压力下从 git log 里填空白。

解决方案：每个 PR 一个 YAML 片段，在实现时编写，由 CI 验证，发布时自动组装。

---

## 三层架构

这个系统有效，因为强制执行在三个独立层面发生。每层捕获不同的失败模式。

### 第一层：CLAUDE.md 工作流规则

第一层是每次会话加载到 Claude Code 上下文中的规则。它将完整的片段工作流编码化，这样当要求 Claude 创建 PR 时，Claude 可以自主完成它。

```Markdown
# git-workflow.md（通过 CLAUDE.md 加载）

## Changelog 片段——每个 PR 前必须执行

在创建 PR 之前，始终生成一个 changelog 片段。

### 步骤

1. **从 git diff 推断** — 分析 `git diff main...HEAD` 以确定：
   - `type`：feat | fix | perf | refactor | security | docs | chore
   - `scope`：受影响的功能区域（auth、sessions、api 等）
   - `title`：一行面向用户的摘要（<80 个字符）

2. **创建片段** — 运行 `pnpm changelog:add` 或直接写入：

```

changelog/fragments/{PR_NUMBER}-{slug}.yml

````Plain Text

3. **验证** — 运行 `pnpm changelog:validate changelog/fragments/{file}.yml`

4. **随 PR 一起提交** — 将片段包含在同一分支中

### 片段 schema

```yaml
pr: 886                    # 必须与文件名前缀匹配
type: fix                  # feat|fix|perf|refactor|security|docs|chore
scope: "visiochat"
title: "修复 SSE 竞态条件后的空聊天"   # <80 个字符
description: |             # 可选——解释对用户的影响，而非实现细节
SSE 工作计划在 AI 流完成之前触发，导致 ChatWrapper
以 0 条消息挂载。
breaking: false
migration: false           # 如果 PR 添加了 DB 迁移，设为 true

````

### 绕过

为没有用户影响的 PR 添加标签 `skip-changelog`（CI 配置、依赖更新、发布提交）。

````Plain Text

这条规则使 Claude Code 成为强制执行的参与者，而非仅仅是编码工具。当开发者说「帮我创建 PR」时，Claude 从 diff 推断片段内容，并在开启 PR 之前创建它。

### 第二层：UserPromptSubmit 钩子（行为检测）

第二层在意图变成行动之前拦截。当开发者输入表示 PR 创建意图的内容时，钩子检查是否提到了 changelog 片段。

```bash
# .claude/hooks/smart-suggest.sh（节选——Tier 0 强制执行）

# 检测到 PR 创建意图
if echo "$PROMPT_LC" | grep -qE '(create.*pr|open.*pr|make.*pr|pull.?request|push.*pr)'; then
    # 未提到片段 → 先重定向到创建步骤
    if ! echo "$PROMPT_LC" | grep -qE '(changelog|fragment|skip-changelog)'; then
        suggest "pnpm changelog:add" \
            "合并前必须执行——创建 changelog/fragments/{PR}-{slug}.yml"
    else
        # 已提到 → 正常建议 PR 命令
        suggest "/pr" "带结构化描述的 PR 创建"
    fi
fi

````

该钩子是 `UserPromptSubmit`：非阻塞，每次提示最多一个建议，无匹配时静默。它在 Claude Code 处理提示之前运行，因此开发者在 Claude 开始任何操作之前就能内联看到提醒。

条件逻辑（`如果 X 但没有 Y`）是这里的关键模式。它不是全面阻断——它根据上下文适配。如果开发者已经提到了片段，他们会得到正常建议。如果没有，他们会得到强制执行提醒。

**带 3 层架构的完整钩子**：examples/hooks/bash/smart-suggest.sh

### 第三层：CI 强制执行（GitHub Actions）

第三层是硬门控。两个独立的 job 在每个针对主分支的 PR 上运行。

**check-fragment job**：首先检查绕过标签（封闭列表），然后要求 `changelog/fragments/{PR_NUMBER}-*.yml` 存在并通过结构化验证。

```YAML
- name: 检查片段是否存在且有效
  env:
    PR_NUMBER: ${{ github.event.pull_request.number }}
    PR_LABELS: ${{ toJson(github.event.pull_request.labels.*.name) }}
  run: |
    SKIP_LABELS=("skip-changelog" "dependencies" "release" "chore: deps")
    for LABEL in "${SKIP_LABELS[@]}"; do
      if echo "$PR_LABELS" | grep -q "\"$LABEL\""; then
        echo "检测到绕过标签——不需要片段"
        exit 0
      fi
    done

    FRAGMENT=$(ls "changelog/fragments/${PR_NUMBER}-"*.yml 2>/dev/null | head -1)
    if [ -z "$FRAGMENT" ]; then
      echo "缺少片段。运行：pnpm changelog:add"
      exit 1
    fi

    pnpm tsx changelog/scripts/validate.ts "$FRAGMENT"

```

**check-migration-flag job**（独立运行，无法绕过）：用 `git diff --name-only --diff-filter=A` 检测新的 SQL 迁移文件。如果存在迁移但片段中的 `migration: false`，则失败。这个 job 无法通过标签绕过——带有 `skip-changelog` 标签但添加了迁移的 PR 仍然触发检查。

两个 job 设计上是独立的。PR 可以绕过片段创建（通过标签），但仍然会在迁移检查上失败。

---

## 发布时的片段组装

随着 PR 合并，片段积累在 `changelog/fragments/` 中。发布时，一个命令将它们组装成一个版本化的 CHANGELOG 章节。

```Bash
pnpm changelog:assemble --version 1.8.0 [--dry-run]

```

它的作用：

1. 读取所有 `changelog/fragments/*.yml`
2. 按固定顺序按类型分组（feat、fix、perf、refactor、security、docs、chore）
3. 将 `breaking: true` 条目整合到专用的 `🔨 Breaking Changes` 章节
4. 内联注释 `migration: true` 条目为 `⚠️ Migration DB.`
5. 替换 `CHANGELOG.md` 中的 `## [Next Release]` 占位符
6. 将片段归档到 `changelog/fragments/released/{version}/`

输出：

```Markdown
## [1.8.0] - 2026-03-15

### 🔨 Breaking Changes
- **移除旧版 Token 格式（#871）** — v1.6.0 之前签发的 Token 已失效。

### ✨ 新功能
- **添加实时在线状态指示器（#892）**

### 🔧 Bug 修复
- **修复 SSE 竞态条件后的空聊天（#886）** — SSE 工作计划在 AI 流完成之前
  触发，导致 ChatWrapper 以 0 条消息挂载。

```

---

## 为什么是 3 层而非 1 层

每层捕获不同的失败模式：

| 层级 | 捕获的失败 | 时机 |
|-|-|-|
| CLAUDE.md 规则 | Claude 忘记工作流 | 每次会话 |
| UserPromptSubmit 钩子 | 开发者不假思索地输入「帮我创建 PR」 | 提示词前 |
| CI 门控 | 片段被跳过或损坏 | 合并前 |

单一的 CI 门控捕获问题太晚——开发者在 PR 已经开启后必须切换上下文回来修复。钩子在意图发生时捕获它。CLAUDE.md 规则意味着当被赋予任务时 Claude 会自主处理它。

各层不会冲突，而是相互强化。看到钩子建议的开发者会运行 `pnpm changelog:add`。Claude 会遵循 CLAUDE.md 规则并验证输出。CI 在合并前确认一切。

---

## 采用这个模式

TypeScript 脚本（add、validate、assemble、audit）是 Méthode Aristote 技术栈特定的。三层强制执行模式不是——它适用于任何片段格式、任何 CI 系统、任何组装器。

**最小可行配置：**

1. **定义你的片段 schema**（YAML、JSON，适合你技术栈的格式）
2. **添加一条 CLAUDE.md 规则**，将创建工作流编码化，让 Claude 能自主处理
3. **添加一个 UserPromptSubmit 钩子**，使用 `如果 PR 意图但没有提到片段 → 建议` 模式
4. **添加一个 CI job**，在合并前检查片段是否存在

钩子模式可推广到任何必须执行的工作流步骤。将「changelog 片段」替换为「ADR」、「迁移标志」、「测试覆盖率检查」——条件检测逻辑是相同的。

---

## 相关内容

- 钩子示例：examples/hooks/bash/smart-suggest.sh
- 钩子文档：「UserPromptSubmit 钩子」（搜索「UserPromptSubmit」）
- 片段验证器和组装器脚本：在 Méthode Aristote 仓库中提供

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/AWgiwfl6SigPDwk5qUkceDmRnVb> ｜ 归档：2026-06-04
