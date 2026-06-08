# 别再切屏问 AI 了！把 Claude、Gemini、Codex 塞进命令行的保姆级教程与避坑指南

现在网上各种CLI都很火，GUI反而是慢慢的退出大众视野，为什么大家开始喜欢终端里使用AI呢？抱着这个态度，我基本上把市面上主流的CLI都安装了一遍，随便把安装教程也梳理了一遍。

在我刚开始折腾 CLI，选择的是`Claude Code`，但是后面、`Gemini CLI`、`Codex CLI` 我也在后面的日子安装并体验了，而且都跑了一段时间。但是我可能和其他的文章不一样，这篇文章不聊跑分，也不争谁最强，我只讲 3 件事：怎么装、怎么判断它真的能用、我现在大概怎么分。你如果想第一时间使用上CLI，那这篇文章绝对可以当你的第一轮上手参考资料。

---

## 基础环境确认

对于新手，尤其是小白，我一般不建议上来就复制命令。CLI 这类工具最烦的不是安装命令本身，而是基础环境和知道为什么会错的解决方案。

首先先确认 4 件事：

1. 机器上有没有安装的 `node` 和 `npm`
2. 当前 shell 的 `PATH` 正不正常
3. 账号和订阅有没有到位
4. 网络环境是否稳定，会不会影响登录流程

先跑这几个最基础的检查命令：

```bash
node -v
npm -v
which node
which npm
echo $SHELL
```

如果这里已经报 `command not found`，那后面先别装 CLI，先把 Node 环境补齐。  
如果 Node 和 npm 都有，但你后面装完还是跑不起来，十有八九是 `PATH` 没刷进去，或者 shell 没重新加载。

---

## 开始安装

其实我相信网上各种各样的安装教程肯定已经满天飞了，我这里就不啰嗦直接使用最直接的方式讲解安装，中间也会插入一些踩坑和注意事项。

### Claude Code

如果你本身就在用 Claude 生态，`Claude Code` 上手会比较顺。最常见做法就是用 npm 直接装：

```bash
npm install -g @anthropic-ai/claude-code
```

装完先确认命令在不在：

```bash
claude --version
```

如果这里直接报：

```bash
command not found: claude
```

先别急着重装。优先看：

- 全局 npm bin 在不在 `PATH`
- shell 有没有重开
- 你是不是在用 nvm，但当前终端没切到正确版本

### Gemini CLI

`Gemini CLI` 其实是三个CLI客户端里面，我觉得做的最简陋的，很多功能都是缺失的。我也不知道谷歌这家公司为什么在CLI上面怎么不上心。但是我还是推荐尝试一下这个 CLI，不是他有多好，而是可以体验Gemini相关的模型，后续我也会讲解为什么推荐安装：

```bash
npm install -g @google/gemini-cli
```

装完照样先看版本：

```bash
gemini --version
```

其实安装没多难，只要是去AI studio里面获取API KEY，很多人都找不到入口。我这里就把谷歌获取API KEY的入口放在这里了，只需要生成API KEY配置上就可以使用了。

> AI Studio API Key 入口：https://aistudio.google.com/api-keys

### Codex CLI

`Codex CLI` 现在是我的主力，现在经常拿来干代码相关的活，很多人肯定会问我为什么不使用Claude Code作为主力，主要还是额度和封号问题。我觉得稳定才是第一位的，如果出现问题无法继续使用，会对现有的工作流造成冲击。

安装命令：

```bash
npm install -g @openai/codex
```

装完先看：

```bash
codex --version
```

如果版本能正常出来，就可以使用官方的授权登入登入使用了，如果不喜欢CLI也可以使用官方的Codex App也是十分好用的，但是我自己习惯了CLI的方式，也有可能是因为玩多了Linux，对命令行情有独钟。

注意事项：

- 环境变量没配好
- 账号状态不对
- 当前 shell 没拿到你刚写进去的配置

---

## 推荐小技巧

如果不熟悉命令，我们可以使用官方的`help` 命令找到常用的命令：

```bash
claude --help
gemini --help
codex --help
```

装完以后，我一般会先跑下面这几条最常用的命令，先确认这套 CLI 已经不是“简单的装上了而是真的能干活开始工作了。

登入授权：

```bash
claude auth
codex login
```

交互测试：

```bash
claude -p "用一句话说明当前目录适合做什么"
gemini -p "用一句话说明当前目录适合做什么"
codex exec "用一句话说明当前目录适合做什么"
```

还有几条我觉得比较重要的：

```bash
claude -c
gemini --list-sessions
codex resume --last
```

命令介绍：

- `-p` / `exec`：先试一条最小任务，不用一上来进交互界面
- `-c`、`--list-sessions`、`resume --last`：回到上次会话，不用每次重开

---

## 我的体验和感受

其实三个CLI我都有高强度的去使用，但是最主力的还是Codex，以前是Claude code是我的绝对主力，但是在我封了三个号之后，我慢慢的有意识的迁移到了Codex上去了。

我自己经常干的活有三类：

1. 写代码
2. 写文案
3. 干一些脚本或者任务编排

写代码我最愿意使用的是Codex，因为额度足够还有单独的review代码的额度，写后端代码也深得我心。

文案创作方面，我喜欢用Claude去完成任务的编排，最主要的还是很简单就可以凝练我需要的中心思想，而且官方仓库的skill真的很完善充足。

Gemini在很多时候都是给我打下手的，我为什么会去安装Gemini CLI的原因就是方便其他AI调用它，只需要告诉他相关的命令和安装的位置，整合成一个对应的Skill。在需要调用其他模型组合完成任务的使用，就可以使用对应的CLI完成很方便的组合调用，Token也很节省。

再简单任务分配，我都是让Claude Code调用Gemini的CLI完成，我知道肯定还有其他方案，但是在我自己使用下来这种方式还是最舒服的。

> 延伸阅读：  
> 如果你更关心装完以后到底怎么分工，可以接着看这篇：高强度实测 6 大 AI 模型：Claude 写文最强，但我写代码不选它。

---

## 我的一些建议

其实别把东西想的太复杂，可以先上手试一试，是不是自己喜欢的，每天都有新的知识，我们不可能不去更新迭代。

前面我测试国产模型也是一样的意思，多尝试和实际体验才知道哪个才是自己的，虽然网上有很多相关的测评和解说，但是跟自己真正上手还是有很大差距的。也不要带着有色眼镜去看待其他厂商的测评，觉得某一个厂商就一定行，某一个厂商就绝对不行。谷歌这么大的厂，在CLI的完成度确是最差的，这也是我没想到的。

具体建议：

你主要写代码，就装 `Codex CLI`。  
你更看重连续协作和解释，就装 `Claude Code`。  
你只是想让其他模型使用Gemini的模型，而且是通过 CLI 的方式，那你就可以选择安装 `Gemini CLI`。

---

## 延伸阅读

- [Claude Code 安装教程：Mac、Windows、Linux 从 0 到跑通](Claude%20Code%20安装教程：Mac、Windows、Linux%20从%200%20到跑通.md) — Claude Code 单独装的详细版
- [小白必看！Opencode 傻瓜式安装教程，把 DeepSeek 接上](小白必看！Opencode%20傻瓜式安装教程，终于把%20DeepSeek%20接上了！.md) — 国产模型命令行选项
- [高强度实测 6 大 AI 模型：Claude 写文最强，但我写代码不选它](../工具测评/高强度实测%206%20大%20AI%20模型：Claude%20写文最强，但我写代码不选它.md) — 三家命令行用哪个模型

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/XQXGwEHs4iC9i1kIKoycVCOlnkh> ｜ 归档：2026-06-04
