# Windows上使用 Claude Code的最佳方式

---

周末在家，正好看到蚂蚁 InclusionAI 正式发布了面向真实复杂任务的万亿级思考模型 **Ring-2.6-1T。**

我很想试一试这个新模型，工作用的电脑是Mac studio 放在公司了，家里只有一台Windows 的笔记本，我现在主力使用的还是 Claude Code，在mac上用的比较熟练，我想在Windows上应该不会太麻烦，结果第一步安装就把我撂倒了。

## 原生安装

我首先选择官方推荐的原生安装（Native Install）

我打开Windows 自带的终端，进入的是 powershell，输入下面的一键安装脚本后，一直没啥反应

```PowerShell
irm https://claude.ai/install.ps1 | iex
```

第一步就把我卡住了，结果没有办法，我打开了 Warp（另外一个可以使用ai的终端），来帮我分析为啥安装失败。

Warp确实给力，帮我拆解分析的很清楚，上面的脚本含义是下载并且执行脚本，所以Warp先帮我下载脚本到本地，然后分析脚本具体做了啥，一步步拆解，没想到，一步步拆解后，Claude code就安装好了。我继续问了ai，在Windows的powershell中能否使用 shell 函数的方式，设置环境变量来使用 Ring-2.6-1T

```PowerShell
# OpenRouter API Key
$env:OPENROUTER_API_KEY = "你的api-key"

# 会话: OpenRouter + inclusionai/ring-2.6-1t
function claude-ring {
    $env:ANTHROPIC_BASE_URL = "https://openrouter.ai/api"
    $env:ANTHROPIC_AUTH_TOKEN = $env:OPENROUTER_API_KEY
    $env:ANTHROPIC_API_KEY = ""
    $env:ANTHROPIC_MODEL = "inclusionai/ring-2.6-1t"
    $env:ANTHROPIC_SMALL_FAST_MODEL = "inclusionai/ring-2.6-1t"
    $env:CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC = "1"
    claude @args
}
```

居然可以，我在终端里很开心的开启了 Claude code，但是很快又遇到了问题，生成的代码居然无法保存到本地，一直说 Write 工具路径映射有问题，还要换成Python来写入文件，这越扯越离谱了。我在Mac上使用Claude code 一直无比丝滑，但是没想到Windows上遇到接二连三的问题。

## 换成wsl

后面我换了另外一个思路。Windows 可以安装wsl，其实就是一个linux的子系统（我安装的是 Ubuntu），Claude code 这种命令行，在Windows上有点水土不服，在linux上一定没啥问题，linux就是命令行软件的最佳土壤环境。

可是在wsl 子系统中，遇到了另外一个问题，那就是在命令行中如何使用代理的问题，我的Windows上已经开启了本地代理：127.0.0.1:7890。如何让wsl 中访问这个代理呢？我搜了网上，一开始这么设置的

```Bash
# setup shell proxy
# 1. dynamic get hostip

export hostip=$(cat /etc/resolv.conf |grep -oP '(?<=nameserver\ ).*')

function proxyon() {
  export https_proxy="http://${hostip}:7890"
  export http_proxy="http://${hostip}:7890"
  export all_proxy="http://${hostip}:7890"
  echo "shell proxy on"
}

function proxyoff() {
  unset https_proxy
  unset http_proxy
  unset all_proxy
  echo "shell proxy off"
}
```

上面的意思是，首先获取Windows主机的ip地址，然后shell中如果执行 proxyon这个函数，就会将代理设置为访问主机的7890端口，原理上好像一点问题都没有，实际上结果还是不行。

后面又换成了下面的办法。

**配置 WSL2 镜像网络模式 (Windows 11/新版)**

效果： 启用后，WSL2 将与 Windows 共享同一网络接口和 IP 地址，Windows 上监听的端口（如代理 7890）在 WSL 内可直接通过 127.0.0.1 访问，无需额外端口转发。

如何开启WSL2 镜像网络模式？

在用户目录下添加一个配置文件（没有的话就新建）.wslconfig

例如我的电脑是在 `C:\Users\yiran\.wslconfig` 文件中添加以下配置：

```TOML
[wsl2]
networkingMode=mirrored
```

1. 写入上面的配置内容并保存（整个配置文件，内容就是上面简单的两行）

2. 先关闭wsl后，重启 WSL：

    

```PowerShell
wsl --shutdown
```

wsl里面的 .zshrc 配置代理的部分调整一下

```Bash
# setup shell proxy
export hostip="127.0.0.1"

function proxyon() {
  export https_proxy="http://${hostip}:7890"
  export http_proxy="http://${hostip}:7890"
  export all_proxy="http://${hostip}:7890"
  echo "shell proxy on"
}

function proxyoff() {
  unset https_proxy
  unset http_proxy
  unset all_proxy
  echo "shell proxy off"
}
```

然后在wsl中，开启代理后，直接安装，这一次我们用的是

这次安装一路通畅。

## 我们要真正开始测试 Ring-2.6-1T 模型了。

Ring-2.6-1T是一款面向真实复杂任务场景打造的万亿级旗舰思考模型。

## 🛠️ 第一步：获取 Ring-2.6-1T 的 API Key

Ring-2.6-1T 目前已经在 OpenRouter 上线，可以免费使用到5.15号。

1. 前往 OpenRouter 官网并注册/登录账号。

2. 在后台找到 **Keys** 页面，点击生成一个新的 API Key。

3. **复制并保存好这串 API Key**，我们接下来的步骤会用到它。

    

## 📥 第二步：在 shell 配置里加个函数

把下面这段加到 **\~/.zshrc** 或 **\~/.bashrc** (zsh 示例, bash 几乎一样):

```Bash
export OPENROUTER_API_KEY="sk-or-v1-你的key"

# 会话: OpenRouter + inclusionai/ring-2.6-1t
claude-ring() {
  ANTHROPIC_BASE_URL="https://openrouter.ai/api" \
  ANTHROPIC_AUTH_TOKEN="$OPENROUTER_API_KEY" \
  ANTHROPIC_API_KEY="" \
  ANTHROPIC_MODEL="inclusionai/ring-2.6-1t" \
  ANTHROPIC_SMALL_FAST_MODEL="inclusionai/ring-2.6-1t" \
  CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1 \
  claude "$@"
}
```

## 🚀 第三步：在 Claude Code 中起飞！

在终端中，输入 claude-ring 回车，直接跑的是 Ring-2.6-1T。

## 📥 第四步：使用 Ring-2.6-1T 帮我生成粒子交互系统

使用下面的Prompt

```Plain Text
帮我生成一个交互式粒子流场氛围生成器（高质感动态背景艺术工具）

项目目标：
创建一个单文件、可直接部署的高级 p5.js 动态网页工具。核心是用 Perlin Noise 驱动的粒子流场，把抽象「氛围」翻译成实时可交互、可切换主题的高级视觉体验。要求视觉高级、不 AI Slop，交互丝滑，性能稳定（目标 60fps），支持桌面和移动端。核心能力要求（必须全部满足）：懂风格：支持 6-8 种高辨识度主题，每种主题有独立色阶、粒子形态、背景特效、噪声参数。
懂交互：鼠标/触屏实时扰动流场、能量爆发、拖拽控制。
懂修复：第一轮可能存在坐标系错误、性能问题、颜色突变等，必须在后续迭代中快速修复。
最终效果要达到「可直接作为工作台动态壁纸 / 直播氛围灯 / 创意灵感工具」的生产级水准。

技术栈要求：使用 p5.js（通过 CDN 引入，版本最新稳定版）
单个完整 HTML 文件（含 <script> 内全部 JS 代码）
优先使用 TypeScript 风格的清晰注释和结构（即使最终输出 JS 也保持 TS 思维）
必须使用 p5.Vector、noise()、blendMode() 等原生 p5 API
避免使用外部库（除 p5 本身）

MVP 功能清单（必须完整实现）：流场粒子系统（核心）2000-5000 个粒子（可调）
Perlin Noise 驱动的 2D 流场
粒子有生命周期、淡入淡出、拖尾效果
多主题系统（审美核心）至少 6 个预设主题（赛博朋克、梦幻樱花、宇宙星云、赛博禅意、极光森林、蒸汽朋克）
每个主题包含：背景渐变、粒子色相范围、速度、尺寸范围、额外特效
主题切换必须平滑过渡（使用 lerp）
实时交互鼠标位置产生局部力场扰动
点击/长按触发「能量爆发」（高亮粒子向外辐射）
拖拽可局部改变流场方向
控制面板（p5.dom）粒子密度、流场速度、噪声尺度、拖尾长度滑块
主题选择按钮（带文字或 emoji 缩略）
「随机 Vibe」按钮（自动生成新参数组合）
「保存当前配置」按钮（下载 JSON）
高质感细节背景多层渐变 + 轻微噪点/辉光
FPS 显示 + 性能自适应（粒子数自动调节）
全屏 + 响应式布局（手机端流畅）
优雅 UI 叠加（半透明深色面板）
```

当然不是一次就完美结果的，交互迭代了2，3次，就得到下面的页面效果

[Allow to sign in - Google Chrome 2026-05-10 19-44-34\_1.mp4]

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/RkRGwEMVSi5jkgko7Ircm1dTnzh> ｜ 归档：2026-06-04
