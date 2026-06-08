# 生成 PDF

使用具有现代排版和设计的 Claude Code，生成专业 PDF（文档、白皮书、报告）。

---

## 目录

1. TL;DR
2. 适用场景
3. 技术栈概览
4. 配置
5. 工作流
6. 与 Claude Code 集成
7. 自定义
8. 故障排查
9. 延伸阅读

---

## TL;DR

```Bash
# 安装
brew install quarto  # macOS

# 生成
quarto render document.qmd  # → document.pdf

# 预览
quarto preview document.qmd  # 热重载

```

**技术栈**：Quarto（编排）+ Typst（排版）+ Pandoc（Markdown）

---

## 适用场景

| 使用场景 | 适合度 | 替代方案 |
|-|-|-|
| 技术文档 | ✅ | — |
| 白皮书 / 报告 | ✅ | — |
| API 文档 | ⚠️ | OpenAPI + Redoc |
| 幻灯片 / 演示文稿 | ⚠️ | Quarto Revealjs |
| 快速笔记 | ❌ | 纯 Markdown |
| 协作编辑 | ❌ | Google Docs、Notion |

**最适合**：需要专业排版、版本控制和可复现性的长篇技术内容。

---

## 技术栈概览

```Plain Text
┌─────────────────────────────────────────────────┐
│               你的 .qmd 文件                    │
│       （Markdown + YAML 前置元数据）            │
└─────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────┐
│                   Quarto                        │
│             （文档渲染引擎）                    │
│         • 处理 YAML 元数据                      │
│         • 管理扩展                              │
│         • 管理输出格式                          │
└─────────────────────────────────────────────────┘
                        │
          ┌─────────────┴─────────────┐
          ▼                           ▼
┌─────────────────────┐    ┌─────────────────────┐
│       Pandoc        │    │       Typst         │
│   （MD → AST → ?）  │    │  （排版/PDF）       │
│  • Markdown 解析器  │    │  • 现代引擎         │
│  • AST 转换         │    │  • 快速编译         │
│  • 格式桥接         │    │  • 无需 LaTeX       │
└─────────────────────┘    └─────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────┐
│                 document.pdf                    │
│          （专业排版输出）                       │
└─────────────────────────────────────────────────┘

```

### 输出格式与命令

```Plain Text
  格式                  命令                          输出
  ──────                ────────                      ──────

  标准 PDF        →  quarto render doc.qmd            doc.pdf
                     --to typst                       （无自定义模板）

  样式化 PDF ✅   →  quarto render doc.qmd            doc.pdf
                     --to whitepaper-typst            (~270K–1.7M, Bold Guy)
                     （通过 _extensions/ 自定义格式）

  EPUB            →  quarto render doc.qmd            doc.epub
                     --to epub

  预览            →  quarto preview doc.qmd           浏览器热重载

```

### 扩展结构

```Plain Text
  _extensions/
  └── whitepaper/
      ├── _extension.yml       ← 声明 "whitepaper-typst" 格式
      ├── typst-template.typ   ← 设计系统（颜色、字体、标注框）
      └── typst-show.typ       ← Quarto → Typst 桥接

  ⚠️  若在 fr/ en/ 和根目录维护多份副本：
      保持 3 个 typst-template.typ 文件同步

```

### 快速故障排查

````Plain Text
  现象                              原因                     修复
  ────────                          ─────                    ───
  PDF 很小 (~80-190K)，无样式       使用了 --to pdf          改用 --to whitepaper-typst
                                    而非 --to whitepaper-typst

  "bibliography" 报错               标注框标题中有 @ref       删除标题中的 @
                                    → 被解释为引用

  表格被渲染为代码                  反引号 ``` 未关闭         检查 ``` 数量（必须成对）

  "Extension not found"             目录错误                  检查 _extensions/ 路径

````

| 组件 | 版本 | 作用 |
|-|-|-|
| **Quarto** | ≥1.4.0 | 编排、扩展、多格式输出 |
| **Typst** | 0.13.0 | 现代排版（替代 LaTeX） |
| **Pandoc** | 3.x | Markdown 解析（随 Quarto 捆绑） |

---

## 配置

### 安装

**macOS**：

```Bash
brew install quarto

```

**Linux（Debian/Ubuntu）**：

```Bash
wget https://github.com/quarto-dev/quarto-cli/releases/download/v1.4.555/quarto-1.4.555-linux-amd64.deb
sudo dpkg -i quarto-1.4.555-linux-amd64.deb

```

**Windows**：

```PowerShell
winget install Posit.Quarto

```

**验证**：

```Bash
quarto --version  # 应为 ≥1.4.0

```

### 项目结构

```Plain Text
project/
├── _extensions/           # Quarto 扩展（模板）
│   └── custom-template/
│       ├── _extension.yml
│       ├── typst-template.typ
│       └── typst-show.typ
├── documents/
│   ├── guide.qmd          # 源文件
│   └── guide.pdf          # 生成的输出
└── assets/
    └── logo.png           # 共享资源

```

### 最小文档

创建 `document.qmd`：

````YAML
---
title: "我的文档"
author: "作者姓名"
date: 2026-01-17
format:
  typst:
    toc: true
lang: zh
---

# 介绍

在此处填写你的内容……

## 第 1 节

更多内容，包含**粗体**和 `代码`。

```bash
echo "代码块正常工作！"

````

## 第 2 节

| 列 A | 列 B |
|-|-|
| 数据 1 | 数据 2 |

````Plain Text

生成：
```bash
quarto render document.qmd  # 创建 document.pdf

````

---

## 工作流

### 1. 内容优先方法

```Plain Text
1. 在 Markdown（.qmd）中编写内容
2. 添加 YAML 前置元数据
3. 使用热重载预览
4. 生成最终 PDF
5. 对源文件和 PDF 进行版本控制

```

### 2. 可用的 YAML 参数

| 参数 | 类型 | 描述 | 示例 |
|-|-|-|-|
| `title` | 字符串 | 主标题 | `"技术指南"` |
| `subtitle` | 字符串 | 副标题 | `"v2.0 版"` |
| `author` | 字符串/数组 | 作者 | `"张三"` |
| `date` | 日期 | 文档日期 | `2026-01-17` |
| `date-format` | 字符串 | 显示格式 | `"MMMM YYYY"` |
| `toc` | 布尔值 | 目录 | `true` |
| `toc-depth` | 数字 | 目录层级（1-3） | `2` |
| `lang` | 字符串 | 语言 | `zh` 或 `en` |
| `section-numbering` | 字符串 | 编号格式 | `"1.1"` |

### 3. Markdown 功能

**分页符**：

```Markdown
{{< pagebreak >}}

```

**代码块**（带语法高亮）：

````Markdown
```typescript
function hello(): string {
  return "world";
}
```

````

**表格**：

```Markdown
| 功能 | 支持 |
|---------|-----------|
| 表格  | ✅        |
| 图片  | ✅        |
| 链接  | ✅        |

```

**图片**：

```Markdown
![替代文本](path/to/image.png){width=50%}

```

---

## 与 Claude Code 集成

### 使用 pdf-generator Skills（技能模块）

调用 Skills（技能模块）进行引导式 PDF 生成：

```Plain Text
/pdf-generator

```

该 Skills（技能模块）提供：

- 带 YAML 前置元数据的模板
- 设计系统配置
- 常见故障排查修复
- 生成命令

### 提示词示例

**生成文档**：

```Plain Text
将我们的 API 创建为 Quarto 文档形式的技术指南。
使用带目录的 Typst 格式。
包含以下章节：认证、端点、错误代码。

```

**转换现有 Markdown**：

```Plain Text
将 README.md 转换为专业 PDF。
添加带标题和日期的封面页。
使用 Quarto/Typst 格式。

```

**创建模板**：

```Plain Text
为我们公司的文档样式创建一个 Quarto 扩展：
- 页眉中的 Logo
- 自定义颜色：主色 #0f172a，强调色 #6366f1
- 正文使用 Inter 字体，代码使用 JetBrains Mono

```

### 配合计划模式

用于复杂文档：

```Plain Text
[按 Shift+Tab 进入计划模式]

我需要创建一系列 5 份技术白皮书。
规划结构：
1. 通用模板/扩展
2. 共享资源
3. 构建自动化
4. 版本管理

```

### 配合 Hooks（钩子）

使用工具后钩子在编辑后自动生成 PDF：

```JSON
// 在 .claude/settings.json 中
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "command": "if echo \"$TOOL_INPUT\" | grep -q '.qmd'; then quarto render \"$FILE\"; fi"
      }
    ]
  }
}

```

---

## 自定义

### 自定义模板扩展

创建 `_extensions/mytemplate/_extension.yml`：

```YAML
title: 我的模板
author: 你的名字
version: 1.0.0
contributes:
  formats:
    typst:
      template: typst-template.typ
      template-partials:
        - typst-show.typ

```

### Typst 模板变量

在 `typst-template.typ` 中：

```Plain Text
// 颜色
#let primary = rgb("#0f172a")      // 深色文字
#let secondary = rgb("#334155")    // 浅色文字
#let accent = rgb("#6366f1")       // 高亮

// 排版
#set text(
  font: ("Inter", "Helvetica Neue", "Arial"),
  size: 11pt,
)

#set par(
  leading: 0.75em,  // 行高
  justify: true,
)

// 代码块
#show raw.where(block: true): it => {
  block(
    fill: rgb("#f8fafc"),
    stroke: (left: 3pt + accent),
    inset: 10pt,
    radius: 4pt,
    it,
  )
}

```

### 标注框

在模板中定义：

```Plain Text
#let info(title: "注意", body) = {
  block(
    fill: rgb("#E0F2FE"),
    stroke: (left: 3pt + rgb("#0284C7")),
    inset: 12pt,
    radius: 4pt,
    [*#title*: #body]
  )
}

#let warning(title: "警告", body) = { ... }
#let success(title: "成功", body) = { ... }
#let danger(title: "危险", body) = { ... }

```

在文档中使用：

```Plain Text
#info[这是一条信息提示。]
#warning(title: "注意")[请检查你的配置。]

```

---

## 故障排查

### 快速检查

````Bash
# 验证 Quarto
quarto --version

# 检查扩展是否存在
ls _extensions/*/

# 验证代码块是否成对（必须为偶数）
grep -c '^```' document.qmd

# 检查编码
file -i document.qmd  # 应显示 utf-8

````

### 常见问题

| 问题 | 现象 | 修复 |
|-|-|-|
| 嵌套代码块 | 内容从块中逃逸 | 外层块使用 4+ 个反引号 |
| 表格变成代码 | 灰色背景 | 检查上方未匹配的 ```` ``` ```` |
| 扩展缺失 | "Extension not found" | 验证 `_extensions/` 路径 |
| 字体警告 | "unknown font family" | 正常；使用备用字体 |
| 特殊字符乱码 | `?` 或乱码 | 转换为 UTF-8 |

### 嵌套代码块

**问题**：内层代码块提前关闭外层块。

**解决方案**：外层块使用更多反引号：

`````Markdown
````markdown
# 外层块使用 4 个反引号

```bash
echo "内层块使用 3 个反引号"
```

外层块继续……
````

`````

### 验证脚本

````Bash
#!/bin/bash
# validate-qmd.sh

for f in *.qmd; do
  count=$(grep -c '^```' "$f")
  if [ $((count % 2)) -ne 0 ]; then
    echo "错误：$f 的代码块数量为奇数 ($count)"
  fi
done

````

---

## 延伸阅读

- [Quarto 文档](https://quarto.org/docs/guide/)
- [Typst 文档](https://typst.app/docs/)
- [Quarto + Typst 指南](https://quarto.org/docs/output-formats/typst.html)
- 「examples/skills/pdf-generator.md」 — Skills（技能模块）模板
- 「whitepapers/README.md」 — 生产示例

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/OsawwTV9xi3tfFkmcSpc0HabnUL> ｜ 归档：2026-06-04
