# Hooks 与事件

**用时**：1 小时 | **难度**：⭐⭐ 进阶

## 目标

自动化系统事件的响应。创建在 Claude Code 操作前后自动运行的脚本。

---

## 你将学到

- Hooks（钩子）的工作原理和触发时机
- 创建提交前验证
- 构建操作后通知
- 编写安全的自动化脚本
- 常见 Hooks 模式

---

## 什么是 Hooks？

**Hook（钩子）**是响应某个事件自动运行的脚本。

### 示例：提交前钩子

提交变更前：

```Plain Text
你执行：git commit -m "Fix bug"
       ↓
钩子运行：检查版本号一致性
       ↓
版本号错误时：
  ❌ 提交被阻止
  错误信息说明问题所在
       ↓
你修复：更新 VERSION 文件
       ↓
提交成功

```

Hooks 防止常见错误进入 Git。

### Hook 事件

Hooks 可在以下事件触发：

| 事件 | 时机 | 使用场景 |
|-|-|-|
| 工具前钩子（PreToolUse） | Claude 运行工具前 | 验证请求 |
| 工具后钩子（PostToolUse） | Claude 运行工具后 | 记录结果、检查输出 |
| PreCommit | Git 提交前 | 验证变更 |
| PostPush | Git 推送后 | 通知团队 |

---

## 创建你的第一个 Hook

Hooks 是 `.claude/hooks/` 中的 Bash（或 PowerShell）脚本。

### 基本 Hook 结构

```Bash
#!/bin/bash

# Hook: validate-version
# Event: PreCommit
# Description: Check that VERSION file is updated with other changes

# 获取被提交的文件
FILES=$(git diff --cached --name-only)

# 检查 guide/ 下的文件是否被修改
if echo "$FILES" | grep -q "guide/"; then
  # 如果 guide/ 有变更，VERSION 也必须有变更
  if ! echo "$FILES" | grep -q "VERSION"; then
    echo "❌ Error: guide/ was modified but VERSION wasn't updated"
    echo "Run: echo '3.x.x' > VERSION"
    exit 1  # 阻止提交
  fi
fi

exit 0  # 允许提交

```

### 文件位置

```Plain Text
my-project/
└── .claude/
    └── hooks/
        ├── validate-version.sh
        └── notify-team.sh

```

### Hook 退出码

```Bash
exit 0   # 成功——允许操作继续
exit 1   # 失败——阻止操作并显示错误
exit 2   # 警告——允许但显示警告信息

```

---

## Hooks 模式

### 模式 1：提交前验证

阻止验证失败的提交：

```Bash
#!/bin/bash
# Hook: security-check.sh
# 发现安全问题则阻止提交

# 检查硬编码的 API Key
if grep -r "sk_live_" .; then
  echo "❌ ERROR: Found hardcoded Stripe key"
  exit 1
fi

# 检查生产代码中的 console.log（排除测试）
if grep -r "console.log" src/ --exclude-dir=tests; then
  echo "❌ ERROR: Found console.log in source code"
  exit 1
fi

# 检查 TODO 注释（警告，不阻止）
if grep -r "TODO:" src/; then
  echo "⚠️  Warning: TODO comments found (not blocking)"
fi

exit 0

```

### 模式 2：提交后通知

提交成功后：

```Bash
#!/bin/bash
# Hook: notify-team.sh
# 特定提交后通知团队

COMMIT_MSG=$(git log -1 --pretty=%B)

# 如果是安全相关提交
if echo "$COMMIT_MSG" | grep -i "security"; then
  echo "🔐 Security commit: $COMMIT_MSG"
  # 发送到 Slack（可选）
  # curl -X POST $SLACK_WEBHOOK -d "Security update: $COMMIT_MSG"
fi

exit 0

```

### 模式 3：依赖检查

警告是否需要更新依赖：

```Bash
#!/bin/bash
# Hook: check-deps.sh
# 检查 package.json 是否在未更新 lock 文件的情况下被修改

FILES=$(git diff --cached --name-only)

if echo "$FILES" | grep -q "package.json"; then
  if ! echo "$FILES" | grep -q "package-lock.json"; then
    echo "⚠️  Warning: package.json changed but lock file wasn't updated"
    echo "Run: npm install"
  fi
fi

exit 0

```

---

## 注册 Hooks

Hooks 在 `.claude/settings.json` 中注册：

```JSON
{
  "hooks": {
    "pre_commit": ["validate-version.sh", "security-check.sh"],
    "post_commit": ["notify-team.sh"],
    "post_push": ["deploy-staging.sh"]
  }
}

```

或在 `settings.yaml` 中：

```YAML
hooks:
  pre_commit:
    - path: hooks/validate-version.sh
      description: "Check VERSION file updated"
      blocking: true
    - path: hooks/security-check.sh
      blocking: true
  post_commit:
    - path: hooks/notify-team.sh
      blocking: false

```

---

## 安全 Hooks 最佳实践

### 应该做

✅ 让 Hooks **幂等**（多次运行安全）✅ 记录 Hook 在做什么✅ 提供清晰的错误信息后退出✅ 在开头使用 `set -e`，遇到第一个错误即退出✅ 使脚本可执行：`chmod +x hook.sh`

### 不应该做

❌ 让 Hook 运行超过 5 秒（阻塞工作流）❌ 在 Hook 中进行网络调用（不可靠）❌ 让 Hook 修改文件（Hook 仅做验证）❌ 规则过于严格（令开发者沮丧）❌ 忘记先在本地测试 Hook

---

## 安全 Hook 模板

```Bash
#!/bin/bash
set -euo pipefail

# 安全、清晰自动化的 Hook 模板

HOOK_NAME="my-hook"
HOOK_VERSION="1.0.0"

# 输出颜色
RED='\033[0;31m'
YELLOW='\033[1;33m'
GREEN='\033[0;32m'
NC='\033[0m' # 无颜色

log_error() {
  echo -e "${RED}❌ $1${NC}"
}

log_warn() {
  echo -e "${YELLOW}⚠️  $1${NC}"
}

log_success() {
  echo -e "${GREEN}✅ $1${NC}"
}

# 主验证逻辑
main() {
  echo "Running: $HOOK_NAME ($HOOK_VERSION)"
  
  # 你的检查逻辑
  if some_check_fails; then
    log_error "Check failed because X"
    return 1
  fi
  
  log_success "All checks passed"
  return 0
}

# 运行并退出
main
exit $?

```

---

## 练习：创建验证 Hook

### 场景

你想防止意外提交以下内容：

- 行尾多余空格
- 新代码缺少测试文件
- 未解决的合并冲突

### 第一步：创建 Hook

```Bash
cat > .claude/hooks/pre-commit-validation.sh << 'EOF'
#!/bin/bash
set -euo pipefail

echo "🔍 Running pre-commit validation..."

# 检查 1：无行尾空格
if git diff --cached | grep -E '^[+].*\s+$' > /dev/null; then
  echo "❌ Trailing whitespace found:"
  git diff --cached | grep -E '^[+].*\s+$'
  exit 1
fi

# 检查 2：无合并冲突标记
if git diff --cached | grep -E '^[+].*<<<<<<|^[+].*======|^[+].*>>>>>>' > /dev/null; then
  echo "❌ Merge conflict markers found"
  exit 1
fi

# 检查 3：新文件应有测试
STAGED_FILES=$(git diff --cached --name-only)
for file in $STAGED_FILES; do
  if [[ $file == src/*.ts && $file != *test* ]]; then
    TEST_FILE="${file%.ts}.test.ts"
    if ! git ls-files | grep -q "$TEST_FILE"; then
      echo "⚠️  Warning: New file $file has no test file"
    fi
  fi
done

echo "✅ Pre-commit validation passed"
exit 0
EOF

chmod +x .claude/hooks/pre-commit-validation.sh

```

### 第二步：在 settings.json 中注册

```JSON
{
  "hooks": {
    "pre_commit": ["hooks/pre-commit-validation.sh"]
  }
}

```

### 第三步：测试它

创建一个有行尾空格的文件：

```Bash
echo "test line   " > test.txt  # 注意行尾的空格
git add test.txt

```

尝试提交：

```Bash
git commit -m "Test hook"

```

Hook 阻止了提交：

```Plain Text
❌ Trailing whitespace found:
+test line

```

### 第四步：修复并重试

```Bash
echo "test line" > test.txt  # 删除行尾空格
git add test.txt
git commit -m "Test hook (fixed)"

```

现在成功了：

```Plain Text
✅ Pre-commit validation passed

```

---

## 调试 Hooks

如果 Hook 神秘地失败了：

1. **手动运行**：

```Bash
bash .claude/hooks/my-hook.sh

```

1. **添加调试输出**：

```Bash
set -x  # 打印每条命令

```

1. **检查退出码**：

```Bash
bash .claude/hooks/my-hook.sh; echo "Exit: $?"

```

1. **测试 Hook 条件**：

```Bash
# 测试文件是否被修改
git diff --cached --name-only | grep "VERSION"
echo $?  # 0 = 找到，1 = 未找到

```

---

## 验证：以下都满足说明你已准备好

✓ 已创建至少一个 Hook 脚本

✓ 理解 Hook 事件类型（pre-commit、post-commit 等）

✓ 能在 settings.json 或 settings.yaml 中注册 Hooks

✓ 已在本地测试过 Hook

✓ 知道退出码的含义（0 = 成功，1 = 失败）

---

## 下一步

**模块 07：高级模式**涵盖：

- 多智能体编排
- 构建复杂工作流
- 错误处理与恢复
- 生产级自动化
- 团队协调模式

这将教你如何将前面所有概念组合成复杂的多智能体系统。

---

**完成模块 06？** → 进入 模块 07：高级模式

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/MQh9wFdZ5i2BtekBQkocSL5rnUh> ｜ 归档：2026-06-04
