# Ghostty 终端入门指南：安装、配置、用起来

![](assets/MtZ0d3Gw1oSepZx18uzcsMVvnie_001.png)

## Ghostty 是什么

Ghostty 是一个开源终端，用 GPU 加速渲染，速度快、内存省。配置文件是纯文本 key = value，没有 JSON 嵌套地狱。内置分屏、下拉终端、窗口状态恢复，不装插件就能多任务。作者是 HashiCorp 创始人 Mitchell Hashimoto，开源免费。

⚠️ Ghostty 目前只支持 macOS 和 Linux，暂不支持 Windows。

这篇文章只教一件事：**装好 Ghostty，配到舒服能用。**

> **⚡** **两种读法： 急着用？** 直接跳到文末「完整配置参考」，复制粘贴就行。 **想搞懂每条配置？** 往下看，每个配置项都有讲解。建议先收藏，用到的时候翻回来查

## 安装与第一次启动

**安装**

```Bash
brew install --cask ghostty

```

或者去 [ghostty.org](https://ghostty.org/) 下载

**第一次启动你会看到什么**

打开 Ghostty 后，你会看到两个东西：

1. **一个普通的终端窗口**——这就是主窗口
2. **一个从屏幕顶部滑下来的终端**——这是 Quick Terminal（下拉终端），按 Esc 或点别处可以收起来

别慌，这是正常行为。Quick Terminal 是 Ghostty 的特色功能，后面会详细讲。

**配置文件在哪**

在 Ghostty 里按 **Cmd + ,** 就能直接打开配置文件，不用记路径。如果你好奇，文件位置是：

```Bash
~/.config/ghostty/config

```

配置格式极其简单——每行一个 **key = value**，没有 JSON、没有 YAML、没有大括号。注释用 #。

```YAML
# 这是注释
theme = Catppuccin Mocha
font-size = 14

```

**三条必知命令**

开始配之前，先记三个内置命令：

```YAML
# 列出所有内置主题（200+ 个）
ghostty +list-themes

# 列出系统可用字体
ghostty +list-fonts

# 查看所有配置项的默认值和文档
ghostty +show-config --default --docs

```

最后一条特别有用——这就是 Ghostty 的完整配置手册，比翻网页快得多。

**重载配置**

改完配置文件后，不需要重启 Ghostty：

```Bash
Cmd + Shift + ,

```

按一下，配置立即生效。这是你接下来会反复用到的快捷键。

## 核心配置

**外观：主题 + 透明度 + 标题栏**

装完打开，默认主题有点素。而且白天晚上得手动切主题，我们本次直接设置好自动切换。

加三行配置，让它跟着系统自动切：

```YAML
# 亮色用 Catppuccin Latte，暗色用 Catppuccin Mocha
theme = light:Catppuccin Latte,dark:Catppuccin Mocha

# 背景透明度（1.0 = 完全不透明，0.85 = 微透明）
background-opacity = 0.85

# 隐藏原生标题栏，获得更多屏幕空间
macos-titlebar-style = hidden

```

几个细节：

- theme 支持 light:xxx,dark:yyy 语法，跟着 macOS 深色模式走，不用手动改。
- background-opacity 设在 0.85-0.95 之间比较合适。想对照文档写代码可以再降到 0.8，但太低了背景会干扰阅读。
- background-blur 配合透明度用，加上毛玻璃效果。
- macos-titlebar-style = hidden 藏掉标题栏但保留红绿灯。不过要注意：**hidden 模式下 Cmd+T 会开新窗口而不是新 Tab**——因为 macOS 原生 Tab 需要标题栏。如果你常用 Tab，建议改成 tabs，标题栏会变窄并集成 Tab 栏，视觉上也很干净。

**字体**

想要编程连字（!= 显示成 ≠），还要支持中文，又想要终端图标。以前这意味着装三个字体然后祈祷它们不打架。

现在一个就够了。Maple Mono 的 NF CN 版本把等宽、连字、中文、图标全打包在一起。而且 Ghostty 自带 Nerd Font 图标渲染，就算你用别的字体（比如 JetBrains Mono），终端图标也能正常显示，不用专门找 NF 补丁版。

```YAML
# 推荐方案 A：Maple Mono NF CN（连字 + 图标 + 中文 全包）
font-family = "Maple Mono NF CN"
font-size = 14

# 推荐方案 B：JetBrains Mono（Ghostty 内置 Nerd Font 图标，不需要 NF 版本）
# font-family = "JetBrains Mono"
# font-size = 14

# macOS 专属：字体加粗渲染，让细字体在 Retina 屏上更清晰
font-thicken = true

```

装字体：

```YAML
# Maple Mono（推荐）
brew install --cask font-maple-mono-nf-cn

# 或 JetBrains Mono
brew install --cask font-jetbrains-mono

```

连字可以微调——比如你不喜欢 != 变成 ≠，用 font-feature 精确控制：

```YAML
# 启用常用连字
font-feature = calt
font-feature = liga

```

**快捷键与分屏**

一个窗口经常不够用——你想一边跑命令一边看输出结果。Ghostty 内置分屏，不需要装插件。

常用快捷键：

**分屏**

- 左右分屏：Cmd + D
- 上下分屏：Cmd + Shift + D
- 下一个分屏：Cmd + Shift + ]
- 上一个分屏：Cmd + Shift + [
- 放大/还原当前分屏：Cmd + Shift + Enter
- 缩放分屏（增大/减小）：Cmd + Ctrl + = / -
- 关闭当前分屏：Cmd + W

**Tab 与窗口**

- 新建 Tab：Cmd + T
- 切换 Tab：Cmd + 数字键
- 全屏切换：Cmd + Enter

**搜索与工具**

- 搜索终端输出：Cmd + F（1.3.0 新增）
- 下一个/上一个结果：Cmd + G / Cmd + Shift + G
- 命令面板：Cmd + Shift + P
- 打开配置文件：Cmd + ,
- 重载配置：Cmd + Shift + ,

默认的 Cmd+Shift+[/] 是循环切换分屏。想按方向跳（像 Vim 那样）可以加：

```Plain Text
keybind = cmd+shift+h=goto_split:left
keybind = cmd+shift+j=goto_split:down
keybind = cmd+shift+k=goto_split:up
keybind = cmd+shift+l=goto_split:right

```

**Quick Terminal（下拉终端）**

正在看文档，突然想跑条命令。切到终端 → 跑完 → 切回来，心流断了。

Quick Terminal 就干这事——全局热键呼出来，用完自动收回去：

```YAML
# 全局热键（默认就有，这里可以自定义）
keybind = global:ctrl+grave_accent=toggle_quick_terminal

# 下拉终端的位置：top / bottom / left / right / center
quick-terminal-position = top

# 占屏幕比例（v1.2+ 支持精确尺寸）
quick-terminal-size = 50%

# 在哪个屏幕显示：main / mouse / macos-menu-bar
quick-terminal-screen = main

# 自动隐藏：失去焦点时收起
quick-terminal-autohide = true

# 动画时长（秒，0 = 无动画）
quick-terminal-animation-duration = 0.15

```

比如跑一条 git status、查个环境变量、临时算个数——按 Ctrl + ` 呼出，再按一次或切到别的窗口就自动收起。

**窗口行为**

每次重启 Ghostty，之前的分屏布局、Tab、工作目录全没了？加几行配置：

```YAML
# 永远记住窗口状态（分屏布局、Tab、目录）
window-save-state = always

# 新分屏/Tab 继承当前目录
window-inherit-working-directory = true

# 新窗口继承字体大小
window-inherit-font-size = true

# 内边距（像素），让文字不贴边
window-padding-x = 4
window-padding-y = 4

# 缩小窗口时，内边距等比缩小
window-padding-balance = true

```

window-save-state = always 是最实用的一条——重启后分屏布局原封不动恢复，连每个分屏的工作目录都记得。window-inherit-working-directory = true 也好用：在 ~/projects/my-app 目录按 Cmd+D 分屏，新分屏自动就在这个目录，不用再 cd 一次。

## 完整配置参考

不想一条一条配？下面是一份完整配置，直接复制到 ~/.config/ghostty/config 就能用。

> ⚠️ 配置里用了 Maple Mono 字体，复制前先装一下：brew install --cask font-maple-mono-nf-cn，否则字体会回退到系统默认。

```YAML
# ===========================
# Ghostty 完整配置
# ===========================

# --- 外观 ---
# 主题跟随系统深色模式自动切换
theme = light:Catppuccin Latte,dark:Catppuccin Mocha

# 背景透明度（0.0 ~ 1.0）
background-opacity = 0.88

# 背景模糊（配合透明度使用，毛玻璃效果）
background-blur = 20

# 背景图片（可选，放一张喜欢的图，终端瞬间好看）
# background-image = ~/Pictures/wallpaper.png
# background-image-opacity = 0.3
# background-image-fit = cover

# 标题栏集成 Tab 栏（比 hidden 多了 Tab 支持）
macos-titlebar-style = tabs

# 非活跃分屏的透明度（让你一眼看出焦点在哪）
unfocused-split-opacity = 0.9

# --- 字体 ---
# 推荐 Maple Mono NF CN（brew install --cask font-maple-mono-nf-cn）
font-family = "Maple Mono NF CN"
font-size = 14
font-thicken = true

# 连字支持
font-feature = calt
font-feature = liga

# --- 窗口行为 ---
# 永远记住窗口状态（分屏、Tab、目录）
window-save-state = always

# 新分屏继承当前目录
window-inherit-working-directory = true

# 新窗口继承字体大小
window-inherit-font-size = true

# 内边距
window-padding-x = 4
window-padding-y = 4
window-padding-balance = true

# --- Quick Terminal（下拉终端） ---
keybind = global:ctrl+grave_accent=toggle_quick_terminal
quick-terminal-screen = main
quick-terminal-position = top
quick-terminal-size = 50%
quick-terminal-autohide = true
quick-terminal-animation-duration = 0.15

# --- Shell 集成 ---
# 自动注入 shell 集成（光标样式、sudo、标题、SSH terminfo）
shell-integration-features = cursor,sudo,title,ssh-terminfo,ssh-env

# --- 滚动 ---
# 滚动缓冲区大小，单位是字节（默认 10MB，这里设为 50MB）
scrollback-limit = 50000000

# --- 光标 ---
cursor-style = block
cursor-style-blink = false

# 鼠标隐藏（打字时自动隐藏鼠标）
mouse-hide-while-typing = true

# --- 剪贴板 ---
# 选中即复制到系统剪贴板（和 iTerm2 一样）
copy-on-select = clipboard

# 复制时自动去除行尾空格
clipboard-trim-trailing-spaces = true
```

# --- macOS 专属 ---

# 退出时不弹确认框（如果你习惯了 Cmd+Q）

confirm-close-surface = false

# Option 键作为 Alt 使用（对 vim/emacs 用户很重要）

macos-option-as-alt = true

````Plain Text

**使用方法：**

在 Ghostty 里按 Cmd + , 打开配置文件，把上面的内容粘贴进去，保存。然后按 Cmd + Shift + , 重载配置，搞定。

别忘了装字体：

```bash
brew install --cask font-maple-mono-nf-cn

````

## 写在最后

到这里你的 Ghostty 应该已经跑起来了——主题跟着系统走、分屏随手开、Quick Terminal 一键呼出。比起 iTerm2，你大概能明显感觉到渲染更跟手，滚动更丝滑。

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/IZ4ywklcdi08qbk4qNCc9f17nsf> ｜ 归档：2026-06-04
