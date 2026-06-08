# 企业 AI 治理

> **目标读者**：在团队中大规模部署 Claude Code 的技术负责人、工程经理、安全负责人。

> **范围**：组织层面治理（策略、审批流程、级别、合规）。个人开发者安全（注入防御、MCP 审查、CVE）请参见 「security-hardening.md」。6 条不可妥协的生产规则请参见 「production-safety.md」。

---

## 速览

**治理盲区**：Claude Code 安全文档涵盖了个人开发者应做的事情，但没有涵盖整个组织使用时的情况——50 名开发者、不同的风险画像、缺乏统一策略。

**本文涵盖内容**：

| 章节 | 提供的内容 |
|-|-|
| 本地 vs 共享 | 风险矩阵 + 决策框架 |
| 使用章程 | 精简模板，可直接改编 |
| MCP 治理 | 审批流程 + YAML 注册表 |
| 护栏级别 | 4 个预配置级别，可直接复制 settings.json |
| 规模化策略执行 | 发布、入职、CI/CD 门控 |
| 审计与合规 | SOC2/ISO27001 审计师实际关注的内容 |

---

## 1. 本地 vs 共享：治理分界

企业 AI 治理中最大的错误是对所有情况应用相同的规则。本地使用和共享使用具有根本不同的风险画像。

### 1.1 风险矩阵

| 维度 | 本地使用 | 共享使用 |
|-|-|-|
| **数据暴露** | 开发者自己的文件 | 客户数据、共享代码库、密钥 |
| **影响范围** | 单台机器 | 整个代码库、CI/CD、生产环境 |
| **责任归属** | 个人 | 团队/组织 |
| **可重现性** | 会话结束，历史消失 | 需要审计跟踪 |
| **合规范围** | 通常无 | SOC2、ISO27001、HIPAA（如适用） |
| **配置漂移** | 个人偏好 | 团队一致性至关重要 |

### 1.2 可以和不能控制的内容

**可以控制**（通过提交的配置）：

- 哪些 MCP 服务器已获批准（代码库中的 `settings.json`）
- Claude 可以使用哪些工具（`permissions.deny`）
- CLAUDE.md 中的项目规范
- 工具调用前后运行的 Hook（钩子）脚本
- 验证 AI 生成代码的 CI/CD 门控

**无法直接控制**：

- 开发者的个人 `~/.claude/settings.json`
- 他们在个人 API 密钥上使用的模型
- 他们在代码库之外的个人项目中的行为
- 会话间的记忆/会话内容

**实际含义**：将治理重点放在提交到代码库和部署在共享环境中的内容上。个人开发工作流是开发者自己的责任。

### 1.3 决策框架：何时需要治理

并非所有内容都需要严格治理，按比例应用控制措施。

```Plain Text
您在治理什么？
│
├─ 个人开发工作流（本地、临时代码）
│   └─ 最小化：CLAUDE.md 指导方针 + 基础钩子
│
├─ 团队代码库（共享代码库，非生产环境）
│   └─ 标准：共享 settings.json + MCP 注册表 + PR 门控
│
├─ 生产系统（面向客户、真实数据）
│   └─ 严格：完整级别配置 + 审批流程 + 审计日志
│
└─ 受监管环境（HIPAA、SOC2、PCI、金融）
    └─ 合规：以上所有 + 合规审计跟踪

```

---

## 2. AI 使用章程

使用章程回答了一个根本性问题："我们在公司使用 Claude Code 允许做什么？"没有它，每个团队的回答都不同，造成不一致的风险敞口。

这是精简版。完整章程（含法律注意事项）请参见 白皮书 #11：企业 AI 治理（待发布）。

### 2.1 精简章程模板

将以下内容复制到您组织的 `docs/ai-usage-charter.md` 并进行改编：

```Markdown
# AI 编码工具使用章程

**适用范围**：Claude Code（及任何 AI 编码助手）
**生效日期**：[日期]
**负责人**：工程负责人/CTO
**审查周期**：季度

---

## 已批准的工具

| 工具 | 范围 | 数据分类 |
|------|-------|---------------------|
| Claude Code（Pro/团队/企业版） | 所有开发工作 | 最高至机密级 |
| Claude Code（个人账户） | 仅限个人开发 | 仅公开/内部级别 |
| [其他已批准工具] | [范围] | [分类] |

---

## 数据分类规则

| 分类 | 示例 | 是否允许使用 Claude Code？ |
|----------------|----------|--------------------------|
| **公开** | 开源、公共文档 | 是，无限制 |
| **内部** | 内部工具、非敏感代码 | 是，标准配置 |
| **机密** | 内部商业机密、非受监管的 IP | 是，仅限企业版 |
| **受限** | 客户个人身份信息、PCI 卡数据、PHI、凭证 | 否——未经法律/合规审批，绝不纳入 AI 上下文 |

**硬性规定**：受限数据绝不进入 AI 上下文窗口。无论是提示词中、Claude 读取的文件中，还是示例中都不行。配置 `permissions.deny` 阻止访问受限文件。

---

## 已批准的使用场景

- 代码补全、审查、重构
- 测试生成
- 文档起草
- 调试和根因分析
- 架构分析（仅限内部系统）
- CLI 脚本和自动化

---

## 禁止的使用场景

- 处理支付卡数据（PCI 范围）
- 生成处理原始 PHI 的代码（未经安全审查）
- 未经人工审批的自主生产部署
- 在机密或更高级别数据上使用个人 AI 账户
- 在提示词中以客户数据为示例

---

## 审批权限

| 操作 | 审批人 |
|--------|---------|
| 向团队配置添加新 MCP 服务器 | 技术负责人 + 安全审查 |
| 在新项目中启用 Claude Code | 团队负责人 |
| 启用企业功能（零信任、SSO） | IT/安全团队 |
| 对任何章程规则的豁免 | 工程总监 |

---

## 合规义务

使用公司系统上的 Claude Code，即表示您同意：
1. 遵守本章程
2. 在 24 小时内向 security@[公司] 报告疑似数据泄露
3. 不绕过治理控制（钩子、权限拒绝规则）
4. 参与季度访问审查

---

**章程违规**：遵循标准纪律程序。首次：辅导。重复或严重：升级处理。

```

### 2.2 数据分类与 Claude Code 设置

将数据分类转化为实际配置：

```JSON
{
  "permissions": {
    "deny": [
      "Read(./**/*.pem)",
      "Read(./**/*.key)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(**/credentials*)",
      "Bash(cat .env*)",
      "Bash(printenv*)",
      "Bash(env)"
    ]
  }
}

```

```Markdown
<!-- CLAUDE.md — 数据处理规则 -->
## 数据处理

**绝不**读取、引用或在输出中包含以下内容：
- 匹配以下模式的文件：.env、*.pem、*.key、credentials.*、secrets/
- 客户个人身份信息字段（字段名为：email、phone、ssn、dob、card_*）
- 凭证或 API 密钥（即使已遮蔽/脱敏的示例）

如果在读取文件时遇到受限数据，请停止并告知用户。
在被明确告知跳过该内容之前，不要继续处理。

```

---

## 3. MCP 治理流程

个人 MCP 审查（5 分钟审计）已在 「security-hardening.md §1.1」 中介绍。本节涵盖组织工作流：新 MCP 如何在整个团队中获得批准、部署和监控。

### 3.1 审批流程

```Plain Text
开发者需要新 MCP
        │
        ▼
[1] 提交 MCP 申请
    - 名称、来源 URL、版本
    - 拟定使用场景
    - 数据范围（将访问什么？）
        │
        ▼
[2] 安全审查（技术负责人 + 可选安全团队）
    - 5 分钟 MCP 审计（参见 security-hardening.md）
    - 检查：Star 数 >50、近期提交、无危险标记
    - 检查：无 CVE（搜索 NVD + GitHub 安全公告）
    - 风险分类：低/中/高
        │
     ┌──┴──┐
   低风险  中/高风险
     │      │
     ▼      ▼
  批准    扩展审查
          （2 周沙箱试验）
          + 安全团队审批
        │
        ▼
[3] 添加至已批准注册表
    - 固定精确版本
    - 记录批准范围
    - 设置到期日期（6 个月）
        │
        ▼
[4] 通过共享 settings.json 部署
    - 提交至代码库
    - 已批准的 MCP 不允许本地覆盖
        │
        ▼
[5] 监控 + 定期重审
    - 每 30 天检查安全公告
    - 版本升级时重新批准（补丁版本：自动；次版本+：手动）
    - 季度全面注册表审查

```

### 3.2 MCP 注册表格式

在您团队的共享配置库中维护已批准的 MCP 注册表 `.claude/mcp-registry.yaml`：

```YAML
# .claude/mcp-registry.yaml
# [组织名称] 已批准的 MCP 服务器
# 最后更新：2026-03-10
# 审查人：[姓名、职位]

metadata:
  review_cycle: quarterly
  next_review: "2026-06-10"
  owner: "platform-team@company.com"

approved:
  - name: context7
    version: "1.2.3"
    source: "https://github.com/context7/mcp-server"
    approved_by: "john.doe@company.com"
    approved_date: "2026-01-15"
    expires: "2026-07-15"
    data_scope: PUBLIC
    risk: LOW
    rationale: "只读文档查询，无数据外发。"
    config:
      command: npx
      args: ["-y", "@context7/mcp-server@1.2.3"]

  - name: sequential-thinking
    version: "0.6.2"
    source: "https://github.com/modelcontextprotocol/servers"
    approved_by: "jane.smith@company.com"
    approved_date: "2026-01-15"
    expires: "2026-07-15"
    data_scope: INTERNAL
    risk: LOW
    rationale: "仅本地推理，无网络访问。"
    config:
      command: npx
      args: ["-y", "@modelcontextprotocol/server-sequential-thinking@0.6.2"]

  - name: internal-db-readonly
    version: "2.1.0"
    source: "internal"
    approved_by: "security@company.com"
    approved_date: "2026-02-01"
    expires: "2026-05-01"  # 较高风险，到期时间更短
    data_scope: CONFIDENTIAL
    risk: MEDIUM
    rationale: "只读副本访问，允许列表中无个人身份信息表。"
    restrictions:
      - "仅限只读凭证"
      - "不访问 users、payments 或 audit 表"
    config:
      command: npx
      args: ["-y", "@company/db-mcp@2.1.0"]

pending_review:
  - name: github-mcp
    requested_by: "dev@company.com"
    requested_date: "2026-03-05"
    use_case: "PR 自动化"
    status: under_review

denied:
  - name: browser-automation-mcp
    denied_date: "2026-02-10"
    reason: "全浏览器访问，无范围限制，风险过高。"

```

### 3.3 通过 Hook（钩子）强制执行注册表

使用治理 Hook（钩子）验证只有已批准的 MCP 在使用中。以下脚本是一个可放入 `.claude/hooks/governance-check.sh` 的精简内联版本。如需包含更多检查（拒绝列表强制执行、危险允许列表检测）的更完整实现，请参见 examples/hooks/bash/governance-enforcement-hook.sh。

```Bash
#!/bin/bash
# .claude/hooks/governance-check.sh
# 事件：SessionStart
# 验证活跃 MCP 配置是否在已批准的注册表中

REGISTRY=".claude/mcp-registry.yaml"
SETTINGS="${HOME}/.claude.json"

if [[ ! -f "$REGISTRY" ]]; then
  exit 0  # 无注册表 = 不强制执行（选择性治理）
fi

# 检查未批准的 MCP（需要 yq 和 jq）
if command -v jq &>/dev/null && command -v yq &>/dev/null; then
  ACTIVE=$(jq -r '.mcpServers | keys[]' "$SETTINGS" 2>/dev/null)
  APPROVED=$(yq e '.approved[].name' "$REGISTRY" 2>/dev/null)

  for mcp in $ACTIVE; do
    if ! echo "$APPROVED" | grep -q "^${mcp}$"; then
      echo "治理警告：MCP '${mcp}' 不在已批准注册表中。"
      echo "提交申请请访问：https://your-internal-wiki/mcp-requests"
      echo "会话继续——请在 48 小时内处理。"
    fi
  done
fi

exit 0

```

**注意**：此钩子发出警告，不会阻断。在会话启动时阻断会产生过多摩擦，请改用定期合规检查（参见 §5.3）。

---

## 4. 护栏级别

针对四种常见场景预配置的护栏级别。将相关级别复制到您项目的 `.claude/settings.json` 和 `CLAUDE.md` 中。

### 级别 1：入门级

**适用场景**：小型团队（<5 人）、内部项目、无生产数据、低合规要求。

```JSON
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./**/*.key)",
      "Read(./**/*.pem)"
    ]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": ["~/.claude/hooks/dangerous-actions-blocker.sh"]
      }
    ]
  }
}

```

```Markdown
<!-- CLAUDE.md 补充内容——入门级 -->
## 安全基础
- 绝不读取 .env 文件或凭证文件
- 运行破坏性命令前先询问（DROP、DELETE、rm -rf）
- 遵循代码库现有模式

```

**投入**：10 分钟配置，覆盖基础内容。

### 级别 2：标准级

**适用场景**：5–20 人团队、接近生产的代码、部分敏感数据、无强制合规要求。

{"permissions": {"deny": ["Read(./.env)","Read(./.env.*)","Read(./**/*.key)","Read(./**/*.pem)","Read(./secrets/**)","Bash(cat .env*)","Bash(printenv*)","Edit(docker-compose.yml)","Edit(.github/workflows/**)","Edit(terraform/**)"]},"hooks": {"PreToolUse": [{"matcher": "Bash","hooks": ["~/.claude/hooks/dangerous-actions-blocker.sh"]},{"matcher": "Edit|Write","hooks": [".claude/hooks/prompt-injection-detector.sh"]}],"PostToolUse": [{"matcher": "Bash","hooks": ["~/.claude/hooks/output-secrets-scanner.sh"]}],"SessionStart": [".claude/hooks/governance-check.sh"]}}

````Plain Text

```markdown
<!-- CLAUDE.md 补充内容——标准级 -->
## 生产安全
- 基础设施文件（docker-compose、terraform、CI/CD）已锁定。
  修改前需获得权限。
- 新依赖需要技术负责人审批，不要运行 npm install <pkg>。
- 数据库破坏性操作（DROP、DELETE、TRUNCATE）需要备份确认。

## 代码审查门控
- 所有涉及 auth、支付或数据访问的 AI 生成代码，必须在 PR 描述中
  标注"AI 生成：需要审查"注释。

````

**投入**：30–45 分钟配置，覆盖大多数团队需求。

### 级别 3：严格级

**适用场景**：20+ 人团队、生产关键系统、客户数据、非正式合规预期。

```JSON
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./.env.local)",
      "Read(./**/*.key)",
      "Read(./**/*.pem)",
      "Read(./secrets/**)",
      "Read(**/credentials*)",
      "Bash(cat .env*)",
      "Bash(printenv*)",
      "Bash(env)",
      "Bash(npm install *)",
      "Bash(pnpm add *)",
      "Bash(pip install *)",
      "Edit(docker-compose.yml)",
      "Edit(docker-compose.prod.yml)",
      "Edit(.github/workflows/**)",
      "Edit(terraform/**)",
      "Edit(kubernetes/**)",
      "Edit(prisma/schema.prisma)"
    ]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          "~/.claude/hooks/dangerous-actions-blocker.sh",
          "~/.claude/hooks/velocity-governor.sh"
        ]
      },
      {
        "matcher": "Edit|Write",
        "hooks": [
          ".claude/hooks/prompt-injection-detector.sh",
          ".claude/hooks/unicode-injection-scanner.sh"
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": ["~/.claude/hooks/output-secrets-scanner.sh"]
      },
      {
        "matcher": "Edit|Write",
        "hooks": [".claude/hooks/session-logger.sh"]
      }
    ],
    "SessionStart": [
      ".claude/hooks/governance-check.sh",
      "~/.claude/hooks/mcp-config-integrity.sh"
    ]
  }
}

```

```Markdown
<!-- CLAUDE.md 补充内容——严格级 -->
## 安全态势：严格

您正在严格安全环境中运行。请无例外地遵守以下规则。

### 锁定文件
以下文件未经本次对话明确授权不可修改：
- docker-compose.yml、Dockerfile、.github/workflows/**、terraform/**、kubernetes/**
- prisma/schema.prisma（数据库架构）
- /src/auth/、/src/payments/、/src/crypto/ 中的任何文件

### 依赖协议
添加任何依赖前：
1. 说明依赖名称和用途
2. 列出至少 2 个考虑过的替代方案
3. 等待明确批准后再运行任何安装命令

### 数据访问协议
读取项目根目录以外的任何文件前：
1. 说明文件路径及需要它的原因
2. 如果路径看起来敏感，等待批准

### AI 归因
您在 PR 中生成的所有代码块必须以 `// AI 生成` 为前缀。
AI 生成的测试必须包含 `// AI 生成测试` 注释。

```

**投入**：1–2 小时配置，适合大多数生产团队。

### 级别 4：合规级

**适用场景**：金融、医疗、受监管行业。需要 HIPAA、SOC2、PCI、ISO27001 合规。

在严格级基础上增加合规特定控制。

```JSON
{
  "permissions": {
    "deny": [
      "Read(./.env*)",
      "Read(./**/*.key)",
      "Read(./**/*.pem)",
      "Read(./secrets/**)",
      "Read(**/credentials*)",
      "Read(**/patient*)",
      "Read(**/phi*)",
      "Read(**/pii*)",
      "Read(**/card*)",
      "Read(**/ssn*)",
      "Bash(cat .env*)",
      "Bash(printenv*)",
      "Bash(env)",
      "Bash(npm install *)",
      "Bash(pnpm add *)",
      "Bash(pip install *)",
      "Bash(curl *)",
      "Bash(wget *)",
      "Edit(docker-compose*.yml)",
      "Edit(.github/workflows/**)",
      "Edit(terraform/**)",
      "Edit(kubernetes/**)",
      "Edit(prisma/schema.prisma)",
      "Edit(**/auth/**)",
      "Edit(**/crypto/**)",
      "Edit(**/encryption/**)"
    ]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          "~/.claude/hooks/dangerous-actions-blocker.sh",
          "~/.claude/hooks/velocity-governor.sh"
        ]
      },
      {
        "matcher": "Edit|Write",
        "hooks": [
          ".claude/hooks/prompt-injection-detector.sh",
          ".claude/hooks/unicode-injection-scanner.sh"
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": ".*",
        "hooks": [
          "~/.claude/hooks/output-secrets-scanner.sh",
          ".claude/hooks/session-logger.sh"
        ]
      }
    ],
    "SessionStart": [
      ".claude/hooks/governance-check.sh",
      "~/.claude/hooks/mcp-config-integrity.sh"
    ]
  }
}

```

```Markdown
<!-- CLAUDE.md 补充内容——合规级 -->
## 合规模式：[HIPAA | SOC2 | PCI]——已激活

您正在法规合规要求下运行。以下规则不可商量。

### 禁止的数据
绝不在输出、建议或示例中包含：
- PHI（患者健康信息）、PII（客户场景下的姓名、邮箱、电话）
- 卡号、CVV、银行账户
- 身份证号、税号、政府 ID
- 原始认证令牌、会话 cookie、API 密钥

### 强制审查门控
以下变更在代码提交前需要人工审批：
- 对认证或授权逻辑的任何修改
- 对加密或密钥管理的任何修改
- 任何数据库迁移
- 任何新的外部 API 集成

### 审计跟踪
每个在受监管数据上运行的会话必须：
- 在会话开始时注明用户 ID（"本次会话用户：[您的邮箱]"）
- 在会话开始时说明任务描述（"任务：[简要描述]"）
- 在自然断点处添加检查点注释

### AI 归因（合规级强制要求）
所有 AI 生成的代码必须包含：
- `// AI 生成：[日期] [模型] [审查人]` 注释
- PR 描述必须包含 AI 披露部分

```

**受监管环境的附加工具**：考虑使用 Entire CLI 进行完整会话审计跟踪和审批门控。详情及评估清单请参见 「AI 可追溯性 §5.1」。这是多个可选工具之一，请根据您的具体合规需求进行评估。

---

## 5. 规模化策略执行

拥有策略和执行策略是两回事。本节介绍如何让治理在 10–100 名开发者的团队中真正落地。

### 5.1 配置分发

**核心原则**：治理配置放在代码库中，而非个人机器上。

```Plain Text
your-org-config/                 ← 独立的"平台配置"代码库
├── .claude/
│   ├── settings.json            ← 共享设置（基于级别）
│   ├── mcp-registry.yaml        ← 已批准的 MCP
│   ├── hooks/
│   │   ├── governance-check.sh  ← MCP 注册表检查
│   │   ├── session-logger.sh    ← 审计跟踪
│   │   └── velocity-governor.sh ← 速率限制
│   └── agents/
│       └── security-reviewer.md ← 代码审查共享智能体
├── templates/
│   ├── CLAUDE.md.starter        ← 各级别 CLAUDE.md 模板
│   ├── CLAUDE.md.standard
│   ├── CLAUDE.md.strict
│   └── CLAUDE.md.regulated
└── scripts/
    └── setup-project.sh         ← 用正确级别初始化新项目

```

**初始化新项目**：

```Bash
#!/bin/bash
# scripts/setup-project.sh
# 用法：./setup-project.sh [starter|standard|strict|regulated]

TIER=${1:-standard}
CONFIG_REPO="https://github.com/your-org/claude-code-config"

echo "设置 Claude Code 治理：$TIER 级"

# 创建 .claude 目录
mkdir -p .claude/hooks

# 复制级别配置
curl -s "$CONFIG_REPO/raw/main/templates/.claude/settings.${TIER}.json" \
  -o .claude/settings.json

# 复制 CLAUDE.md 模板
curl -s "$CONFIG_REPO/raw/main/templates/CLAUDE.md.${TIER}" \
  -o CLAUDE.md

# 复制治理钩子
curl -s "$CONFIG_REPO/raw/main/hooks/governance-check.sh" \
  -o .claude/hooks/governance-check.sh
chmod +x .claude/hooks/governance-check.sh

echo "完成。请将 .claude/ 和 CLAUDE.md 提交到您的代码库。"

```

### 5.2 入职检查清单

加入使用 Claude Code 团队的新开发者应完成此检查清单：

```Markdown
## Claude Code 入职检查清单

### 配置（30 分钟）
- [ ] 安装 Claude Code：`npm i -g @anthropic-ai/claude-code`
- [ ] 配置全局安全钩子：`./scripts/install-global-hooks.sh`
- [ ] 验证项目配置已加载：运行 `claude` 然后询问"这个项目是哪个级别？"
- [ ] 阅读 AI 使用章程（链接至您的文档）
- [ ] 查看已批准的 MCP 列表：`.claude/mcp-registry.yaml`

### 安全基础
- [ ] 确认 `~/.claude/settings.json` 中没有 `permissions.allow` 覆盖
  能绕过项目的拒绝规则
- [ ] 确认没有访问生产数据的个人 MCP 服务器在运行
- [ ] 知道如何报告数据泄露：security@[公司]

### 第一周
- [ ] 用 Claude Code 完成一项任务（修 bug、小功能）
- [ ] 提交至少一个带有适当 AI 归因部分的 PR
- [ ] 向技术负责人反馈任何摩擦点以改进配置

### 季度
- [ ] 参与 MCP 注册表审查
- [ ] 审查 AI 使用章程更新
- [ ] 确认没有个人配置覆盖项在运行

```

### 5.3 合规检查

自动化定期合规检查，以检测配置漂移：

```Bash
#!/bin/bash
# scripts/claude-governance-audit.sh
# 通过 CI/CD 或 cron 每周运行

PASS=0
FAIL=0
WARN=0

check() {
  local name="$1"
  local result="$2"
  local severity="${3:-FAIL}"

  if [[ "$result" == "OK" ]]; then
    echo "  通过：$name"
    ((PASS++))
  else
    echo "  $severity：$name — $result"
    [[ "$severity" == "FAIL" ]] && ((FAIL++)) || ((WARN++))
  fi
}

echo "=== Claude Code 治理审计 ==="
echo ""

# 检查：settings.json 是否存在且已提交
echo "1. 代码库配置"
[[ -f ".claude/settings.json" ]] \
  && check "settings.json 存在" "OK" \
  || check "settings.json 存在" "缺失——团队配置未强制执行" "FAIL"

git ls-files --error-unmatch .claude/settings.json &>/dev/null \
  && check "settings.json 已提交" "OK" \
  || check "settings.json 已提交" "未被 git 跟踪——不会应用于团队" "WARN"

# 检查：密钥的拒绝规则
echo ""
echo "2. 密钥保护"
if [[ -f ".claude/settings.json" ]]; then
  jq -e '.permissions.deny[]? | select(test("env|pem|key"))' \
    .claude/settings.json &>/dev/null \
    && check ".env 保护规则" "OK" \
    || check ".env 保护规则" "没有针对 .env 或密钥文件的拒绝规则" "FAIL"
fi

# 检查：钩子已安装且可执行
echo ""
echo "3. 钩子堆栈"
for hook in ".claude/hooks/governance-check.sh"; do
  if [[ -f "$hook" ]]; then
    [[ -x "$hook" ]] \
      && check "$hook 可执行" "OK" \
      || check "$hook 可执行" "不可执行——运行 chmod +x $hook" "FAIL"
  else
    check "$hook 存在" "缺失" "WARN"
  fi
done

# 检查：MCP 注册表是否存在
echo ""
echo "4. MCP 治理"
[[ -f ".claude/mcp-registry.yaml" ]] \
  && check "MCP 注册表存在" "OK" \
  || check "MCP 注册表存在" "无注册表——MCP 使用无治理" "WARN"

# 检查：CLAUDE.md 是否存在且已提交
echo ""
echo "5. 文档"
[[ -f "CLAUDE.md" ]] || [[ -f ".claude/CLAUDE.md" ]] \
  && check "CLAUDE.md 存在" "OK" \
  || check "CLAUDE.md 存在" "缺失——AI 没有项目上下文" "WARN"

echo ""
echo "=== 摘要 ==="
echo "  通过：   $PASS"
echo "  失败：   $FAIL（必须修复）"
echo "  警告：   $WARN（应该修复）"
echo ""

[[ $FAIL -gt 0 ]] && exit 1 || exit 0

```

### 5.4 基于角色的护栏

不同的开发者有不同的风险画像，相应地定制 Claude Code 设置。

**方式：在 CLAUDE.md 中按经验/角色分层**

```Markdown
<!-- CLAUDE.md — 角色感知指南 -->
## 开发者上下文

这是一个[初级|高级|负责人]开发者项目上下文。

### 如果是初级（入职 <1 年）
- 实施前始终确认架构决策
- 未与高级开发者结对，不要修改数据库架构、迁移或认证代码
- 每个 PR 都必须有人工审查员明确检查 AI 生成部分
- 实施任何超过 50 行的内容前使用计划模式

### 如果是高级（入职 1 年以上）
- 适用标准审查
- 可以修改大多数文件，但 auth/支付/加密仍需负责人审查
- PR 中需要 AI 归因

### 如果是负责人/主任工程师
- 完全访问，基于判断的限制
- 负责为其团队项目设置护栏级别
- 必须主持季度 MCP 注册表审查

```

**方式：每个环境使用不同的 settings.json**

在 CI/CD 中，在流水线启动时检查活跃的 `settings.json` 路径以强制执行正确的级别。Claude Code 从项目根目录读取 `.claude/settings.json`——在那里提交您的严格级别配置，使 CI 始终能获取它，无论开发者本地有什么配置。

```Bash
# 在 CI 流水线设置步骤中，验证提交了正确的级别
if ! grep -q '"Bash(curl \*)"' .claude/settings.json; then
  echo "错误：CI 需要合规级 settings.json（必须拒绝 curl）"
  exit 1
fi

```

### 5.5 CI/CD 门控

阻止不合规的 AI 使用到达生产环境：

```YAML
# .github/workflows/ai-governance.yml
name: AI 治理检查

on: [pull_request]

jobs:
  governance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: 检查治理配置是否存在
        run: |
          if [[ ! -f ".claude/settings.json" ]]; then
            echo "::error::缺少 .claude/settings.json——需要治理配置"
            exit 1
          fi

      - name: 检查无凭证访问权限
        run: |
          if jq -e '.permissions.allow[]? | select(test("env|pem|key|secret"))' \
            .claude/settings.json 2>/dev/null; then
            echo "::error::检测到危险的 permissions.allow——凭证可能暴露"
            exit 1
          fi

      - name: 运行治理审计
        run: |
          chmod +x scripts/claude-governance-audit.sh
          ./scripts/claude-governance-audit.sh

      - name: 检查 PR 描述中的 AI 归因
        if: ${{ env.REQUIRE_AI_ATTRIBUTION == 'true' }}
        uses: actions/github-script@v7
        with:
          script: |
            const body = context.payload.pull_request.body || '';
            const hasAttribution = body.includes('AI') ||
                                   body.includes('Claude') ||
                                   body.includes('AI-generated');
            if (!hasAttribution) {
              core.warning('未找到 AI 归因部分，请披露 AI 使用情况。');
            }

```

---

## 6. 审计、合规与治理结构

### 6.1 SOC2 和 ISO27001 审计师实际关注的内容

审计师审查 AI 编码工具使用时，通常会寻找以下控制措施的证据：

| 审计师问题 | 他们希望看到的 | Claude Code 实现 |
|-|-|-|
| "您是否有 AI 工具使用策略？" | 书面章程，已签署/确认 | `docs/ai-usage-charter.md` + 入职检查清单 |
| "您如何控制发送给 AI 供应商的数据？" | 数据分类 + 技术控制 | `permissions.deny` 拦截敏感文件 |
| "您如何审查第三方 AI 组件？" | 审批流程 + 注册表 | MCP 注册表 + 审批流程 |
| "您是否有 AI 操作的审计跟踪？" | 工具调用、文件访问日志 | 会话 JSONL 日志 + `compliance-audit-logger.sh` |
| "您如何审查 AI 生成的代码？" | 带有 AI 披露的代码审查流程 | PR 模板 + 归因策略 |
| "发生事故时如何处理？" | 事故响应程序 | 现有 IR 流程 + AI 特定补充 |

**对于 SOC2**：相关的信任服务标准是 CC6.1（逻辑访问控制）、CC6.3（访问删除）、CC7.1（监控）和 CC9.2（供应商风险）。您的 Claude Code 治理应映射到这些标准。

**对于 ISO27001**：相关的附录 A 控制包括 A.8.3（信息访问限制）、A.8.24（密码学使用）、A.8.25（安全开发生命周期）和 A.5.23（云服务信息安全）。

### 6.2 审计跟踪设置

Claude Code 会话已记录到 `~/.claude/projects/<project>/*.jsonl`。挑战在于让它们：

1. 事后可访问（开发者离职后不丢失）
2. 防篡改（不能事后编辑）
3. 可查询以用于审计目的

**最小审计跟踪（无需额外工具）**：

```Bash
#!/bin/bash
# .claude/hooks/compliance-audit-logger.sh
# 事件：PostToolUse（所有工具）
# 将结构化审计条目追加到共享日志

LOG_DIR="${COMPLIANCE_LOG_DIR:-/var/log/claude-audit}"
LOG_FILE="$LOG_DIR/$(date +%Y-%m-%d).jsonl"

mkdir -p "$LOG_DIR"

INPUT=$(cat)
TOOL=$(echo "$INPUT" | jq -r '.tool.name // "unknown"')
USER=$(whoami)
PROJECT=$(basename "$PWD")
TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

echo "{\"timestamp\":\"$TIMESTAMP\",\"user\":\"$USER\",\"project\":\"$PROJECT\",\"tool\":\"$TOOL\"}" \
  >> "$LOG_FILE"

```

**将日志传送至不可变存储**（受监管环境推荐）：

```Bash
# 每日：将会话日志同步至不可变存储桶
aws s3 sync ~/.claude/projects/ \
  s3://your-audit-bucket/claude-sessions/$(whoami)/ \
  --storage-class GLACIER_INSTANT_RETRIEVAL \
  --exclude "*.tmp"

```

**对于具有审批门控的完整合规审计跟踪**，可考虑 Entire CLI——它通过密码学链接到 git 提交来捕获完整会话上下文（提示词、推理、工具调用、文件差异）。详情及设置请参见 「AI 可追溯性 §5.1」。这是多个可选工具之一，请根据您的具体合规需求进行评估。

### 6.3 AI 治理委员会（精简参考）

对于在规模上管理 AI 风险的组织，轻量级的 AI 治理委员会提供了审计师期望的问责结构。这是一个协调机制，而非瓶颈。

**最小结构**（适用于 10–100 名开发者）：

| 角色 | 人员 | 职责 |
|-|-|-|
| **治理负责人** | 工程经理或负责人 | 策略更新、季度审查、升级处理 |
| **安全代表** | SecEng 或 DevSecOps | MCP 风险审查、事故响应 |
| **开发代表** | 高级开发者轮换（3 个月任期） | 开发者反馈、可用性平衡 |
| **合规代表** | 法律/合规（仅受监管行业） | 章程、法规映射 |

**会议频率**：季度（30 分钟）。固定议程：

1. MCP 注册表审查——有什么需要添加、删除或标记的？
2. 事故审查——上次会议以来有什么 AI 相关安全事件？
3. 策略更新——章程是否需要任何变更？
4. 指标——治理审计结果、合规检查状态

详细的 AI 治理委员会结构、RACI 矩阵和合规映射，请参见白皮书 #11：企业 AI 治理（法语/英语）。

### 6.4 合规监控

治理的可观测层在 「observability.md」 中介绍。对于合规而言，以下监控查询最为相关：

```Bash
# Claude 在过去 30 天访问了哪些文件？
# macOS: date -v-30d; Linux: date -d '30 days ago'
if [[ "$OSTYPE" == "darwin"* ]]; then
  SINCE=$(date -v-30d +%Y-%m-%d)
else
  SINCE=$(date -d '30 days ago' +%Y-%m-%d)
fi

find ~/.claude/projects/ -name "*.jsonl" -newer "$SINCE" | \
  xargs jq -r 'select(.type == "assistant") |
    .message.content[]? |
    select(.type == "tool_use" and .name == "Read") |
    .input.file_path' 2>/dev/null | sort -u

# 是否有访问敏感模式的记录？
# （在上面的命令之后运行，通过 grep 过滤）
grep -E '\.(env|pem|key)$|secrets/|credentials'

# Claude 本周运行了哪些 Bash 命令？
if [[ "$OSTYPE" == "darwin"* ]]; then
  SINCE_WEEK=$(date -v-7d +%Y-%m-%d)
else
  SINCE_WEEK=$(date -d '7 days ago' +%Y-%m-%d)
fi

find ~/.claude/projects/ -name "*.jsonl" -newer "$SINCE_WEEK" | \
  xargs jq -r 'select(.type == "assistant") |
    .message.content[]? |
    select(.type == "tool_use" and .name == "Bash") |
    .input.command' 2>/dev/null | sort

```

---

## 快速参考

### 级别选择

| 您的情况 | 级别 | 配置时间 |
|-|-|-|
| 个人项目、个人使用 | 入门级 | 10 分钟 |
| 小型团队、内部项目 | 入门级 | 10 分钟 |
| 5–20 人团队、任何生产代码 | 标准级 | 45 分钟 |
| 20+ 人团队、客户数据 | 严格级 | 2 小时 |
| 受监管行业（HIPAA/SOC2/PCI） | 合规级 | 半天 |

### 治理成熟度级别

| 成熟度 | 您拥有的 | 缺少的 |
|-|-|-|
| **临时** | 每个开发者自行配置 | 一致性、责任制 |
| **基础** | 共享 CLAUDE.md + settings.json | MCP 治理、审计跟踪 |
| **管理** | + MCP 注册表 + 钩子 | 合规报告 |
| **合规** | + 审计日志 + 章程 + 审查周期 | 无关键缺失 |
| **审计** | + 外部验证 + 可追溯性 | — |

### 常见错误

| 错误 | 修复方法 |
|-|-|
| 治理仅在 `~/.claude`（个人）中 | 移至代码库的 `.claude/` 中 |
| `permissions.allow` 覆盖团队的 `deny` | 每季度审查个人配置 |
| 无 MCP 注册表 → 每个开发者添加不同的 MCP | 即使只有 3 个条目也要建立注册表 |
| CLAUDE.md 太长 → Claude 忽略规则 | 保持在 8KB 以下，优先显示关键规则 |
| 审计师要求 AI 日志 → 什么都没保存 | 设置会话日志同步到 S3 |

---

## 另见

- 「安全加固」 — 个人开发者安全：MCP CVE、注入防御、5 分钟审计
- 「生产安全规则」 — 生产团队的 6 条不可妥协规则（端口、数据库安全、基础设施锁定）
- 「数据隐私指南」 — Claude Code 向 Anthropic 发送什么数据、保留策略
- 「AI 可追溯性」 — 归因策略、Entire CLI、git-ai、合规框架
- 「可观测性」 — 会话监控、成本跟踪、活动审计查询
- 「采用方式」 — 团队发布模式、CLAUDE.md 策略
- MCP 注册表模板 — 即用型注册表格式
- 治理钩子 — 验证配置是否符合策略的钩子
- 「AI 使用章程模板」 — 可直接改编的章程模板

---

## 参考资料

- [Liminal AI 企业治理指南](https://www.liminal.ai/blog/enterprise-ai-governance-guide) — 实践实现
- [Databricks AI 治理框架](https://www.databricks.com/blog/practical-ai-governance-framework-enterprises) — 企业规模框架
- [Augmentcode AI 代码治理](https://www.augmentcode.com/guides/ai-code-governance-framework-for-enterprise-dev-teams) — 开发团队专项
- [Partnership on AI — 2026 年六大治理优先级](https://partnershiponai.org/resource/six-governance-priorities/) — 评估框架、问责制
- [欧盟 AI 法案](https://www.europarl.europa.eu/doceo/document/TA-9-2024-0138_EN.html) — 高风险 AI 系统的强制停止要求
- [NIST AI RMF](https://airc.nist.gov/RMF/Overview) — 风险管理框架
- [SOC2 信任服务标准](https://www.aicpa.org/resources/article/soc-2-trust-services-criteria) — CC6.1、CC7.1、CC9.2

---

*版本 1.0.0 | 2026 年 3 月 | 《Claude Code：从入门到大师》的一部分*

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/Z0X3wMmGhiPbMBkF0sOcjzHRnQg> ｜ 归档：2026-06-04
