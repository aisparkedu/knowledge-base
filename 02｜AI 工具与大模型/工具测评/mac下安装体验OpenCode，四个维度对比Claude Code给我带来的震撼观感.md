# mac下安装体验OpenCode，四个维度对比Claude Code给我带来的震撼观感

![](assets/BwUody7pPo7QktxIC29cGBJ4nrc_001.jpg)

这是这个系列的第五篇文章，我会把自己最近从零开始梳理，整理 Mac 使用过程中的经验与步骤记录下来，作为留存与分享。如果拿到一台新的Mac电脑，我觉得你有必要来试试这个神级的OpenCode，相同模型他很多情况下竟然比Claude Code还强？

我的Mac使用指南系列文章如下：

- 1、[Mac使用指南系列文章：Homebrew软件包管理器从入门到精通](../../05｜AI%20效率提升/个人工具箱与环境配置/Mac使用指南系列文章：Homebrew软件包管理器从入门到精通.md)
- 2、[Mac使用指南系列文章：从零搭建Codex App桌面端结合GitHub CLI，体验 AI 自动化克隆与提交](../../05｜AI%20效率提升/个人工具箱与环境配置/Mac使用指南系列文章：从零搭建Codex%20App桌面端结合GitHub%20CLI，体验%20AI%20自动化克隆与提交.md)
- 3、[Mac使用指南系列文章：告别自带终端，Mac装机首选的 AI 友好型终端 Ghostty 配置指南](../../05｜AI%20效率提升/个人工具箱与环境配置/Mac使用指南系列文章：告别自带终端，Mac装机首选的%20AI%20友好型终端%20Ghostty%20配置指南.md)
- 4、[Mac 新机一条 Prompt 搞定 3 个 AI 开发工具：Claude Code + Codex CLI + Gemini CLI](../AI%20工具教程/Mac%20新机一条%20Prompt%20搞定%203%20个%20AI%20开发工具：Claude%20Code%20+%20Codex%20CLI%20+%20Gemini%20CLI.md)

首先就是在 Mac 电脑上，先安装好Homebrew软件包管理工具（可参考系列第一篇），然后通过Homebrew将Codex App 桌面端安装好（可参考系列第二篇）。

如果你不想装或者用不了Codex App，可以直接下载OpenCode客户端。

官网地址：[https://opencode.ai/download](https://opencode.ai/download)，下载对应的版本进行安装即可，window和mac都支持的，同时mac的话他竟然还支持老的intel芯片系列，非常不错，我的8G 256 mac 战机还能继续再战。并且里面有免费模型可以使用。

![](assets/BwUody7pPo7QktxIC29cGBJ4nrc_002.jpg)

在电脑上安装好第一个AI 客户端之后，就可以践行AI First理念，能让AI来帮我们完成的，就坚决不动手了，除非AI搞不定了，那我们再上手干。

本文内容目录如下，可进行选看

- 一、给OpenCode客户端配置免费模型
- 二、准备nodejs环境
- 三、通过OpenCode客户端安装OpenCode CLI
- 四、安装Claude Code并给他配置同样的免费模型
- 五、全文重点：四个终端执行同一个提示词任务
- 六、最后总结

> 第五部分有我录制的精彩小视频可以直接跳过去查看

## 一、给OpenCode客户端配置免费模型

上面文章开头我们已经下载了OpenCode客户端

![](assets/BwUody7pPo7QktxIC29cGBJ4nrc_003.jpg)

按照上面箭头的位置进行点击选择模型，然后再点击连接提供商

![](assets/BwUody7pPo7QktxIC29cGBJ4nrc_004.jpg)

选择OpenRouter，这里为什么选择OpenRouter呢，因为最近蚂蚁百灵刚好开源了一个大模型inclusionai/ring-2.6-1t:free，而且免费到5月15日。

链接地址：[https://openrouter.ai/inclusionai/ring-2.6-1t:free](https://openrouter.ai/inclusionai/ring-2.6-1t:free)

> Ring-2.6-1T 是万亿参数规模的思考模型(激活参数 630 亿),面向真实场景的智能体工作流,擅长编码、工具调用和长程任务,在 PinchBench、ClawEval、TAU2-Bench、GAIA2-search 等基准上表现领先。它支持 high 与 xhigh 两档自适应推理,可按任务复杂度动态分配推理预算,在工具密集和多轮交互中以更低的 token 开销实现更强性能。适用于高级编码智能体、复杂推理流水线及大规模自主系统等对质量、延迟和成本均有严苛要求的场景。

继续操作，选择上图中的OpenRouter

![](assets/BwUody7pPo7QktxIC29cGBJ4nrc_005.jpg)

到这里就需要在OpenRouter平台申请API Keys。

在OpenRouter平台[https://openrouter.ai/workspaces/default/keys](https://openrouter.ai/workspaces/default/keys) 设置通用的API Keys，然后复制到上面的截图中点击提交。

> 注意：我超出了每天针对免费模型50次的请求限额，于是我添加 10个积分（也就是充值10美元）之后，即可解锁每日 1000 次免费模型请求。貌似就是20倍请求数量，还是很香的，偶尔想测试一个新模型的长任务应该是绰绰有余了，有兴趣的可以去绑定一下。

API密钥设置好了，该怎么配置我们上面说的免费模型呢

```JSON
"openrouter": {

  "models": {

    "inclusionai/ring-2.6-1t:free": {

      "cost": { "input": 0, "output": 0 },

      "limit": { "context": 262144, "output": 65536 },

      "modalities": { "input": ["text"], "output": ["text"] },

      "name": "inclusionAI: Ring-2.6-1T (free)",

      "reasoning": true,

      "temperature": true,

      "tool_call": true

    }

  }

}


```

直接将上面的json内容拷贝给OpenCode客户端，帮我将上面的模型配置配置到opencode.json中进行使用（它可能找不到路径，你可以根据我下图所示的路径）

![](assets/BwUody7pPo7QktxIC29cGBJ4nrc_006.jpg)

如上图所示代表暂时配置完毕，关掉OpenCode客户端进行重启

![](assets/BwUody7pPo7QktxIC29cGBJ4nrc_007.jpg)

此时就能直接找到我们上面配置的inclusionai/ring-2.6-1t:free，选择之后，再输入个"你好"，看看回不回复消息。

![](assets/BwUody7pPo7QktxIC29cGBJ4nrc_008.jpg)

## 二、准备nodejs环境

为什么要装 Node.js？因为 Claude Code、Codex CLI、Gemini CLI、OpenCode 等 AI 命令行工具都是基于 Node.js 开发的，安装和运行它们都需要 Node.js 环境。

直接打开本地的命令行中输入 `node -v` 如果你看到如下的截图，那就说明node还没有安装

![](assets/BwUody7pPo7QktxIC29cGBJ4nrc_009.jpg)

本着AI first的理念，我还是使用Codex App或者OpenCode客户端来安装nodejs，两者都是非常优秀的AI Agent客户端。原则就是能用homebrew来安装的尽量就用homebrew来安装，方便统一。

![](assets/BwUody7pPo7QktxIC29cGBJ4nrc_010.jpg)

## 三、通过OpenCode客户端安装 OpenCode CLI

访问opencode官网首页 [https://opencode.ai](https://opencode.ai/)

就可以看到很多种安装方法，这里我使用的是npm进行安装

直接在OpenCode客户端输入安装命令即可

![](assets/BwUody7pPo7QktxIC29cGBJ4nrc_011.jpg)

启动之后如下图所示安装

![](assets/BwUody7pPo7QktxIC29cGBJ4nrc_012.jpg)

OpenCode客户端和OpenCode CLI读取的模型都是同一份配置，所以我们在OpenCode客户端中指定的免费模型，在OpenCode CLI 中也是可以使用的。

![](assets/BwUody7pPo7QktxIC29cGBJ4nrc_013.jpg)

## 四、安装Claude Code并给他配置同样的蚂蚁百灵大模型

在文章开头的mac系列文章的第四篇，里面有详细的安装过程，在文章最后也有配置蚂蚁百灵大模型的方式，只需要重新修改一下模型名称就可以了。

![](assets/BwUody7pPo7QktxIC29cGBJ4nrc_014.jpg)

然后设置命令行快捷别名

![](assets/BwUody7pPo7QktxIC29cGBJ4nrc_015.jpg)

然后退出并重新加载配置文件，或者关闭当前终端窗口，重新打开一个终端执行cc-ring命令

![](assets/BwUody7pPo7QktxIC29cGBJ4nrc_016.jpg)

## 五、全文重点：四个终端执行同一个提示词任务

![](assets/BwUody7pPo7QktxIC29cGBJ4nrc_017.jpg)

- 左上：ClaudeCode中蚂蚁百灵大模型
- 左下：OpenCode中蚂蚁百灵大模型
- 右上：ClaudeCode中DeepSeek
- 右下：ClaudeCode中Claude Opus 4.7

四个维度：相同模型不同终端，相同终端不同模型，不同模型不同终端，再加上地标最强Claude Code+ Claude Opus 4.7

四个终端都执行的提示词任务是：

```Plain Text
给我生成一个html小游戏：坦克自动对战,并写入到当前文件夹中的单个的html文件，xxxxx.html


```

四个终端先自主规划，然后实现，都是一次性执行完毕后的效果。下面请看视频的对比效果，跟上面终端的位置是一一对应的。

![](assets/BwUody7pPo7QktxIC29cGBJ4nrc_018.jpg)

## 六、最后总结

右下角Claude Code中使用模型Claude Opus 4.7 生成的是最快的，而且多坦克对战，效果还是非常不错的，应该再微调一下效果会更好一些。

左下角OpenCode中使用蚂蚁百灵免费模型的效果还是让我最意外的，没想到能有这样的效果。

而且比左上角同模型的Claude Code中的效果还要好很多，我本以为Claude Code的效果肯定比OpenCode要好的，没想到结果有点意外，当然了测试数据相对样本比较少，应该可以继续多抽几次卡。

右上角的DeepSeek感觉也一般般，甚至也比不上OpenCode的，难道OpenCode的Agent设计有他的独到之处吗？

看来是时候深度体验一波OpenCode，趁着蚂蚁百灵还在免费期也继续深度体验一波。

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/LJoKwTFUAi4tsykZhXxce7f4nrc> ｜ 归档：2026-06-04
