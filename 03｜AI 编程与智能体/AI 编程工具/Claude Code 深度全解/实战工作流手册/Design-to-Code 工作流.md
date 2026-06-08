# Design-to-Code 工作流

使用 Figma MCP 服务器的自动化设计系统落地，使产品设计师能够将生产就绪的规范交接给 Claude Code，由其实现组件并保持 1:1 的设计-代码保真度。

---

## 目录

1. TL;DR
2. 已记录的影响
3. 架构概述
4. 三层 Token 层次结构
5. 前提条件
6. 核心工作流
7. Code Connect 配置
8. 示例提示词
9. 团队采用模式
10. 反模式
11. 实施路线图
12. 资源

---

## TL;DR

```Plain Text
设计师（Figma Make）→ 导出（Figma Design）→ Claude（Figma MCP）→ 生产代码

核心洞见：设计系统 = 真理来源
Claude 直接从 Figma 消费 Token/组件
实现自动保持设计保真度

```

---

## 已记录的影响

基于 2026 年 1 月的生产案例研究：

| 指标 | 改善 | 来源 |
|-|-|-|
| 设计不一致性 | 减少 62% | Parallel HQ 研究 |
| 工作流效率 | 提升 78% | builder.io 案例研究 |
| 节省的工程时间 | 75 天（6 个月） | Parallel HQ 生产数据 |
| 上市时间 | 缩短 56% | 多组织综合数据 |
| 设计技术债 | 减少 82% | 实施后审计 |

**典型工作流耗时**：

- 单帧 → 生产组件：2-3 分钟
- 设计系统漂移审计：3 周 → 3 分钟
- Token 更新传播：数小时手动 → 秒级自动化

*来源：builder.io/blog/claude-code-figma-mcp-server, parallelhq.com/blog/automating-design-systems-with-ai, composio.dev/blog/how-to-use-figma-mcp-with-claude-code*

---

## 架构概述

### 全栈架构

```Plain Text
[Figma 设计文件]
    ↓ （变量与样式）
[Tokens Studio 插件]（可选但推荐）
    ↓ （JSON 导出）
[GitHub 仓库]
    ↓ （CI/CD）
[Style Dictionary]
    ↓ （转换）
[CSS 自定义属性 / Tailwind 配置]
    ↓ （被消费）
[组件库]
    ↑ （通过读取）
[Claude Code + Figma MCP]

```

### MCP 集成点

Claude Code 通过 Figma MCP 服务器访问 Figma：

```Plain Text
Claude Code
    ↓ （使用）
Figma MCP 服务器（mcp-server-figma）
    ↓ （通过认证）
Figma 个人访问令牌
    ↓ （读取）
Figma 文件（Dev Mode 数据）

```

**Claude 可以访问的内容**：

- 文件结构和帧
- 颜色/文字/效果样式
- 组件属性
- 变量（Token）
- Dev Mode 注解
- Code Connect 代码片段（若已配置）

**Claude 无法访问的内容**：

- 无令牌权限的私有文件
- 编辑功能（只读）
- 实时协作数据
- 版本历史（仅当前状态）

---

## 三层 Token 层次结构

现代设计系统使用层次化的 Token 结构。Claude Code 在消费 Figma 数据时理解这种层次结构。

| 层级 | 定义 | Figma 实现 | 代码输出 |
|-|-|-|-|
| **基础层** | 原始值 | Figma 变量（如 `blue-600: #0066CC`、`spacing-2: 8px`） | CSS 自定义属性（`--blue-600`、`--spacing-2`） |
| **复合层** | 组合的原始值 | 引用变量的组件填充 | Tailwind 配置或 CSS 类 |
| **语义层** | 上下文含义 | 上下文变量别名（如 `color-interactive-primary` → `blue-600`） | 组件 props 或主题 Token |

### 层次结构示例

```Plain Text
基础层：
  --color-blue-600: #0066CC
  --spacing-2: 8px
  --radius-md: 4px

复合层：
  --button-padding: var(--spacing-2) var(--spacing-4)
  --button-border-radius: var(--radius-md)

语义层：
  --interactive-primary: var(--color-blue-600)
  --interactive-primary-hover: var(--color-blue-700)

```

**Claude Code 的行为**：给定一个 Figma 组件时，Claude：

1. 提取引用的变量（基础层）
2. 识别复合模式（间距、尺寸）
3. 应用你的 Token 约定中的语义命名
4. 生成匹配此层次结构的代码

---

## 前提条件

### 设计师所需

| 需求 | 详情 |
|-|-|
| **Figma 许可证** | Dev Mode 席位（启用变量检查、代码片段） |
| **有组织的变量** | 使用 Figma 变量或 Tokens Studio 插件进行 Token 管理 |
| **组件结构** | 自动布局、命名图层、一致的命名约定 |
| **帧命名** | 描述性的帧名称（Claude 用这些名称命名组件） |

### 开发者所需

| 需求 | 详情 |
|-|-|
| **Claude Code** | 版本 1.5.0+（MCP 支持） |
| **Figma MCP 服务器** | `npm install -g @modelcontextprotocol/server-figma` |
| **个人访问令牌** | 从 Figma 账号设置 → Token 中生成 |
| **MCP 配置** | 在 Claude Code 设置中配置令牌 |

### MCP 配置

添加到你的 Claude Code MCP 设置（`.claude/mcp.json` 或设置 UI）：

```JSON
{
  "mcpServers": {
    "figma": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-figma"],
      "env": {
        "FIGMA_PERSONAL_ACCESS_TOKEN": "your-token-here"
      }
    }
  }
}

```

**安全说明**：生产环境使用环境变量：

```JSON
{
  "env": {
    "FIGMA_PERSONAL_ACCESS_TOKEN": "${FIGMA_TOKEN}"
  }
}

```

然后在 shell 中导出：`export FIGMA_TOKEN="figd_..."`

---

## 核心工作流

### 工作流 A：单帧 → 生产组件

**耗时**：每个组件 2-3 分钟

**步骤**：

1. **设计师**：在 Figma 中用适当的变量/样式创建组件
2. **设计师**：与开发/Claude 分享 Figma 文件 URL
3. **开发者**：向 Claude Code 发出提示：

```Plain Text
从 Figma 文件读取"Button/Primary"组件：
https://www.figma.com/design/FILE_KEY

实现为带 TypeScript 的 React 组件。
使用 Tailwind 进行样式设计，将 Figma 变量映射到我们的设计 Token。
确保响应式行为与 Figma 的自动布局约束一致。

```

1. **Claude**：

   - 通过 Figma MCP 获取组件
   - 提取样式、尺寸、间距
   - 将变量映射到代码 Token
   - 生成 props 与 Figma 变体匹配的组件
2. **验证**：`npm run dev` → 与 Figma 进行视觉对比

**示例输出**：

```Plain Text
// components/Button/Primary.tsx
interface ButtonProps {
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  children: React.ReactNode;
}

export function PrimaryButton({ size = 'md', disabled, children }: ButtonProps) {
  return (
    <button
      className={cn(
        'rounded-md font-medium transition-colors',
        'bg-interactive-primary text-white',
        'hover:bg-interactive-primary-hover',
        'disabled:opacity-50 disabled:cursor-not-allowed',
        {
          'px-3 py-1.5 text-sm': size === 'sm',
          'px-4 py-2 text-base': size === 'md',
          'px-6 py-3 text-lg': size === 'lg',
        }
      )}
      disabled={disabled}
    >
      {children}
    </button>
  );
}

```

---

### 工作流 B：设计系统漂移审计

**耗时**：3 周手动审查 → 3 分钟自动化

**问题**：随着时间推移，代码与设计系统产生偏差（魔法数字、硬编码颜色、间距不一致）。

**解决方案**：Claude 根据 Figma 真理来源审计代码库。

**提示词**：

```Plain Text
审计 src/components 中的设计系统合规性。
对照我们的 Figma 设计系统进行比较：
https://www.figma.com/design/FILE_KEY

报告：
1. 未使用设计 Token 的硬编码颜色
2. 魔法数字间距值
3. 与 Figma 结构不匹配的组件
4. 缺少的响应式模式

然后提出带 Token 替换的修复方案。

```

**Claude 输出**：

```Markdown
## 设计系统审计结果

### 发现的问题（共 23 个）

#### 硬编码颜色（8 处）
- `src/components/Card.tsx:45` → `#0066CC` 应为 `var(--interactive-primary)`
- `src/components/Header.tsx:12` → `#F3F4F6` 应为 `var(--surface-secondary)`
...

#### 魔法数字（11 处）
- `src/components/Modal.tsx:23` → `padding: 16px 24px` 应为 `var(--spacing-4) var(--spacing-6)`
...

#### 结构不匹配（4 处）
- `src/components/Button.tsx` → 缺少 Figma 中存在的 `icon-left` 变体
...

```

---

### 工作流 C：Token 自动化流水线

**目标**：Figma 变量变更自动传播到代码。

**架构**：

```Plain Text
Figma 变量
    ↓ （Tokens Studio 导出或 Figma API）
GitHub 仓库（tokens.json）
    ↓ （GitHub Actions CI/CD）
Style Dictionary 转换
    ↓ （生成）
CSS / Tailwind / 平台特定 Token
    ↓ （提交并部署）
生产环境

```

**配置**（一次性）：

1. **Tokens Studio 插件**：连接到 GitHub 仓库
2. **Style Dictionary 配置**：定义转换规则
3. **GitHub Actions**：Token 更新时自动运行
4. **Claude 角色**：审查并验证生成的 Token

**开发者提示词**（CI 运行后）：

```Plain Text
审查来自提交 abc1234 的 Token 更新。
检查是否有组件需要更新以使用新 Token。
若存在破坏性变更，生成迁移指南。

```

**Claude 输出**：

```Markdown
## Token 更新审查（v2.3.0 → v2.4.0）

### 变更
- 新增：`--spacing-7`、`--spacing-8`（设计方请求）
- 修改：`--interactive-secondary` 色相偏移 5°（品牌刷新）
- 废弃：`--legacy-blue`（Q3 前移除）

### 影响分析
- 12 个组件引用了 `--interactive-secondary` → 通过 Token 引用自动更新
- 3 个组件使用了废弃的 `--legacy-blue` → 需要迁移

### 需要迁移
1. `src/components/LegacyButton.tsx:34` → 将 `--legacy-blue` 替换为 `--interactive-primary`
2. `src/components/OldCard.tsx:67` → 替换为新 Token
3. `src/utils/theme.ts:12` → 更新主题导出

### 迁移脚本
[Claude 生成代码修改工具（codemod）或查找/替换脚本]

```

---

### 工作流 D：视觉迭代循环（Figma + Playwright）

**目标**：针对 Figma 设计的自动化视觉回归测试。

**MCP 栈**：Figma MCP + Playwright MCP

**配置**：

```Plain Text
Claude Code 访问：
- Figma（设计真理来源）
- Playwright（自动化浏览器测试）

```

**提示词**：

```Plain Text
对所有变体截图我们的 Button 组件。
与以下 Figma 帧进行对比：
https://www.figma.com/design/FILE_KEY → "Button Tests"页面

报告任何视觉差异（颜色、间距、排版）。

```

**工作流**：

1. Claude 读取 Figma 帧（期望状态）
2. Claude 使用 Playwright 截图实时组件（实际状态）
3. Claude 比较（像素差异对比或视觉检查）
4. 报告差异并提出修复建议

**示例输出**：

```Markdown
## 视觉回归报告

### ✅ 匹配（5/7）
- Button/Primary/Default
- Button/Primary/Hover
- Button/Secondary/Default
...

### ❌ 不匹配（2/7）

#### Button/Primary/Disabled
- **问题**：代码中文字透明度 0.4，Figma 中为 0.5
- **修复**：将 `disabled:opacity-50` 改为 `disabled:opacity-40`
- **文件**：`src/components/Button.tsx:23`

#### Button/Large
- **问题**：代码中 padding 为 12px，Figma 中为 16px
- **修复**：将 `py-3` 改为 `py-4`（16px）
- **文件**：`src/components/Button.tsx:18`

```

---

## Code Connect 配置

**Code Connect** 是 Figma 的无代码工具，用于将设计组件链接到代码片段。这增强了 Claude 生成正确代码的能力。

### 它的作用

- 设计师为 Figma 组件添加代码示例注解
- Claude 通过 MCP 读取这些注解
- 生成的代码自动匹配团队约定

### 配置步骤（针对设计师）

1. 在 Figma Dev Mode → 选择组件 → Code Connect 面板
2. 添加展示组件使用方式的代码片段：

```Plain Text
// Figma 中的 Code Connect 注解示例
<Button variant="primary" size="lg">
  点击我
</Button>

```

1. Claude 在被要求实现时会看到这个，并使用团队的精确模式

### 优势

| 无 Code Connect | 有 Code Connect |
|-|-|
| Claude 生成通用代码 | Claude 使用团队约定 |
| prop 命名不一致 | props 与 Figma 变体完全匹配 |
| 需要手动修正 | 第一次就生产就绪 |

**参考**：更多信息请阅读 parallelhq.com/blog（Code Connect UI 文章）

---

## 替代方案：Pencil（IDE 原生画布）

**概述**：[Pencil](https://pencil.dev) 将无限设计画布直接带入 Claude Code/Cursor/VSCode，消除外部工具切换，实现设计即代码工作流。

### 架构

**核心创新**：与 Figma（云端）或 Excalidraw（独立）不同，Pencil 将设计画布直接嵌入你的 IDE——Claude 和你的代码就在那里。

```Plain Text
传统工作流：
Figma（设计）→ 导出 → Claude Code → 实现 → 手动同步

Pencil 工作流：
IDE 画布（设计 + AI 智能体 + 代码）→ Git 提交 → 持续对齐

```

**主要功能**：

- **WebGL 画布**：无限、高性能、完全可编辑
- **AI 多人协作智能体**：并行智能体协作处理设计
- **Git 原生**：`.pen` 文件（JSON 格式）与代码并排版本控制
- **MCP 双向**：完整的读写访问（不像 Figma MCP 只读）
- **Figma 导入**：直接从 Figma 复制粘贴，保留矢量和样式

### Pencil vs Figma MCP

| 维度 | Pencil | Figma MCP |
|-|-|-|
| **位置** | IDE 原生（Cursor/VSCode/Claude Code） | 外部云端 |
| **格式** | `.pen` JSON（开放） | 专有二进制 |
| **版本控制** | Git 原生（分支/合并/历史） | Figma 云端版本 |
| **AI 智能体** | 多人并行 | 通过 MCP 单线程 |
| **协作** | 代码优先（开发者 + 设计师） | 设计优先（设计师 + 开发者） |
| **MCP 访问** | 双向（读写） | 只读 |
| **工作流** | 在同一环境中设计 → 提交 → 代码 | 设计 → 导出 → 交接 → 代码 |
| **最适合** | 工程师-设计师、代码中心团队 | 传统的设计-开发分离 |
| **成熟度** | 新兴（2026 年 1 月发布） | 成熟（2024 年+） |
| **定价** | 目前免费，未来待定 | Freemium（有免费层） |

### 何时使用 Pencil

✅ **适合**：

- 团队以 Cursor 或 VSCode + Claude Code 为主要环境
- 熟悉终端/IDE 工作流的工程师-设计师
- 需要紧密设计-代码对齐的项目（设计即代码范式）
- 希望使用 Git 原生设计版本控制（分支保护、回滚等）
- 想利用并行 AI 智能体进行设计自动化

⚠️ **谨慎考虑**：

- 传统设计团队（非技术型）→ Figma 可能更好
- 需要企业 SLA/支持 → Pencil 仍在成熟
- 复杂设计系统（50+ 组件）→ Figma 生态更成熟
- 团队不使用 Cursor/VSCode → 兼容性有限

### 配置

1. **安装 Pencil 扩展**：

   - 访问 [pencil.dev](https://pencil.dev)
   - 按照 Cursor/VSCode/Claude Code 的安装说明操作
   - 创建账号（目前免费）
2. **创建第一个画布**：
3. **Git 工作流**：
4. **Claude 集成**：

   - Claude 可以通过 MCP 读取 .pen 文件
   - 提示词："从 design/components.pen 实现 Button 组件"
   - Claude 提取设计规格并生成代码

### 示例提示词

```Plain Text
从 design/homepage.pen 读取"Hero Section"。

实现为 React 组件：
- 响应式行为与画布断点一致
- 来自设计的动画（淡入、上滑）
- 完全按照画布中指定的文案
- 使用 Tailwind 进行样式设计

确保与设计规格完全一致。

```

### 创始人与背景

**Tom Krcha**（CEO，Pencil）：

- Adobe XD 联合创始人（2014-2018），在 Adobe 工作 10 年
- 此前退出：Alter Avatars（被 Google 收购）、Around（被 Miro 收购）
- 14 年以上开发经验

**融资**：a16z Speedrun（约 100 万美元）+ KAYA VC

**牵引力**：发布时超过 100 万次浏览，数千名注册用户，包括 Microsoft、Shopify、Uber 的高管。

### 成熟度说明

**⚠️ 状态**：2026 年 1 月发布（非常新）。早期信号强劲，但文档和生态系统仍在成熟。

**建议**：

- **生产项目**：先用 1-2 个非关键功能试点
- **新项目**：对于 Cursor/Claude Code 团队，可以安全采用
- **传统工作流**：等 Pencil 成熟（3-6 个月）后再从 Figma MCP 切换

**关注**：定价公告、公开 GitHub 仓库，2026 年 Q2 预计发布成熟文档。

---

## 示例提示词

### 组件实现

```Plain Text
从我们的 Figma 设计系统实现"Card/Product"组件：
[Figma URL]

要求：
- 使用 tailwind.config.ts 中现有的设计 Token
- 包含与 Figma 交互一致的悬停状态
- 实现所有变体（默认、精选、紧凑）
- 为所有 props 添加 TypeScript 类型

```

### 设计系统扩展

```Plain Text
我们的设计团队在 Figma 中添加了新的"Badge"组件：
[Figma URL → Badge 帧]

生成：
1. 包含所有变体的 React 组件
2. Storybook stories
3. props 组合的单元测试
4. 更新设计系统文档

```

### Token 验证

```Plain Text
将我们 Tailwind 配置中的颜色 Token 与以下 Figma 变量进行对比：
[Figma URL]

报告任何不匹配并生成更新脚本。

```

### 响应式实现

```Plain Text
从 Figma 实现"Hero"部分，带有精确的响应式行为：
[Figma URL → Hero/Responsive 帧]

Figma 已配置 3 个断点。精确匹配。

```

### 可访问性审计

```Plain Text
对照 Figma 规范审查"Modal"组件实现：
[Figma URL]

检查：
- 焦点管理与 Figma 的交互流一致
- 颜色对比度达到 WCAG AA（Figma 有对比度检查器）
- 键盘导航（Figma 注解指定 tab 顺序）

```

### 交接前设计质量检查

```Plain Text
审查"Checkout Flow"帧的实现就绪情况：
[Figma URL → Checkout Flow 页面]

检查：
- 所有交互状态已定义（悬停、焦点、禁用、错误）
- 变量使用一致（无魔法值）
- 自动布局约束可实现
- 生产代码还缺少什么？

```

### 多组件原子化实现

```Plain Text
按顺序实现原子设计系统组件：

1. 原子层：[Figma URL → Atoms 页面]
   - Button、Input、Label、Badge

2. 分子层：[Figma URL → Molecules 页面]
   - FormField（Label + Input + Error）
   - SearchBar（Input + Button）

3. 生物层：[Figma URL → Organisms 页面]
   - LoginForm（使用分子层）

确保每一层只从更低的层导入。

```

---

## 团队采用模式

### 对于产品设计师

**新工作流**：

1. 在 Figma 中设计，使用正确的变量结构
2. 使用 Figma Make 进行快速原型
3. 启用 Dev Mode 导出到 Figma Design
4. 与开发团队分享文件 URL + 特定帧
5. Claude 消费设计 → 生成实现
6. 设计师通过视觉对比审查代码输出（不需要读代码）

**核心洞见**：设计师不需要学习代码。他们通过与 Figma 的视觉对比来审查实现。

### 对于开发者

**新工作流**：

1. 从设计师处接收 Figma URL
2. 提示 Claude 从 Figma 源实现
3. 审查生成的代码是否符合架构
4. 运行视觉对比（Playwright 或手动）
5. 提交生产就绪组件

**节省的时间**：跳过手动像素级完美实现。专注于逻辑，而不是布局匹配。

### 对于产品经理

**新能力**：根据 Figma 帧请求设计实现估算。

**PM 提示词**：

```Plain Text
审查"Dashboard Redesign" Figma 文件：
[Figma URL]

估算实现复杂度：
- 需要多少个新组件？
- 哪些现有组件可以复用？
- 有什么技术阻碍？

提供开发实现的大致时间线。

```

Claude 输出：

```Markdown
## 实现分析

### 范围
- 共 12 帧
- 4 个新组件（DataTable、MetricCard、FilterPanel、DateRangePicker）
- 8 个现有组件可复用

### 复杂度评估
- **低**：MetricCard（与现有 Card 类似，1-2 小时）
- **中**：FilterPanel（多选逻辑，4-6 小时）
- **高**：DataTable（排序、分页、虚拟化，2-3 天）
- **高**：DateRangePicker（第三方库集成，1-2 天）

### 技术注意事项
- DataTable 需要后端 API 支持服务端分页
- DateRangePicker：评估 date-fns vs dayjs vs 原生
- FilterPanel 状态管理（本地 vs 全局）

### 估算时间线
- 开发：5-7 天
- 代码审查 + QA：2 天
- 合计：1.5-2 周

```

---

## 反模式

| ❌ 反模式 | 为何失败 | ✅ 正确方式 |
|-|-|-|
| **手动设计转录** | 易出错、耗时、漂移不可避免 | 让 Claude 通过 MCP 直接读取 Figma |
| **截图作为规范** | 无 Token 数据、无交互性、有歧义 | 分享 Figma URL，让 Claude 访问结构化数据 |
| **硬编码值** | 设计系统更新时会失效 | 使用来自 Figma 变量的设计 Token |
| **设计师写代码** | 对设计师技能的低效利用 | 设计师设计 → Claude 编码 → 开发者审查 |
| **开发者猜测间距** | 与设计系统不一致 | Claude 从 Figma 提取精确值 |
| **无 Code Connect 注解** | 输出通用代码 | 注解一次 → Claude 使用团队约定 |
| **跳过视觉对比** | 实现漂移 | 始终对照 Figma 源验证 |
| **Token 命名不匹配** | Figma 变量 ≠ 代码 Token | 建立命名约定，使用 Style Dictionary |
| **缺少响应式规范** | 开发者猜测断点 | Figma 有响应式帧 → Claude 读取精确规格 |
| **单层 Token** | 灵活性差，难以主题化 | 使用三层层次结构（基础/复合/语义） |

---

## 实施路线图

### 第一阶段：基础建设（第 1-2 周）

**目标**：基础的 Figma → Claude → 代码流水线

- [ ] 安装 Figma MCP 服务器

- [ ] 配置个人访问令牌

- [ ] 测试连接：Claude 读取公开的 Figma 文件

- [ ] 在项目 CLAUDE.md 中创建设计系统约定

- [ ] 实现 3-5 个简单组件（Button、Input、Badge）

- [ ] 建立视觉 QA 流程（手动对比）

**成功标准**：

- Claude 从 Figma URL 生成组件
- 输出在视觉上与设计一致
- 开发团队理解工作流

---

### 第二阶段：规模化（第 3-4 周）

**目标**：完整设计系统实现 + 自动化

- [ ] 从 Figma 库实现 20+ 个组件

- [ ] 设置 Token 自动化（Tokens Studio + Style Dictionary）

- [ ] 创建组件测试套件（Storybook + 视觉回归）

- [ ] 培训设计师进行变量规范化

- [ ] 在 CLAUDE.md 中记录团队约定

- [ ] 运行第一次设计系统漂移审计

**成功标准**：

- 80%+ 的 UI 组件自动生成
- Token 更新自动传播
- 设计师对交接流程有信心

---

### 第三阶段：编排（第 5 周+）

**目标**：多 MCP 工作流 + 持续同步

- [ ] 集成 Playwright MCP 进行自动化视觉测试

- [ ] 设置 CI/CD 进行设计-代码保真度检查

- [ ] 创建 Figma → GitHub → 生产流水线

- [ ] 实施设计系统治理（lint、审计）

- [ ] 允许非开发者触发 Claude 实现（工单、Slack）

- [ ] 衡量指标（上市时间、不一致率、节省的开发时间）

**成功标准**：

- 设计更新 → 生产在 1 天内
- 零手动设计转录
- 可衡量的团队速度提升

---

## 资源

### 官方文档

- **Figma MCP 服务器**：[@modelcontextprotocol/server-figma](https://github.com/modelcontextprotocol/servers/tree/main/src/figma)（GitHub）
- **Figma 开发者文档**：[figma.com/developers](https://www.figma.com/developers)
- **Style Dictionary**：[amzn.github.io/style-dictionary](https://amzn.github.io/style-dictionary/)
- **Tokens Studio 插件**：[tokens.studio](https://tokens.studio/)

### 案例研究与教程

- **builder.io**："Claude Code + Figma MCP 服务器：AI 设计到代码工作流"（2026 年 1 月）

  - [builder.io/blog/claude-code-figma-mcp-server](https://www.builder.io/blog/claude-code-figma-mcp-server)
  - 生产指标、工作流示例
- **Vladimir Siedykh**："使用 Claude Code 的多 MCP 编排"

  - [vladimirsiedykh.com/blog/claude-code-mcp-workflow](https://vladimirsiedykh.com/)
  - Figma + Playwright + Linear 集成
- **Parallel HQ**："用 AI 自动化设计系统"

  - [parallelhq.com/blog/automating-design-systems-with-ai](https://parallelhq.com/)
  - 节省 75 天，Code Connect UI 指南
- **Composio**："如何在 Claude Code 中使用 Figma MCP"

  - [composio.dev/blog/how-to-use-figma-mcp-with-claude-code](https://composio.dev/)
  - Token 层次模式、配置指南

### 社区资源

- **Figma 社区**：搜索"Design System Tokens"查找入门模板
- **MCP 注册表**：[mcp.run](https://mcp.run/) → Figma 服务器示例
- **Discord**：Anthropic Discord → #mcp-servers 频道

### 相关工作流

- 处理图片与截图 — Claude Code 图片分析
- ASCII 艺术与线框图 — 低保真设计迭代
- Playwright MCP — 视觉回归测试

---

## 另请参阅

- Figma MCP 部分 — 主指南 Figma MCP 章节
- 「examples/claude-md/product-designer.md」 — 产品设计师 CLAUDE.md 模板
- 「../cheatsheet.md」 — 快速参考

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/GnnPwVIaRiyFeyknIincL7ehnze> ｜ 归档：2026-06-04
