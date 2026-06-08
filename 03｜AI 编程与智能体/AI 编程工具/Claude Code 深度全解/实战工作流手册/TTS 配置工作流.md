# TTS 配置工作流

**目标**：为 Claude Code 添加文字转语音旁白**时间**：18 分钟**难度**：中等**系统**：macOS（需要 Homebrew）

---

## 决策点：是否应该安装 TTS？

快速评估：

| 问题 | 回答 | 分数 |
|-|-|-|
| 你是否从事长时间的代码审查工作？ | 是 | +2 |
| 你是否在调试时同时处理多任务？ | 是 | +2 |
| 你更喜欢音频通知吗？ | 是 | +1 |
| 你是否需要离线 TTS（无云端）？ | 是 | +2 |
| 延迟是否关键（需要 <100ms）？ | 是 | -2 |
| 你是否在公共场所工作（无音频）？ | 是 | -3 |
| 你是否偏好安静的工作环境？ | 是 | -2 |

**分数**：

- **≥3**：安装 TTS（适合）
- **0-2**：可选（试用，可卸载）
- **<0**：跳过 TTS（不适合）

---

## 工作流概览

```Plain Text
阶段 1：前置条件（5 分钟）
    ↓
阶段 2：Agent Vibes 安装（5 分钟）
    ↓
阶段 3：Piper TTS + 语音（5 分钟）
    ↓
阶段 4：测试与配置（3 分钟）
    ↓
阶段 5：验证（1 分钟）

```

---

## 阶段 1：前置条件（5 分钟）

### 检查点 1.1：系统要求

```Bash
# 验证 macOS 版本
sw_vers
# 要求：macOS 10.15+

# 验证 Homebrew
brew --version
# 如果缺少：/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 验证 Node.js
node --version
# 要求：16.0.0+

```

### 检查点 1.2：安装 Bash 5.x

```Bash
# 安装
brew install bash

# 验证
/opt/homebrew/bin/bash --version
# 预期：GNU bash，版本 5.x

# ✅ 检查点：Bash 5.x 已安装

```

### 检查点 1.3：安装依赖

```Bash
# 安装音频工具
brew install sox ffmpeg util-linux espeak-ng

# 验证全部已安装
command -v sox && command -v ffmpeg && command -v espeak-ng && echo "✅ 依赖正常"

# ✅ 检查点：依赖已安装

```

**阶段 1 总时长**：约 5 分钟

---

## 阶段 2：Agent Vibes 安装（5 分钟）

### 步骤 2.1：启动安装程序

```Bash
# 进入你的项目目录
cd /path/to/your/claude-project

# 启动交互式安装程序
npx agentvibes install

```

**预期**：ASCII 横幅 + 4 页交互式安装程序

### 步骤 2.2：浏览页面

**第 1/4 页——依赖**：

- 检查：应显示所有 ✓ 绿色对勾
- 操作：点击「下一步 →」

**第 2/4 页——提供商**：

- **选择**：`Piper TTS`（最佳质量，离线）
- 操作：点击「下一步 →」

**第 3/4 页——语音**：

- **法语**：选择 `fr_FR-tom-medium`（男声，专业）
- **英语**：选择 `en_US-ryan-high`（最佳质量）
- 操作：点击「下一步 →」

**第 4/4 页——设置**：

- **混响**：`Light`（推荐）
- **背景音乐**：`Disabled`（避免干扰）
- **语言量**：`Low`（减少冗余）
- 操作：点击「开始安装」

### 检查点 2.3：验证安装

```Bash
# 检查已安装的文件
ls .claude/hooks/play-tts.sh
ls .claude/commands/agent-vibes/
cat .claude/tts-provider.txt
# 预期：文件存在，提供商显示「macos」或「piper」

# ✅ 检查点：Agent Vibes 已安装

```

**阶段 2 总时长**：约 5 分钟

---

## 阶段 3：Piper TTS + 法语语音（5 分钟）

### 步骤 3.1：通过 pipx 安装 Piper

```Bash
# 安装 Piper TTS
pipx install piper-tts

# 验证
piper --help
# 预期：Piper 使用说明

# ✅ 检查点：Piper 已安装

```

### 步骤 3.2：下载法语语音

```Bash
# 创建语音目录
mkdir -p ~/.claude/piper-voices
cd ~/.claude/piper-voices

# 下载法语男声（推荐）
curl -L -o fr_FR-tom-medium.onnx \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/tom/medium/fr_FR-tom-medium.onnx"
curl -L -o fr_FR-tom-medium.onnx.json \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/tom/medium/fr_FR-tom-medium.onnx.json"

# 下载法语女声（可选）
curl -L -o fr_FR-siwis-medium.onnx \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/siwis/medium/fr_FR-siwis-medium.onnx"
curl -L -o fr_FR-siwis-medium.onnx.json \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/siwis/medium/fr_FR-siwis-medium.onnx.json"

# ✅ 检查点：语音已下载（约 120MB）

```

**阶段 3 总时长**：约 5 分钟

---

## 阶段 4：配置与测试（3 分钟）

### 步骤 4.1：配置提供商和语音

```Bash
# 设置 Piper 为提供商
echo "piper" > .claude/tts-provider.txt

# 设置法语男声
echo "fr_FR-tom-medium" > .claude/tts-voice.txt

# 验证配置
cat .claude/tts-provider.txt  # 预期：piper
cat .claude/tts-voice.txt     # 预期：fr_FR-tom-medium

# ✅ 检查点：配置已设置

```

### 步骤 4.2：测试音频流水线

```Bash
# 直接测试 Piper
echo "Bonjour, je suis Claude et je parle français" | \
  piper -m ~/.claude/piper-voices/fr_FR-tom-medium.onnx \
  --output-file /tmp/test-fr.wav && afplay /tmp/test-fr.wav

# 测试 TTS 钩子
~/.claude/hooks/play-tts.sh "Ceci est un test audio"

# ✅ 检查点：音频正常

```

**预期**：你应该听到法语男声。

**阶段 4 总时长**：约 3 分钟

---

## 阶段 5：在 Claude Code 中验证（1 分钟）

### 步骤 5.1：启动与测试

```Bash
# 启动 Claude Code
claude

# 在 Claude 中运行：
/agent-vibes:whoami
# 预期：显示「piper」提供商和「fr_FR-tom-medium」语音

# 测试简单请求
> "Dis-moi bonjour en français"
# 预期：法语男声的音频响应

# ✅ 检查点：TTS 在 Claude Code 中已激活

```

### 步骤 5.2：配置偏好

```Bash
# 降低语言量（推荐）
/agent-vibes:verbosity low

# 若界面拥挤，隐藏 34 个命令
/agent-vibes:hide

# ✅ 检查点：偏好已设置

```

**阶段 5 总时长**：约 1 分钟

---

## 总时长：约 18 分钟 ✅

---

## 安装后建议

### 针对你的工作流优化

**用于代码审查**：

```Bash
/agent-vibes:verbosity low
/agent-vibes:effects off

```

**用于专注工作**：

```Bash
/agent-vibes:mute  # 临时静音
# 无音频工作
/agent-vibes:unmute  # 完成后重新启用

```

**用于电池优化**：

```Bash
# 切换到 macOS Say（即时，无 CPU 突发）
/agent-vibes:provider switch macos

```

### 添加到 .gitignore

```Bash
# 防止提交大型音频文件
echo ".claude/audio/" >> .gitignore
echo ".claude/piper-voices/" >> .gitignore
echo "*.wav" >> .gitignore
echo "*.onnx" >> .gitignore

```

---

## 故障排查快速参考

| 问题 | 快速修复 |
|-|-|
| 无音频 | 检查 `cat .claude/tts-provider.txt` |
| 语音错误 | 运行 `/agent-vibes:switch fr_FR-tom-medium` |
| 过于冗长 | 运行 `/agent-vibes:verbosity low` |
| 命令拥挤 | 运行 `/agent-vibes:hide` |

**完整故障排查**：「Agent Vibes 故障排查」

---

## 后续步骤

- **「语音目录」** - 探索 15 种语音
- **「集成指南」** - 学习命令
- **「安装详情」** - 深入了解

---

## 卸载说明

完全删除 Agent Vibes：

```Bash
# 自动卸载
npx agentvibes uninstall --yes

# 手动清理（如需）
rm -rf .claude/hooks/*vibes*
rm -rf .claude/commands/agent-vibes/
rm -rf .claude/audio/
rm -rf ~/.claude/piper-voices/
pipx uninstall piper-tts

```

---

*工作流指南由 《Claude Code：从入门到大师》团队维护最后更新：2026-01-22 | Agent Vibes v3.0.0*

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/C8uBw4mYliJyWBkwEHlcplbSnyc> ｜ 归档：2026-06-04
