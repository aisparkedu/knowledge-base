# 团队 AI 指令管理

跨团队统一管理 AI 指令（CLAUDE.md、.cursorrules），避免碎片化。

**模式**：基于配置文件的模块化组装——共享模块 + 个人配置文件 + 自动化组装器。

**适用场景**：5 人以上团队、多种 AI 工具（Claude Code + Cursor/Windsurf）、混合操作系统。**跳过条件**：单独开发者、同质化团队（相同工具、相同操作系统）、短期项目（<3 个月）。

---

## 问题：N × M × P 碎片化

团队扩大时，AI 指令会迅速碎片化：

| 因素 | 取值 | 示例 |
|-|-|-|
| **N 名开发者** | 5-20 人 | 张三、李四、王五…… |
| **M 种工具** | 2-4 种 | Claude Code、Cursor、Windsurf、Copilot |
| **P 种操作系统** | 2-3 种 | macOS、Linux、WSL |

**变体总数**：N × M × P = 5 × 3 × 2 = **30 种可能的配置**。

没有统一管理时会发生什么：

```Plain Text
第 1 周：团队商定共享 CLAUDE.md
第 3 周：张三在本地添加 TypeScript 严格规则
第 5 周：李四复制了张三的文件，删掉了一半规则
第 8 周：新成员王五得到了李四过时的副本
第 12 周：5 名开发者，5 个不同的 CLAUDE.md，没人知道哪个是权威版本

```

**根本原因**：CLAUDE.md 被当作整体文件，而非组合配置。

---

## 架构概览

```Plain Text
profiles/                    modules/
├── alice.yaml               ├── core-standards.md
├── bob.yaml                 ├── git-workflow.md
├── charlie.yaml             ├── typescript-rules.md
│                            ├── test-conventions.md
│                            ├── macos-paths.md
│                            ├── linux-paths.md
│                            ├── cursor-rules.md
│                            └── communication-verbose.md
│
├── skeleton/
│   └── claude-skeleton.md   ← 包含 {{MODULE:name}} 占位符的模板
│
└── sync-ai-instructions.ts  ← 读取配置文件 → 注入模块 → 写入输出
        │
        ▼
output/
├── alice/CLAUDE.md          ← 自动生成（只读）
├── bob/CLAUDE.md
└── charlie/CLAUDE.md

```

**流程**：配置文件（YAML）+ 骨架（模板）+ 模块（片段）→ 组装器 → 生成的 CLAUDE.md

---

## 阶段 1：审计当前 CLAUDE.md

**目标**：将每一行分类为通用、条件性或个人。

```Markdown
# 审计模板

## 通用（所有开发者，所有工具）
- 架构：六边形
- 测试：PR 前必须通过
- 命名：文件使用 kebab-case

## 条件性（取决于工具或操作系统）
- Cursor：使用 @filename 语法 → 模块：cursor-rules
- macOS 路径：/opt/homebrew → 模块：macos-paths
- Linux 路径：/usr/local → 模块：linux-paths

## 个人（个人偏好）
- 风格：详细解释 → 配置文件偏好
- 语言：中文注释 → 配置文件偏好

```

**测量命令**：

```Bash
wc -l CLAUDE.md  # 模块化前的总行数
# 用 [U]niversal、[C]onditional、[P]ersonal 标记每行
# 按类别计数以估算模块划分

```

**典型结果**：60% 通用，25% 条件性，15% 个人。

---

## 阶段 2：提取模块

**目标**：每个主题组一个 `.md` 文件。

**推荐结构**：

```Plain Text
modules/
├── core-standards.md         # 架构、命名、模式（所有开发者）
├── git-workflow.md           # Git 约定（所有开发者）
├── typescript-rules.md       # TS 严格配置（TypeScript 项目）
├── test-conventions.md       # 测试模式（所有开发者）
├── macos-paths.md            # macOS 特定路径（macOS 用户）
├── linux-paths.md            # Linux 路径（Linux 用户）
├── cursor-rules.md           # Cursor 特定规则（Cursor 用户）
└── communication-verbose.md  # 详细解释风格（有此偏好时）

```

**模块格式**（每个模块是一个独立的 Markdown 片段）：

```Markdown
<!-- modules/typescript-rules.md -->
## TypeScript 规则

- 使用严格模式：tsconfig 中的 `"strict": true`
- union 类型优先用 `type` 而非 `interface`
- 禁止 `any`——使用 `unknown` + 类型守卫
- 在边界使用 Zod 进行运行时验证

```

**指导原则**：

- 保持模块自包含（模块间无交叉引用）
- 每个模块 15-50 行是最佳区间
- 按领域命名模块，而非按受众
- 一个模块 = 一个变更原因

---

## 阶段 3：创建开发者配置文件

**目标**：每名开发者一个 YAML，列出其模块。

```YAML
# profiles/alice.yaml
name: "Alice"
os: "macos"
tools:
  - claude-code
  - cursor
communication_style: "concise"
modules:
  core:
    - core-standards
    - git-workflow
    - typescript-rules
    - test-conventions
  conditional:
    - macos-paths          # os: macos 时自动包含
    - cursor-rules         # tools 中有 cursor 时自动包含
preferences:
  language: "english"

```

**配置文件规则**：

- `core` 模块：每位开发者都包含（团队标准）
- `conditional` 模块：基于 `os` 和 `tools` 字段包含
- `preferences`：注入骨架变量的个人设置

**新团队成员模板**：参见 profile-template.yaml

---

## 阶段 4：编写组装脚本

**目标**：读取配置文件、注入模块、输出 CLAUDE.md 的脚本。

```TypeScript
// sync-ai-instructions.ts（简化版，约 30 行）
import { readFileSync, writeFileSync, mkdirSync } from 'fs';
import { parse } from 'yaml';
import { join } from 'path';

const profile = parse(readFileSync(`profiles/${process.argv[2]}.yaml`, 'utf8'));
let skeleton = readFileSync('skeleton/claude-skeleton.md', 'utf8');

// 从配置文件收集模块
const modules = [...profile.modules.core, ...profile.modules.conditional];

// 用模块内容替换每个占位符
for (const mod of modules) {
  const content = readFileSync(`modules/${mod}.md`, 'utf8');
  skeleton = skeleton.replace(`{{MODULE:${mod}}}`, content);
}

// 移除未使用的占位符
skeleton = skeleton.replace(/\{\{MODULE:\w+\}\}/g, '');

// 写入输出
const outDir = `output/${process.argv[2]}`;
mkdirSync(outDir, { recursive: true });
writeFileSync(join(outDir, 'CLAUDE.md'), skeleton);
console.log(`已生成 ${outDir}/CLAUDE.md（${modules.length} 个模块）`);

```

**运行**：

```Bash
npx ts-node sync-ai-instructions.ts alice   # 单个开发者
npx ts-node sync-ai-instructions.ts --all   # 生成所有配置文件
npx ts-node sync-ai-instructions.ts --check # 验证无漂移

```

完整模板：sync-script.ts

---

## 阶段 5：CI 漂移检测

**目标**：捕获输出文件与配置文件/模块不同步的情况。

```YAML
# .github/workflows/ai-instructions-check.yml
name: AI Instructions Drift Check
on:
  push:
    paths:
      - 'modules/**'
      - 'profiles/**'
      - 'skeleton/**'
  schedule:
    - cron: '0 9 * * 1-5'  # 工作日上午 9 点

jobs:
  check-drift:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npx ts-node sync-ai-instructions.ts --check
      - name: 检测到漂移时失败
        run: |
          if git diff --quiet output/; then
            echo "未检测到漂移"
          else
            echo "::error::AI 指令不同步！"
            git diff output/
            exit 1
          fi

```

**可检测到的情况**：

- 模块已编辑但组装器未重新运行
- 已添加配置文件但不存在输出 CLAUDE.md
- 开发者手动编辑了生成的 CLAUDE.md

**策略**：生成的文件是只读的。所有变更通过配置文件/模块进行，然后重新运行组装器。

---

## 阶段 6：新开发者入职

**目标**：新开发者在 5 分钟内获得其 CLAUDE.md。

```Bash
# 1. 克隆仓库
git clone <repo>

# 2. 复制配置文件模板
cp examples/team-config/profile-template.yaml profiles/dave.yaml
# 编辑：name、os、tools、modules

# 3. 生成
npx ts-node sync-ai-instructions.ts dave

# 4. 安装
cp output/dave/CLAUDE.md .claude/CLAUDE.md

```

**CLAUDE.md 放置提醒**：

- 项目范围：`project/CLAUDE.md`（已提交，用于团队约定）
- 个人覆盖：`.claude/CLAUDE.md`（在 gitignore 中，用于个人偏好）

---

## 故障排查

| 问题 | 原因 | 修复 |
|-|-|-|
| 生成的文件过长 | 包含了太多模块 | 审查配置文件：移除很少使用的模块 |
| 输出中缺少模块 | 骨架中占位符拼写错误 | 检查 `{{MODULE:name}}` 是否与文件名匹配 |
| CI 漂移警报 | 模块编辑后未重新生成输出 | 运行 `sync-ai-instructions.ts` 并提交 |
| 开发者 A 有开发者 B 没有的规则 | 符合预期——这就是设计目的 | 验证该开发者的配置文件是否正确 |
| 合并后输出过时 | 合并未触发重新生成 | 合并后运行组装器（添加 git hook） |

---

## 扩展阈值

| 团队规模 | 方法 |
|-|-|
| 1-2 名开发者 | 共享 CLAUDE.md + 优先级规则（第 3.4 节） |
| 3-5 名开发者，相同工具 | 可选：仅模块，无配置文件 |
| 5+ 名开发者或多工具 | 基于配置文件的模块化组装（此工作流） |
| 20+ 名开发者 | 考虑 CLAUDE.md 配置服务器 + 基于 PR 的模块变更 |

---

## 实测结果

来自生产团队（5 名开发者，3 种工具，2 种操作系统）：

| 指标 | 之前 | 之后 |
|-|-|-|
| 每个 CLAUDE.md 的行数 | ~380（整体） | ~185（组装） |
| Token 减少 | — | 上下文消耗减少 59% |
| 提取的模块数 | 0 | 12 |
| 入职时间 | 「复制别人的文件」 | 5 分钟（模板 + 生成） |
| 漂移事件 | 每周 | 0（CI 捕获） |

---

## 相关内容

- 第 3.5 节 大规模团队配置 — 概念概述和实测结果
- 第 3.4 节 优先级规则 — Claude 如何读取多个 CLAUDE.md 文件
- profile-template.yaml — 配置文件模板
- 「claude-skeleton.md」 — 骨架模板
- sync-script.ts — 完整组装脚本

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/AyDzwUyEpi7e6ok4vkDccGfTn1g> ｜ 归档：2026-06-04
