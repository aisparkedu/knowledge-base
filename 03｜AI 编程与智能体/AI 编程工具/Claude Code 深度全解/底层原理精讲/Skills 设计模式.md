# Skills 设计模式

> **相关内容**：「开发方法论」 | 「多智能体协调」

适用于超越单智能体、单文件提示词的 Skills（技能模块）的实践模式。这些模式针对具体的失败场景：重复发现相同事实的子智能体、应用于错误文件的规则、将检测与修复混为一谈的 Skills（技能模块）。

---

## 共享基础事实注入

**问题**：当你启动 N 个并行子智能体来审计或分析一组产物时，每个智能体都会独立发现相同的基础事实（文件列表、导航结构、CLI 命令、Schema）。这意味着 N 次冗余读取、N 个需要分别信任的独立事实，以及 N 次某个智能体看到过时文件状态的风险。

**模式**：编排者一次性计算一个共享的事实基准，然后将相同的块逐字注入到每个子智能体的提示词中。

```Plain Text
编排者（阶段 1）
  ├── 读取导航结构 → 提取分组
  ├── 文件匹配 → 获取当前列表
  ├── 列出 CLI 命令 → 获取当前命令
  └── 编译成"基础事实"字符串

编排者（阶段 2）
  ├── 智能体 1 提示词："## 基础事实\n{共享块}\n\n## 你的范围\n章节 A"
  ├── 智能体 2 提示词："## 基础事实\n{共享块}\n\n## 你的范围\n章节 B"
  └── 智能体 3 提示词："## 基础事实\n{共享块}\n\n## 你的范围\n章节 C"

```

**共享块应包含的内容**：

- 导航结构或文件层级（让每个智能体知道有哪些内容）
- 当前 CLI 命令或 API 端点（用于捕获对已删除命令的引用）
- 领域实体列表（包、模块、服务）
- 当前日期（用于捕获现在已过时的版本引用）

**为何有效**：每个智能体从相同的快照开始。如果基础事实列表显示某个 CLI 命令不存在，智能体就无法凭空捏造一个。编排者是结构的单一事实来源；子智能体是各自负责章节内容的单一事实来源。

**Token（词元）权衡**：共享块会为每个子智能体提示词增加 Token（词元）。对于 5 个智能体的并行审计，500 个 Token（词元）的基础事实块预先花费 2500 个 Token（词元）。消除每智能体的重复发现会节省这些成本，而且远不止于此。将块的大小控制在实际需要的范围内，而不是你能包含的所有内容。

**实现注意事项**：基础事实块应该是一个字符串，而不是文件引用。如果你让智能体"读取 `docs.json`"，每个智能体都会独立读取它。在编排者中一次性编译，然后直接粘贴进去。

> 在 [Packmind doc-audit skill](https://github.com/packmind/packmind)（.claude/skills/doc-audit/SKILL.md，阶段 1）中观察到。参见「致谢」。

---

## 通过 Frontmatter 路径进行预过滤引用

**问题**：你有一组规则文件（编程规范、安全策略、风格指南）。每个文件适用于代码库中特定的文件子集。如果你将所有规则传递给审查智能体，智能体会将规则应用于它们不适用的文件，产生误报并浪费上下文。

**模式**：在每个规则文件中添加一个 `paths:` frontmatter 字段，使用 glob 模式。在启动审查智能体之前，编排者读取此 frontmatter，将 glob 与已修改文件列表进行匹配，并只向每个智能体传递适用的规则。

```YAML
# .claude/rules/standard-testing-good-practices.md
---
name: 测试最佳实践
paths: "**/*.spec.ts,**/*.test.ts"
alwaysApply: false
description: TypeScript 单元测试和集成测试的测试规范
---

## 规则
- 使用 `describe` 块对相关测试进行分组
...

```

编排者逻辑（简明步骤）：

```Plain Text
1. 通配符匹配 .claude/rules/**/*.md → 列出所有规则文件
2. 对每个规则文件：
   a. 读取 YAML frontmatter → 提取 paths（glob 模式）
   b. 如果 alwaysApply: true → 无条件包含
   c. 如果 alwaysApply: false → 将 paths 与已修改文件列表匹配
   d. 如果至少一个已修改文件与 glob 匹配则包含
3. 将过滤后的规则集传递给每个审查智能体

```

**主要优势**：

- 测试文件不会被后端安全规则检查
- 迁移文件不会被前端风格指南检查
- 每个智能体的上下文只包含对其审查内容可操作的规则

**规模化效果**：有 50 个规则文件时，一个典型的涉及 8 个 TypeScript 规范文件的 PR 可能只匹配 3 条规则。不过滤时，智能体读取 50 条规则。过滤后，它只读取 3 条。智能体的信噪比相应提升。

**何时添加 paths: frontmatter**：任何具有明确文件类型范围的规则（测试文件、迁移文件、前端组件、API 路由）。真正全局适用的规则使用 `alwaysApply: true` 并跳过匹配步骤。

> 在 [Packmind qa-review skill](https://github.com/packmind/packmind)（.claude/skills/qa-review/SKILL.md，阶段 5）中观察到。参见「致谢」。

---

## 仅检测范围边界

**问题**：同时负责检测问题和修复问题的 Skills（技能模块）有两种失败模式：误报（检测并修复了实际上不是问题的内容）和不完整修复（检测正确但修复错误）。混合两者会使两者都变得更糟，并且不给用户提供审查检查点。

**模式**：明确将 Skills（技能模块）限定为仅检测。Skills（技能模块）产生报告。修复是一个独立的步骤，由用户在读取报告后触发。

在 Skills（技能模块）顶部声明这一点：

```Markdown
**本 Skills（技能模块）仅检测问题，不修复问题。**

```

然后强制执行：Skills（技能模块）内不进行文件写入、不进行编辑、不进行 git 提交。输出始终是 Markdown 报告或控制台发现结果。

**何时打破此规则**：专门为自动化修复设计的 Skills（技能模块）（codemod 风格的 Skills（技能模块）、依赖更新 Skills（技能模块））是例外。它们应明确说明会写入变更，并应包含试运行（dry-run）模式。

**实用价值**：仅检测的 Skills（技能模块）可以安全地在 main 分支、CI 中以及针对生产配置运行。用户可以在不担心意外变更的情况下调用它们，这使得它们更值得更频繁地在更多代码库上运行。

> 在 [Packmind 仓库](https://github.com/packmind/packmind)（Apache 2.0）的 qa-review、playbook-audit 和 doc-audit 中均有此重复出现的模式。参见「致谢」。

---

## 输入处理器分发

**问题**：处理两种或多种异构输入类型（图像 vs 文本，GitHub Issue vs 设计稿）的 Skills（技能模块）要么在主文件中产生复杂的分支逻辑，要么对于多样化的入口点过于僵化。

**模式**：在 `inputs/` 子目录中为每种输入类型存储一个输入处理器文件。主 SKILL.md 询问用户适用哪种类型，然后加载匹配的处理器文件。

```Plain Text
.claude/skills/my-skill/
├── SKILL.md                 # 编排者：询问用户，分发到输入处理器
└── inputs/
    ├── github-issue.md      # GitHub Issue 输入处理器
    └── visual-mockup.md     # 截图/图像输入处理器

```

SKILL.md（节选）：

```Markdown
## 步骤 1：识别输入类型

询问用户："你提供的是 GitHub Issue 还是视觉稿？"

- 如果是 GitHub Issue → 按照 `inputs/github-issue.md` 中的指令操作
- 如果是视觉稿 → 按照 `inputs/visual-mockup.md` 中的指令操作

```

每个处理器文件包含该输入类型的完整解析指令，包括字段提取、格式规范化和验证。主文件保持简洁。

**何时使用**：当输入类型需要实质性不同的解析逻辑时（不只是不同的字段名）。如果两种输入相差不超过 5 个步骤，直接内联处理。当处理器各自超过 100 行时，该模式才物有所值。

> 在 [Packmind create-em-spec skill](https://github.com/packmind/packmind)（.claude/skills/create-em-spec/inputs/）中观察到。参见「致谢」。

---

## 工具版本耦合的版本化子目录

**问题**：Skills（技能模块）包装了一个在不同版本间行为会发生变化的 CLI 工具。Skills（技能模块）需要检测用户使用的版本并相应调整指令。

**模式**：为每个工具版本存储一个子目录，每个子目录包含版本特定的指令：

```Plain Text
.claude/skills/update-playbook/
├── SKILL.md                         # 检测 CLI 版本，分发
└── tool-versions/
    ├── v1.21/
    │   └── apply-changes.md         # v1.21 的指令
    ├── v1.23/
    │   └── apply-changes.md         # v1.23 的指令（不同 API）
    └── v1.24/
        └── apply-changes.md         # v1.24 的指令（破坏性变更）

```

SKILL.md 在运行时检测版本：

```Markdown
## 步骤 1：检测工具版本

运行：`my-cli --version`

- 如果输出以 "1.21" 开头 → 读取 `tool-versions/v1.21/apply-changes.md`
- 如果输出以 "1.23" 开头 → 读取 `tool-versions/v1.23/apply-changes.md`
- 如果输出以 "1.24" 开头 → 读取 `tool-versions/v1.24/apply-changes.md`
- 如果版本无法识别 → 停止并告知用户支持哪些版本

```

**避免的反模式**：不要在原始 `my-skill/` 旁边创建 `my-skill-v2/` 目录。对 Skills（技能模块）*内部*的内容进行版本管理，而不是对 Skills（技能模块）本身进行版本管理。两个几乎相同的 Skills（技能模块）更难维护，调用时也容易引起混淆。

**何时使用**：当你包装的 CLI 工具在版本间有破坏性变更，且需要同时支持多个版本时。如果你只需要支持最新版本，直接在原处更新 Skills（技能模块）即可。

> 在 [Packmind update-playbook skill](https://github.com/packmind/packmind)（.claude/skills/packmind-update-playbook/）中观察到。参见「致谢」。

---

## 两级规范

**问题**：全面的编程规范很长（1000-5000 字）。每次文件审查时将完整规范加载到上下文中会增加 Token（词元）成本并分散注意力。

**模式**：在 `.claude/rules/` 中存储一份简短摘要（100-300 字，带有 `paths:` frontmatter glob）。将完整的权威规范保存在一个单独位置，Claude 可以按需读取。

```Plain Text
.claude/rules/standard-testing.md          # 摘要 + paths glob（200 字）
.standards/standard-testing-full.md        # 完整权威规范（2000 字）

```

`.claude/rules/` 中的摘要包含：

- 3-5 条规定性规则（命令式语气："使用 X"、"不要 Y"）
- `paths:` frontmatter，使其只为匹配文件加载
- 完整规范的链接，用于需要详细内容的情况

完整的权威规范存放在 `.claude/rules/` 之外，这样就不会自动加载。

**何时使用**：拥有超过 10 个规则文件的团队，或平均超过 500 字的规则文件。对于更小的配置，单级方式更简单。

**维护**：更新权威规范时，同步更新摘要。两级拆分会产生重复风险。通过将摘要仅限于要点规则（无散文）来缓解这一风险，这样它就不会频繁变更。

> 在 [Packmind 规则配置](https://github.com/packmind/packmind)（.claude/rules/packmind/）中观察到。参见「致谢」。

---

## 将计划和规范作为已提交的产物

**问题**：会话范围内的计划（`/plan` 模式、草稿文件）在会话结束后消失。下一个会话没有可搜索的记录来了解设计决策的原因、考虑了哪些选项，或最后完成了哪个实现步骤。

**模式**：将计划及其相关设计规范作为带日期的 Markdown 配对文件直接提交到 `.claude/`：

```Plain Text
.claude/
├── plans/
│   └── 2026-03-15-auth-refactor.md        # 高层计划：阶段、任务、提交消息
└── specs/
    └── 2026-03-15-auth-refactor-design.md  # 配套规范：上下文、替代方案、理由

```

计划文件包含带有每个任务显式提交消息的实现分解：

```Markdown
# Auth 重构计划

## 任务 1：提取 Token（词元）验证中间件
- 将验证逻辑从控制器迁移到 `middleware/auth.ts`
- git commit -m "refactor(auth): extract token validation into middleware"

## 任务 2：添加刷新 Token（词元）轮换
- 实现带 7 天过期的轮换逻辑
- git commit -m "feat(auth): add refresh token rotation with 7-day expiry"

```

规范文件捕获了计划背后的信息：约束条件、被拒绝的替代方案，以及六个月后从代码中看不出来的推理：

```Markdown
# Auth 重构设计

## 背景
会话 Token（词元）存储被法律审查标记（ISO 27001 合规差距）。
根据新内部安全政策 v2.3，需要实施轮换。

## 被拒绝的方案
- HttpOnly cookie 方案：需要跨域配置变更（延迟处理）
- Redis 会话存储：运维团队尚未准备好管理另一个有状态服务

```

两个文件一起提交并无限期保留在仓库中。

**为何优于仅会话内的计划**：已提交的计划可以被 grep 搜索、差异对比和恢复。新会话可以读取 `.claude/plans/` 来了解进行中的工作，而不需要从 git log 重建。规范捕获了从记忆中消失但在六个月后重新审视决策时至关重要的推理。

**命名规范**：计划使用 `YYYY-MM-DD-<slug>.md`，配套规范使用 `YYYY-MM-DD-<slug>-design.md`，两者使用相同的日期和 slug。这样在目录列表中配对文件在视觉上是关联的。

**何时使用**：多会话实现任务，连续性很重要的情况下。对于单次会话、可以用一个提交完成的任务，这种开销不值得。大致的阈值是：如果工作跨越超过一天或超过一个 Claude 会话，就提交计划。

> 在 [Packmind .claude/plans/](https://github.com/packmind/packmind) 规范（Apache 2.0）中观察到。参见「致谢」。

---

## 运行时提示词日志记录

**问题**：当 AI 提供商调用超时或崩溃时，发送的确切提示词就消失了。如果你正在构建一个有多个智能体并行运行的 Skills（技能模块）或评估流水线，你就无法诊断每个智能体被告知了什么。`--debug` 标志只在你记得传递它时才有用。

**模式**：在调用提供商之前，将提示词作为阻塞操作写入磁盘。使用带时间戳文件名的持久目录。日志记录调用永远不抛出异常。

```TypeScript
// 在 provider.invoke() 之前运行，而不是之后
await writeFile(
  `prompts/debug/${evaluatorName}-${timestamp}.md`,
  fullPrompt,
  "utf-8",
);
// 现在才调用提供商
const response = await provider.invoke(fullPrompt);

```

三个约束使这个模式有效：

1. **阻塞写入**（`await`，不是即发即忘）：提供商调用开始之前，文件就已存在于磁盘。提供商调用期间的崩溃会保留提示词以供检查。
2. **始终开启**（不由调试标志控制）：开销是每个智能体一次文件写入。好处是任何意外结果都有永久记录。
3. **永不抛出**：日志记录失败不能中断评估。将写入包裹在 try/catch 中，失败时记录警告，继续执行。

**目录规范**：使用相对于项目根目录的路径（`prompts/debug/` 或 `claudedocs/debug/`）而不是临时目录。临时目录会被清理；你希望这些文件跨运行持久保存。

**Token（词元）成本**：零（发生在 AI 调用之外的磁盘 I/O）。提示词已经在内存中，你只是在持久化它。

**何时使用**：任何调用 AI 提供商且提示词是动态组装的 Skills（技能模块）（不只是静态字符串）。价值与提示词复杂度成正比：如果你的提示词注入了基础事实、评估器指令和文件内容，在调用前进行一次写入是廉价的保险。

> 在 [PackmindHub/context-evaluator](https://github.com/PackmindHub/context-evaluator)（`src/shared/evaluation/runtime-prompt-logger.ts`，MIT）中观察到。参见「致谢」。

---

## 自适应统一/并行模式

**问题**：你有 N 个文件需要评估，需要决定：将所有文件发送给一个智能体（更好地处理跨文件问题，但上下文成本更高），还是将每个文件发送给各自的并行智能体（更便宜、更快，但对文件间矛盾视而不见）？

**模式**：在确定策略之前，先估算合并后的 Token（词元）数量。如果总量在阈值内，使用统一模式（一个智能体看到所有内容）。如果超出阈值，回退到每个文件独立的并行智能体。

```TypeScript
const DEFAULT_MAX_UNIFIED_TOKENS = 100_000;

function canUseUnifiedMode(context: MultiFileContext, maxTokens = DEFAULT_MAX_UNIFIED_TOKENS) {
  if (context.totalTokenEstimate > maxTokens) {
    return {
      viable: false,
      reason: `合并内容（约 ${context.totalTokenEstimate} Token（词元））超出限制（${maxTokens}）`,
    };
  }
  return { viable: true };
}

```

在构建智能体提示词之前调用此函数。决策控制流水线的其余部分：

```Plain Text
估算所有文件的 Token（词元）数
  ↓
< 10 万 → runUnifiedEvaluation(allFiles)   // 1 个智能体，跨文件检测
> 10 万 → runAllEvaluators(file)           // N 个智能体，每文件一个，并行

```

**为何阈值很重要**：在统一模式下，智能体可以看到根目录 `CLAUDE.md` 和子目录 `CLAUDE.md` 之间的矛盾。在并行模式下，每个智能体只看到一个文件，无法检测到这些矛盾。阈值为较小的仓库保留了跨文件智能，同时对较大的仓库保持在实际上下文限制内。

**阈值校准**：10 万 Token（词元）为评估器提示词（通常 2-5K）和模型响应缓冲区留有余地。对于你自己的用例，将阈值设置为 `model_context_window - evaluator_prompt_tokens - response_buffer`。Packmind 的 `DEFAULT_MAX_UNIFIED_TOKENS = 100_000` 对于大多数当前模型是保守的默认值。

**Token（词元）估算**：不需要精确计数。粗略估算（大多数英文文本用 `chars / 4`）足以做出模式决策。偶尔在估算上偏差 10% 的成本，远低于获得额外精确度的成本。

> 在 [PackmindHub/context-evaluator](https://github.com/PackmindHub/context-evaluator)（`src/shared/evaluation/runner.ts` 中的 `canUseUnifiedMode()`，MIT）中观察到。参见「致谢」。

---

## 参见

- 开发方法论：TDD（测试驱动开发）、SDD（规范驱动开发）、BDD（行为驱动开发）、多智能体编排
- 智能体团队：包括怀疑审查者模式（Skeptical Reviewer Pattern）的智能体团队
- `examples/skills/mcp-integration-reference/`：MCP 参考文件模式模板

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/F5Lbwgr8siCF5hk9dfbcwlUzn2W> ｜ 归档：2026-06-04
