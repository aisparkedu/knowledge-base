# Claude Code + DeepSeek V4 保姆级安装配置

# 超简单 Claude Code + DeepSeek V4 安装手册（Mac 版）

---

## Node.js

Claude Code 的运行环境，必须先装。

https://nodejs.org（下载 `.pkg`，一路下一步）

```Bash
# 验证安装
node -v
npm -v
```

---

## Claude Code

官方安装脚本走 npm 镜像安装。

```Bash
npm install -g @anthropic-ai/claude-code --registry=https://registry.npmmirror.com
```

```Bash
# 验证安装
claude --version
```

---

## Homebrew

CC Switch 的安装依赖 Homebrew，先确认是否已有。

```Bash
# 检查是否已有
brew --version

# 没有则安装（国内镜像）
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

---

## CC Switch（Mac）

用于管理 Claude Code 的 API 配置，安装后将上方 DeepSeek 配置粘贴进去即可。

```Bash
brew install --cask cc-switch
```

---

## DeepSeek API Key

用 DeepSeek 替代 Anthropic 官方 API

[https://platform.deepseek.com](https://platform.deepseek.com) → API Keys → 创建并复制（只显示一次）

---

## 卸载方法

```Bash
  
 1. 卸载 npm（连同 Node.js）
  
  npm 是随 Node.js 一起安装的，取决于你当初的安装方式：

  如果是通过 nvm 安装的

  # 查看已安装的 node 版本
      nvm ls

  # 卸载某个版本
  nvm uninstall <version>

  # 完全移除 nvm 及所有 node/npm
  rm -rf ~/.nvm
  # 然后从 ~/.zshrc 或 ~/.bash_profile 中删除 nvm 相关配置行

  如果是通过 brew 安装的

  brew uninstall node
  brew cleanup

  如果是通过官方 pkg 安装器安装的

  sudo rm -rf /usr/local/lib/node_modules
  sudo rm -rf /usr/local/include/node
  sudo rm -rf /usr/local/bin/node /usr/local/bin/npm /usr/local/bin/npx
  sudo rm -rf /usr/local/share/man/man1/node.1
  sudo rm -rf /usr/local/share/doc/node
  # Apple Silicon 的话路径是 /opt/homebrew 开头

  清理残留

  rm -rf ~/.npm ~/.node-gyp ~/.node_repl_history

  ---
  2. 卸载 Claude（Claude Code CLI）
  
  取决于安装方式：

  通过 npm 全局安装的

  npm uninstall -g @anthropic-ai/claude-code
  # 或者旧版包名
  npm uninstall -g claude-code

  通过 brew 安装的

  brew uninstall claude-code
  # 或
  brew uninstall --cask claude-code

  清理 Claude 残留文件

  rm -rf ~/.claude
  rm -rf ~/.anthropic

  ---
  3. 卸载 brew 安装的 "cc switch"
  
  这个你提到的是 brew 装的，执行：

  # 先确认包名
  brew list | grep -i cc-switch

  # 卸载
  brew uninstall cc-switch
  # 如果装了 cask 版本
  brew uninstall --cask cc-switch

  清理 brew 残留

  brew cleanup
  brew autoremove   # 移除不再需要的依赖

  ---
  执行顺序建议
  
  1. 先卸载 Claude（npm 或 brew）
  2. 再卸载 npm/Node.js
  3. 最后清理 brew 的 claude-code

  如果不确定各项当初是怎么装的，可以先运行这些诊断命令确认：

  which claude && claude --version
  brew list | grep -i claude
  which node && node -v
  which npm && npm -v
  npm ls -g --depth=0 2>/dev/null | grep -i claude

```

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/JIr9wBQReiXGMZkRq1ochbL2nkc> ｜ 归档：2026-06-04
