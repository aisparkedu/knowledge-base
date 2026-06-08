# 0 元打通 Claude + Codex + OpenCode：一个免费模型，6 个入口，20 分钟搞定

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_001.jpg)

20 分钟，0 费用，6 个 AI 编程工具全部接入同一个免费模型。

这是我用 **CC Switch + OpenRouter** 跑通的结果。模型是 OpenRouter 最近上的 **Ring-2.6-1T**（免费版），Claude Code、Codex、OpenCode 的命令行和桌面端全部验证通过。

免费窗口期不知道能撑多久，下面直接上手。

## 一、最终效果

这套方案可以接入 6 个入口：

```Plain Text
Claude Code 命令行
Claude 桌面端
Codex CLI
Codex 桌面端
OpenCode 命令行
OpenCode 桌面端

```

使用的免费模型是：

```Plain Text
inclusionai/ring-2.6-1t:free

```

作为上游配置时，OpenRouter 地址一般填：

```Plain Text
https://openrouter.ai/api/v1

```

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_002.jpg)

## 二、准备 OpenRouter Key

先打开 OpenRouter 的 API Key 页面，新建一个 Key。

```Plain Text
https://openrouter.ai/keys

```

创建后复制保存，后面主要填到 CC Switch 里；Claude 桌面端这种特殊入口，会通过本地路由间接使用。

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_003.jpg)

## 三、安装并打开 CC Switch

下载安装 CC Switch，打开后先看一下首页里有哪些入口。

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_004.jpg)

这里有一个容易误会的地方：**不要把 CC Switch 当成只配置一次、所有入口都自动通用的地方。**

Claude、Codex、OpenCode 每个入口在 CC Switch 里的配置页面和生效方式都不完全一样。所以后面我会按入口分别写：OpenCode 怎么配、Claude 怎么配、Codex 怎么配。

不过 CC Switch 连接 OpenRouter 这一层，都会用到同一组基础配置：

```Plain Text
供应商：OpenRouter
API 地址：https://openrouter.ai/api/v1
API Key：你的 OpenRouter Key
模型：inclusionai/ring-2.6-1t:free

```

> ⚠️ 模型名一定要手动填完整，注意最后的 :free 不要漏。后面每个入口的配置都用这组信息，不再重复列出。

## 四、OpenCode 接入

OpenCode 这边最顺，命令行和桌面端都已经验证通过。

## OpenCode 命令行

在 CC Switch 里切到 OpenCode，按第三节的基础配置填好 OpenRouter 信息，然后保存启用。

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_005.jpg)

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_006.jpg)

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_007.jpg)

打开 OpenCode 命令行，问 你是什么模型？，能正常回答就说明接入成功。

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_008.jpg)

## OpenCode 桌面端

OpenCode 桌面端也能用，但这里有个细节：

如果只在 OpenCode 桌面端里直接添加 OpenRouter Key，不一定能看到这个免费模型。

正确做法是先在 CC Switch 的 OpenCode 页面里把 OpenRouter 配好（包括手动填写模型名 inclusionai/ring-2.6-1t:free），然后回到 OpenCode 桌面端选择对应模型。

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_009.jpg)

问 你是什么模型？，能正常回答就说明 OpenCode 桌面端也通过。

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_010.jpg)

## 五、Claude 接入

Claude 分命令行和桌面端，两边方式不一样。

## Claude Code 命令行

Claude Code 命令行比较简单，在 CC Switch 的 Claude Code 页面里按基础配置填好 OpenRouter 信息即可。

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_011.jpg)

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_012.jpg)

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_013.jpg)

打开 Claude Code 命令行，问 你是什么模型？，能正常回答就说明通过。

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_014.jpg)

## Claude 桌面端

Claude 桌面端这里和前面不一样，不能按"直接填 OpenRouter 地址、Key、模型"的方式来配。

原因是：Claude 桌面端不一定支持直接选择 inclusionai/ring-2.6-1t:free 这个模型，也不适合把 OpenRouter 的参数直接填进去。

正确思路是：**先在 CC Switch 里开本地路由，再让 Claude 桌面端连接这个本地路由。**

先回到 CC Switch，确认 OpenRouter 和 Ring 免费模型已经配置好，然后开启本地路由 / 本地代理。

本地地址一般类似这样：

```Plain Text
http://127.0.0.1:15721

```

端口以你自己的 CC Switch 显示为准，不一定每个人都一样。

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_015.jpg)

然后打开 Claude 桌面端，进入开发者配置入口：

```Plain Text
Help → Troubleshooting → Enable Developer mode
Developer → Configure third-party inference

```

连接方式选择 Gateway，然后填写代理参数：

```Plain Text
Gateway base URL：http://127.0.0.1:15721
Gateway API key：PROXY_MANAGED
Gateway auth scheme：bearer

```

这里要注意：Claude 桌面端填的是 CC Switch 的本地地址，不是 OpenRouter 地址；API Key 通常也不是 OpenRouter Key，而是 PROXY_MANAGED。

真正的 OpenRouter Key 和 Ring 模型，仍然放在 CC Switch 里管理。可以把它理解成两层：Claude 桌面端前面只认 Gateway 和 Claude 模型名，CC Switch 后面负责转到 OpenRouter 和 Ring 模型。

这里还有一个关键点：Claude 桌面端这一层要配置它自己能识别的 Claude 模型名，不能直接填 Ring 的模型名。比如可以填：

```Plain Text
claude-opus-4.7

```

也就是说，Claude 桌面端看到的是 Claude 自己的模型名；真正请求到后面走哪个 OpenRouter 模型，由 CC Switch 负责转发。

如果这里直接填 inclusionai/ring-2.6-1t:free，Claude 桌面端可能识别不了，导致配置看起来对，但实际跑不通。

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_016.jpg)

配置好之后，重启 Claude 桌面端，问 你是什么模型？，能正常回答，并且 CC Switch 里能看到请求记录，就说明 Claude 桌面端通过。

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_017.jpg)

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_018.jpg)

## 六、Codex 接入

Codex 这边也分命令行和桌面端。

## Codex CLI

在 CC Switch 的 Codex 页面里按基础配置填好 OpenRouter 信息，保存并启用。

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_019.jpg)

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_020.jpg)

在 Codex 页面里，把 Codex 切换到刚才配置好的 OpenRouter 供应商。

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_021.jpg)

打开 Codex CLI，如果配置成功，命令行里可以看到这个模型，也可以切换到对应模型正常使用。

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_022.jpg)

问 你是什么模型？，能正常回答就说明 Codex CLI 通过。

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_023.jpg)

## Codex 桌面端

Codex 桌面端也能用，但显示上不如命令行清楚。同样在 CC Switch 里确认 Codex 已经走 OpenRouter 配置。

我们验证到的现象是：供应商能看到 OpenRouter，但具体模型不显示 Ring-2.6-1T，模型位置显示为"自定义"。**不用慌**，只要能正常请求，就说明它实际已经走到了对应配置。

问 你是什么模型？，能正常回答就说明 Codex 桌面端通过。

另外补一句：目前 CC Switch 对 Codex 的兼容性还不是特别完善，我这边发现的问题是历史会话可能无法正常加载。如果你只是新开会话测试和使用，影响不大；如果很依赖历史会话，建议先注意一下这个限制。

![](assets/WtlGdUsSHo779Oxt9ShcDfrVnBh_024.jpg)

## 七、验证结果

6 个入口全部用同一个问题测试：你是什么模型？

判断标准很简单——不要求模型准确自报名字（很多模型做不到），只要能发出请求、能收到回答、切换后能正常工作，就算通过。

这次验证结果如下：

- OpenCode 命令行：通过
- OpenCode 桌面端：通过
- Claude Code 命令行：通过
- Claude 桌面端：通过，需要通过 Gateway 本地路由接入
- Codex CLI：通过
- Codex 桌面端：通过，模型显示为“自定义”，不影响使用

以上只是基础接入验证。长任务稳定性、复杂代码修改、大项目表现，建议按你自己的真实场景再测。

## 写在最后

这篇教程不是在推荐 Ring-2.6-1T 这个模型本身——免费模型的能力上限大家心里有数。

真正的价值在于：**你现在花 20 分钟跑通这套配置，以后不管 OpenRouter 上出什么新模型，都只需要在 CC Switch 里改一行模型名，6 个入口同时切换，不用每个客户端都从头折腾一遍。**

这才是 CC Switch + OpenRouter 这套组合的意义：不是绑死在某个模型上，而是建好一条管道，让你随时能换水。

免费模型的窗口期不会一直在，趁现在还能用，先把管道搭起来。等哪天 OpenRouter 上线了更强的免费模型，你只需要回来换个名字就行。

**更多 AI 干货同步更新公众号：雨哥聊AI，关注我**[@xiangxiang103](https://x.com/@xiangxiang103)**带你玩转 AI 时代！**

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/JWLCw5L2TiHrjSkLTeScWEVxneh> ｜ 归档：2026-06-04
