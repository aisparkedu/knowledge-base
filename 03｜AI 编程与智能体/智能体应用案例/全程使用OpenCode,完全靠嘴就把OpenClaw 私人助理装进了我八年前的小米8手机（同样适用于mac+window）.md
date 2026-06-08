# 全程使用OpenCode,完全靠嘴就把OpenClaw 私人助理装进了我八年前的小米8手机（同样适用于mac+window）

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_001.jpg)

上个周末，我的前端哥说你看“铁锤人”的OpenClaw 教程成为了技术圈的顶流。目前好像793W流量，恐怖如斯。于是我换了个角度，输出了我的米8实战，没想到也引来了部分爱好者的围观，虽然离“铁锤人”的流量还有很远的距离，但自认为流量也算不错，也是我没想到的，也帮我顺利突破2000粉丝。

围观者中：有小米10、苹果手机、一加、红米、平板、树莓派、mac min、window、还有一些特特有设备，也让我看到了像我一样的极客们，也躲在角落里在寻找志同道合的人，才引来了这场看似不经意的碰撞。

本质上，我对OpenClaw私人助理并不是特别感冒，因为Claude Code、Codex、Gemin Cli、OpenCode 等在电脑上的自主能力已经在这几个月被无限放大。而且也有类似插件，可以手机直接语音也好文字也好来实现，类似OpenClaw的代理功能。当然这可能只是在技术圈，而阿里云或腾讯云的一键安装助推了私人助理在普通AI关注者中的流行。

其实安装的难度不大，主要是使用的问题稳定性确实太多，毕竟这个东西很新，出问题是非常正常的。接下来我就像一个小白一样来教你通过OpenCode来安装私人助理，其实更重要的是学会思路，举一反三，看到这类工具真正的强大，我一般都是先跑通，再跑懂。一步一步搞明白。

希望OpenClaw这类工具能进化到像装一个APP一样就完成了部署安装，那才叫方便

再讲一下我的想法是怎么来的呢？

首先我的想法也不是凭空而出，主要基于以下三点：

早期我之前有在手机上体验过ollma 部署大模型，之前就使用过termux，termux相当于终端命令行工具。我相信应该也有很多人接触使用过。

然后前端时间刚好写过一个MCP，MCP的功能实现就是将本地项目，编译打包后自动上传到服务器的对应目录，刚好用到ssh远程服务器的操作。

第三点就是使用OpenCode 或者Claude Code 这类工具，他们在window或者mac上早已体现的有点无所不能的样子，只不过现在对接了一个远程遥控器一样，当然对需要的人来说那也是真的方便了。

当时上周末我还特意去找了一下，简单搜索好像还没什么教程，今天有没有就不清楚了。但是我感觉肯定有很多极客们甚至比我先行。

好了，所以基于以上三点开启，下面就开始了我的探索之旅。全程基本全靠OpenCode来进行黑盒的操作，我本想大不了就是打不开这个手机了。

下面所有的对话都是我在OpenCode中进行的。我是在window下操作的，mac下不知道会不会有不一样?

## 第一步

按照我的惯例来进行，先来问问OpenCode，因为我之前就已经知道要使用这个termux 的APK了（同样IOS也有类似的工具）。

[https://github.com/termux/termux-app](https://github.com/termux/termux-app) 这个开源项目用来干什么的

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_002.jpg)

## 第二步

**给我下载一个最新的apk**

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_003.jpg)

最好用较新的，因为OpenClaw中开发基本都是用的较新的技术，太过于老旧，遇到的问题就多，而且不太好解决。

## 第三步

**拷贝到安卓手机**

我使用的微信进行传输，点击接收，然后右上角使用QQ浏览器打开安装即可，然后打开Termux进行测试，如果出现下图的界面说明安装成功

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_004.jpg)

## 第四步

**开启远程操作手机，**已经安装完毕，能否在window上远程安卓手机

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_005.jpg)

> 注意一下：sshd这个命令后面还会经常用到

看上面写的‘方法一’给我提供了三种方法，剩余两种我暂时就不考虑

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_006.jpg)

这两种方法有兴趣的也可以试试。

这里我先按照第一个方案执行了，顺序执行上面‘方法一’中的四个命令就可以了

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_007.jpg)

要记得自己设置的密码哈

装好了，开始使用window或者mac进行连接，不对好像没有手机IP，继续问问OpenCode。

## 第五步

上面我使用的是第一种方法，IP地址怎么获取

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_008.jpg)

我在termux中，直接使用方法2，但是它提示我要先执行pkg install iproute2

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_009.jpg)

根据上面如图所示之后，就可以继续向下看

在我们的window或者mac的令行工具输入这个命令

ssh u0_a242@192.168.1.6 -p 8022，回车后再输入密码

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_010.jpg)

出现如上图所示，说明我们成功远程了我们的安卓手机了。

其实这里如果是在局域网就没啥问题了，随时可以连接。但是比如我现在在地铁，或者在户外，我想练怎么办呢？继续追问OpenCode

## 第六步（这里简单一点暂时可以省略，因为上面局域网已经可以连接了）

可以了，现在可以远程了。但是这个只是局域网的，能否随时可以连

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_011.jpg)

## 第七步

Tailscale 给我下载window安装包和安卓安装包 都要最新的

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_012.jpg)

## 第八步

现在我就去进行安装了。安装完毕，再打开tailscale官网。

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_013.jpg)

左侧的一个字段就是IP，我就没漏出来了，我现在让OpenCode来给我连接试试看。

（tailscale 有付费功能，也可以换成别的开源工具，这里提一嘴）

## 第九步

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_014.jpg)

接下来我直接让OpenCode 给我远程登录我的安卓手机

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_015.jpg)

这里最好生成SSH 密钥，相信我你后面玩的话还会遇到这个问题

接着我叫他给我把上面的脚本生成一个skill,方便我日后经常使用，并将密码配置在单独的配置文件中

## 第十步

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_016.jpg)

下次你只需要直接呼叫他连接就好了。

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_017.jpg)

看到这里帅不帅，但是我不能让他为所欲为的事情。

## 开始进入正题

先让他分析这个项目能不能安装到我的米8战机上

[https://github.com/openclaw/openclaw](https://github.com/openclaw/openclaw) 给我看看这个项目能否安装在我的米8上

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_018.jpg)

条件挺多的，其实就是要安装nodejs git python 等环境，下面就让他开干吧

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_019.jpg)

**游戏开始了，潘多拉的魔盒逐渐打开**

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_020.jpg)

经过了差不多十分钟

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_021.jpg)

还要进行向导设置

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_022.jpg)

由于我不清楚这个配置文件的位置

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_023.jpg)

直接让他将米8私人助理服务器上的配置文件，给我拷贝到本地，我修改完他再拷贝回去,我只改了两个配置(如上图箭头所示)。

然后进行配置模型，

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_024.jpg)

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_025.jpg)

配置完毕，准备开启

![](assets/IAnrdJ6Q6oDbD4x2fL3ccAy9nxc_026.jpg)

最后一张图推特限制上传不了了。

其实就是我上一篇中手机在的预览对话成功的。

手机访问http://127.0.0.1:18789,要配置一下上面gateway 密码，目前在overview中进行设置，保存后，再去对话，如果AI回复你了就说明你部署成功了。

## 最后

都可以找OpenCode试试，而且每个人因为环境不同或者AI对话效果不同，经历肯定也不尽相同，我这里算是也只是提供一个思路，给愿意折腾的人提供一个途径。window 和Mac应该也是可以的。

快过年了，预祝大家一切顺利。

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/CcJawJ50ci9i3hksCGbcZN70ndi> ｜ 归档：2026-06-04
