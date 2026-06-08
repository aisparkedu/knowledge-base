# AI 编程案例

> 💡
> 收录 AI 编程、Vibe Coding、小工具开发和代码辅助的案例文章。

# 建议收录

- 来自 AI Spark 作者的相关文章。
- 适合小白理解和收藏的资料。
- 能回答一个明确问题的教程、案例或观察。

# 文章入库格式

| 字段 | 说明 |
|-|-|
| 作者 | 来源 IP 或投稿人 |
| 适合谁 | 小白、开发者、内容创作者、运营、创业者等 |
| 解决什么问题 | 一句话写清楚文章价值 |
| 延伸阅读 | 链接到相关主题或作者索引 |

---

# 待填充

- [ ] 补 3 个 AI 编程案例。

---

# 已收录案例

## HyperFrames：用 Codex 与 Claude Code 把提示词跑成 30 秒短视频

自媒体缺素材，用 Codex APP 或 Claude Code 调 HyperFrames by HeyGen，本地 10 分钟出一条 1080×1920 / 30 秒 / 30fps 成片，全程不依赖云端。[→ 打开完整全文](AI%20编程案例/找不到高颜值视频素材？我用%20Codex%20与%20Claude%20Code%20跑通了%20HyperFrames.md)

| 字段 | 内容 |
|-|-|
| 作者 | 小墨同学 |
| 适合谁 | 缺视频素材的自媒体创作者、想试 AI 视频生成的小白 |
| 解决什么问题 | HyperFrames 在 Codex APP 和 Claude Code 上分别怎么装、坑在哪、两条路效果对比 |
| 延伸阅读 | [02.2｜AI 工具教程 → Claude Code 安装教程](../02｜AI%20工具与大模型/AI%20工具教程.md) |

**关键收获**

- Codex APP 走应用内插件菜单一键装；Claude Code 没插件市场，只能命令行 `npx skills add heygen-com/hyperframes` 本地装
- 必须带 `GIT_LFS_SKIP_SMUDGE=1`，否则会卡在 240MB 的 Git LFS 测试基线
- Node.js 22+ / FFmpeg / Chrome Headless Shell 三件套先用 `npx hyperframes doctor` 自检；16GB Mac 建议留 2GB 以上内存防 OOM
- 同一份提示词喂两边，Codex APP 节奏更稳，Claude Code 在开头页面的视觉表达更出彩

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/Gpg9wUYb7i0slMk8MICc4Zybnle> ｜ 归档：2026-06-04
