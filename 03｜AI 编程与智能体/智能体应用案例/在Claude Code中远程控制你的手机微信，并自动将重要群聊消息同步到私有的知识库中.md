# 在Claude Code中远程控制你的手机微信，并自动将重要群聊消息同步到私有的知识库中

![](assets/TGB7dE17lo8peBxm2jFc4dNOn0e_001.jpg)

刷 X 无意间看到了一个新产品 Airtap。它能像人一样操作我们手机里的所有 App —— 点击、滚动、输入、导航等操作都可以执行。当然它最擅长的不是单次复杂任务，而是把我们日常生活里 每天 / 每周 / 每月重复的琐事自动跑起来。出于好奇我去试了试，最后我只能竖起大拇指。

我能想到的 比如帮你刷抖音、短视频、小红书 找到符合我们设定对应的爆款然后，再通过你自己的专属工作流进行二次创作发布到对应平台，追随热点进行创作还是比较容易起爆。

但是今天我想分享的是我一直想实现，但却不能实现的功能，就是如何将手机的微信聊天记录同步到我们私有的专属知识库中呢？有时候随便用个开源项目去获取微信的聊天记录就有可能触发微信的封控，一点都不好玩。但是用这个Airtap就完全不会存在这个问题了。

本文内容目录如下，可进行选看

- 1、先从官网开始
- 2、安卓手机下载APP
- 3、开启安卓手机无障碍
- 4、测试连接我的安卓小米手机
- 5、在 Claude Code 中安装 Skill
- 6、帮我将聊天信息同步保存到电脑
- 7、总结

## 一、先从官网开始

登录官网 [https://airtap.ai](https://airtap.ai/)

![](assets/TGB7dE17lo8peBxm2jFc4dNOn0e_002.jpg)

点击「Set an Airtap」，跳转到谷歌账号登录，好像必须要谷歌账户

![](assets/TGB7dE17lo8peBxm2jFc4dNOn0e_003.jpg)

登录完毕后，看到如下界面。

![](assets/TGB7dE17lo8peBxm2jFc4dNOn0e_004.jpg)

## 二、安卓手机下载APP

> 先重点说一下，体验有两种情况一种就是云手机模式，另外一种情况呢，就是直接在安卓手机上安装APK进行体验，目前呢不支持 IPhone 。

先简单来说一下右上角其实有一个小手机，这就是云手机的效果。

如果右侧云手机显示有问题一直在加载，记得切换网络哈，不行的话多切换几个线路应该就可以了。

![](assets/TGB7dE17lo8peBxm2jFc4dNOn0e_005.jpg)

接下来点击对话框中的「Cloud Phone」，再点击「Pair New Device」

![](assets/TGB7dE17lo8peBxm2jFc4dNOn0e_006.jpg)

会看到下载APK的界面，一种是你直接在电脑上下载好，通过微信传递到手机上，另外一种就是手机浏览器扫码下载apk，这个二维码中其实就是apk的下载链接。

![](assets/TGB7dE17lo8peBxm2jFc4dNOn0e_007.jpg)

我是直接电脑下载然后微信发给小米手机的

![](assets/TGB7dE17lo8peBxm2jFc4dNOn0e_008.jpg)

没想到小米8安装成功了，安装成功后，打开APP，先点击首页的「Activate my phone」

然后到官网首页二维码那个弹窗右侧有一个配对的六位Code，记得要点击一下下面的「Generate new code」

![](assets/TGB7dE17lo8peBxm2jFc4dNOn0e_009.jpg)

点击验证之后应该就可以在官网上看到配对的这个手机了。

![](assets/TGB7dE17lo8peBxm2jFc4dNOn0e_010.jpg)

## 三、开启安卓手机无障碍

配置成功之后就可以看到下面的页面

![](assets/TGB7dE17lo8peBxm2jFc4dNOn0e_011.jpg)

需要设置受限的功能

- 允许读取屏幕内容
- 允许点击和输入
- 启用应用导航

先点击「Allow Accessiaility Permissions」

- 1、开启无障碍功能：我的点击之后，就能看到无障碍功能菜单->是关闭的，点击后直接开启服务即可
- 2、还有一个可下载服务也去点击开启就可以使用了

![](assets/TGB7dE17lo8peBxm2jFc4dNOn0e_012.jpg)

进入到这个页面就代表权限设置成功了。

## 四、测试连接我的安卓小米手机

下面我们测试一下，在网站的对话框中直接输入： “帮我打开微信，并给AI少年发一条消息：早上好啊”

效果看下面的视频，能有这个效果确实真的好牛逼啊。

虽然说目前速度上有一定的延迟，但这个早晚是可以解决的。

第一：他能默认选中第一个微信(我手机中有两个微信)

第二：能直接找到我的微信好友

第三：能直接替我发微信消息

![](assets/TGB7dE17lo8peBxm2jFc4dNOn0e_013.jpg)

这才是我的手机助手啊

## 五、在Claude Code中安装Skill

airtap官方提供了一个Skill , 仓库地址是 [https://github.com/airtap-ai/airtap-skill](https://github.com/airtap-ai/airtap-skill)

```Bash
## 可直接执行这两个命令

## 也可以直接丢给AI让它来安装Skill

claude plugin marketplace add airtap-ai/airtap-skill

claude plugin install airtap@airtap


```

可以直接在 Claude Code、Codex、OpenClaw、Hermes Agent 或者其他 AI Agent 产品中安装这个Skill， 看下图命令已经有了说明已经安装好了。

![](assets/TGB7dE17lo8peBxm2jFc4dNOn0e_014.jpg)

安装完毕就可以看到这个 Skill 了。

然后在Claude Code中继续输入“请列出可用airtap接收器” （我是直接通过上面github中都有明确的使用指南）

![](assets/TGB7dE17lo8peBxm2jFc4dNOn0e_015.jpg)

红色的报错，代表我上面安装了Skill , Skill 中的脚本使用了python代码，他要安装依赖。

下面的报错就提示的非常明显了，我要进行设置一个令牌。到官方网站，然后点击左下角图标。

![](assets/TGB7dE17lo8peBxm2jFc4dNOn0e_016.jpg)

然后「Copy」这个token 直接发送给Claude Code叫他帮你配置就好了。

![](assets/TGB7dE17lo8peBxm2jFc4dNOn0e_017.jpg)

体验在Claude Code中帮你回复消息

![](assets/TGB7dE17lo8peBxm2jFc4dNOn0e_018.jpg)

跟上面的视频体验完全一样，这个Skill还是相当给力的。

## 六、帮我将聊天信息同步保存到电脑

继续直接在Claude Code中进行输入消息“帮我把跟AI少年最近五条聊天记录保存到Claude Code当前文件夹，使用md格式进行存储”

![](assets/TGB7dE17lo8peBxm2jFc4dNOn0e_019.jpg)

当然看提示词，他对图片没有进行识别，可能对视频也没处理，我们可以通过提示词来进一步处理。

我试过了因为如果是一个视频，他可以点开播放，然后进行截图并分析视频的内容和声音的，所以像语音、文件、图片等都可以进行处理的。

来摘取重要信息，再对接自己的工作流或者知识库，对群消息进行定点监控处理。

不会再错过重要群消息的任何一条记录。

## 七、总结

感觉可以深度使用一波，而且他还有云手机可以体验，有兴趣的来玩玩吧。

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/WCZPwhzlqiYi2hkJR8PcD62unbe> ｜ 归档：2026-06-04
