# 动态生成 OG 图片

在构建时自动生成社交预览图，而非维护过时的静态 PNG。Twitter/X、LinkedIn 或 Slack 上的每次分享都将显示准确、最新的数据。

## 为何值得

静态 OG 图片会过时。你添加第 200 个模板或达到 1,000 个 GitHub stars 的那天，你的社交预览仍然显示旧数字。动态生成一次性解决这个问题，并永久保持准确。

下面的模式使用 Satori（Vercel）将类 React 树渲染为 SVG，然后用 resvg 转换为 PNG。它在 Astro 的构建时运行——零运行时成本，无外部服务。

## 技术栈

| 包 | 作用 |
|-|-|
| `satori` | 将类 JSX 对象树渲染为 SVG |
| `@resvg/resvg-js` | 将 SVG 转换为 PNG（Rust，快速） |
| `@fontsource/inter` | 本地字体文件（需要 woff1 格式） |

## 配置

```Bash
pnpm add satori @resvg/resvg-js @fontsource/inter

```

在 `src/pages/og-image.png.ts` 创建文件。Astro 自动在 `/og-image.png` 提供服务。

从你的布局文件引用：

```HTML
<meta property="og:image" content="/og-image.png" />
<meta name="twitter:image" content="/og-image.png" />

```

参见即用模板：examples/scripts/og-image-astro.ts

## 模式

```TypeScript
import type { APIRoute } from 'astro'
import satori from 'satori'
import { Resvg } from '@resvg/resvg-js'
import { readFileSync } from 'fs'
import { resolve, dirname } from 'path'
import { fileURLToPath } from 'url'

const __dirname = dirname(fileURLToPath(import.meta.url))

export const GET: APIRoute = () => {
  const fontData = readFileSync(
    resolve(__dirname, '../../node_modules/@fontsource/inter/files/inter-latin-400-normal.woff')
  ).buffer as ArrayBuffer

  const svg = satori(
    { type: 'div', props: { style: { /* ... */ }, children: [ /* ... */ ] } },
    { width: 1200, height: 630, fonts: [{ name: 'Inter', data: fontData }] }
  )

  const png = new Resvg(svg, { fitTo: { mode: 'width', value: 1200 } }).render().asPng()

  return new Response(png.buffer as ArrayBuffer, {
    headers: { 'Content-Type': 'image/png' },
  })
}

```

## 从内容动态获取数据

在构建时统计内容文件，而非硬编码：

```TypeScript
function countQuestions(): number {
  const dir = resolve(__dirname, '../content/questions')
  let total = 0
  for (const cat of readdirSync(dir, { withFileTypes: true })) {
    if (cat.isDirectory()) {
      total += readdirSync(resolve(dir, cat.name))
        .filter(f => f.endsWith('.md')).length
    }
  }
  return total
}

```

可以自动统计的数据：

- 内容目录中的 Markdown 文件（问题、文章、文档）
- 数据文件中的 YAML 条目
- 大型文档的行数

需要手动更新的数据：

- GitHub stars（动态变化，用 `1.1k+` 作为保守标签）
- 来自另一个仓库的模板
- 性能基准

## 注意事项

### 字体格式很重要

Satori 需要 **woff1** 或 **TTF**。使用 woff2 或重定向到 HTML 的远程 CDN URL 会静默失败或抛出错误。

```TypeScript
// 正确——来自 @fontsource 的本地 woff1
readFileSync('node_modules/@fontsource/inter/files/inter-latin-400-normal.woff')

// 失败——resvg 不支持 woff2
readFileSync('node_modules/@fontsource/inter/files/inter-latin-400-normal.woff2')

// 失败——CDN 可能返回 HTML（重定向、认证墙）
await fetch('https://fonts.gstatic.com/s/inter/...')

```

### 静态文件覆盖 API 路由

Astro 开发服务器在 API 路由**之前**提供 `public/` 中的静态文件。如果你有 `public/og-image.png`，它将始终被提供，而非你的动态端点。

**删除它：**

```Bash
rm public/og-image.png

```

同时检查项目根目录和 `dist/`——那里的文件也可能覆盖路由。使用 `curl -I http://localhost:4321/og-image.png` 诊断：如果响应有 `Last-Modified` 头，你访问的是静态文件，而非 API 路由。

### 浏览器缓存

删除静态文件后，进行强制刷新（`Cmd+Shift+R`）或在新的隐私窗口中测试。浏览器可能已经积极缓存了旧的 PNG。

### `satori` 在新版本中是异步的

某些版本的 satori 返回 `Promise<string>`，其他版本返回 `string`。如果你得到 `[object Promise]` PNG，添加 `await`：

```TypeScript
const svg = await satori(tree, options)

```

## 测试

**本地预览**——直接在浏览器中访问：

```Plain Text
http://localhost:4321/og-image.png

```

**社交预览模拟**——将你的生产 URL 粘贴到：

- [opengraph.xyz](https://www.opengraph.xyz) — 通用 OG 调试器
- LinkedIn Post Inspector（`linkedin.com/post-inspector/`）——强制 LinkedIn 刷新缓存
- Twitter Card Validator（`cards-dev.twitter.com/validator`）

**CI 检查**——如果你想捕获回退，可以添加一个检查生成的 PNG 文件大小是否超过阈值的构建步骤：

```Bash
# 在 CI 中 pnpm build 之后
SIZE=$(wc -c < dist/og-image.png)
if [ "$SIZE" -lt 10000 ]; then
  echo "og-image.png 看起来太小了 ($SIZE 字节) — 生成可能已失败"
  exit 1
fi

```

## 变体

### 个人品牌（无数据网格）

```TypeScript
children: [
  { type: 'span', props: { style: { fontSize: '48px', color: '#c0522a' }, children: 'FB.' } },
  { type: 'span', props: { style: { fontSize: '80px', fontWeight: 800, color: '#f5f5f5' }, children: '你的名字' } },
  { type: 'span', props: { style: { fontSize: '24px', color: '#8b949e' }, children: '你的标语' } },
]

```

### 项目列表徽章

```TypeScript
['project-a.com', 'project-b.com', 'project-c.com'].map(label => ({
  type: 'div',
  props: {
    style: { background: '#161b22', border: '1px solid #30363d', borderRadius: '8px', padding: '8px 16px' },
    children: [{ type: 'span', props: { style: { color: '#c0522a' }, children: label } }],
  },
}))

```

### 终端风格徽章（适合 CLI 工具）

```TypeScript
{
  type: 'div',
  props: {
    style: { background: '#21262d', border: '1px solid #30363d', borderRadius: '20px', padding: '6px 16px', color: '#3fb950', fontFamily: 'monospace' },
    children: '>_ your-cli-tool',
  },
}

```

## 保持数据同步

维护单一真相来源。当你更新 OG 图片中的数据时，在同一次提交中更新所有地方（落地页徽章、README 等）。

对于有多个落地页的项目，创建一个斜杠命令 `/update-stats-image-landings`，遍历每个仓库并提示你验证每个数据点。这能防止各站点之间的漂移。

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/UwxYwYL8xiN8aekYJYXcuPG4nmh> ｜ 归档：2026-06-04
