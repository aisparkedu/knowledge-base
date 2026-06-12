# Codex 插件 Computer Use 修复提示词

> 使用方法：在 Windows 版 Codex 中粘贴下面的提示词，让 Codex 自动诊断、备份、修复并验证 Computer Use 和 Chrome 插件。执行前请确认当前任务允许 Codex 读取相关配置与日志。

## 直接复制版

```Plain Text
请帮我完整诊断并修复 Windows 版 Codex 中的 Computer Use（电脑操控）不可用问题。

当前可能出现的症状包括：

1. Codex 设置 > 电脑操控显示“Computer Use 插件不可用”。
2. 退出并重新打开 Codex 后仍然无效。
3. Chrome 插件可能已经安装，但浏览器或本地应用读取异常。
4. 网上有人建议把 `[windows] sandbox = "elevated"` 改成 `"unelevated"`，但不要直接照做。先找出真实原因。
5. 已知一种常见故障是 Codex 启动日志出现：
   `EBUSY: resource busy or locked`
6. 常见原因是 Chrome 的 `extension-host.exe` 占用了临时的 `openai-bundled` 插件目录，导致 Codex 无法刷新官方插件市场。

请直接执行诊断、备份、修复和验证，不要只给我操作教程。

1. 当前系统是 Windows。
2. 自动识别真实用户名、`USERPROFILE`、`CODEX_HOME`、Codex 安装目录和版本，禁止写死用户名或版本号。
3. 如果命令运行在 `CodexSandboxUser`、`CodexSandboxOffline` 等沙盒账户下，导致读取到错误的用户配置，请申请必要权限，并以真实 Windows 用户环境重新检查。
4. 修改任何文件前必须创建带时间戳的备份。
5. 不得覆盖、删除或格式化无关配置。
6. 不得删除损坏目录，先将其改名保存为备份。
7. 不要自动化操作 Codex 自己的界面。
8. 不要关闭 Codex 主窗口，除非最后明确告诉我需要手动重启。
9. 不要为了尝试修复而关闭 Windows 安全功能、UAC、杀毒软件或降低目录权限。
10. 不要仅凭网上教程修改沙盒模式。除非日志明确证明沙盒配置是原因，否则保留现有 `[windows] sandbox` 设置。
11. 只结束能够确认属于 Codex Chrome 插件的 `extension-host.exe` 及其直属宿主进程，不要结束其他 Chrome、Node 或系统进程。
12. 如果当前故障不是下面描述的插件目录问题，请停止套用固定方案，改为根据实际日志处理。

请依次执行：

1. 确认当前真实 Windows 用户、`USERPROFILE` 和 `CODEX_HOME`。
2. 找到用户配置文件，通常为：
   `%USERPROFILE%\.codex\config.toml`
3. 找到 Codex 应用日志，优先检查：
   `%LOCALAPPDATA%\Codex\Logs`
4. 搜索以下关键词：
   - `Computer Use`
   - `computer-use`
   - `openai-bundled`
   - `EBUSY`
   - `resource busy or locked`
   - `marketplace_resolve_failed`
   - `extension-host`
5. 找到当前 Windows 安装的 Codex 应用包目录。
6. 在安装包中查找完整的官方插件市场，必须包含：
   `.agents\plugins\marketplace.json`
7. 确认该市场中至少存在：
   - `plugins\computer-use\.codex-plugin\plugin.json`
   - `plugins\chrome\.codex-plugin\plugin.json`
8. 使用当前 Codex CLI 的实际帮助信息确认插件命令语法，然后列出插件状态。不要凭记忆猜测命令。
9. 检查以下插件是否被识别：
   - `computer-use@openai-bundled`
   - `chrome@openai-bundled`

先总结诊断结果，再继续修复。只要修改是可逆且符合上述范围，不需要停下来问我。

如果确认用户目录中的 `openai-bundled` 不完整、路径错误，或者日志中存在对应的 `EBUSY` 错误：

1. 给 `config.toml` 创建带时间戳的备份。
2. 找出正在运行的 Codex Chrome `extension-host.exe`。
3. 必须核对其可执行文件路径确实位于：
   `%USERPROFILE%\.codex\plugins\cache\openai-bundled\chrome`
4. 只结束匹配该路径的扩展宿主及其直属命令行宿主。
5. 将损坏的临时插件目录改名备份，不要直接删除。
6. 创建稳定插件目录，例如：
   `%USERPROFILE%\.codex\bundled-marketplaces\openai-bundled-fixed`
7. 从当前 Codex 安装包复制完整的官方 `openai-bundled` 插件市场到稳定目录。
8. 验证稳定目录中存在有效的：
   `.agents\plugins\marketplace.json`
9. 修改 `config.toml` 中的：
   `[marketplaces.openai-bundled]`
10. 将 `source_type` 保持为 `local`，把 `source` 改为刚创建的稳定目录。
11. 保留其他所有配置。
12. 确保以下配置存在并启用：

   [plugins."computer-use@openai-bundled"]
   enabled = true

   [plugins."chrome@openai-bundled"]
   enabled = true

13. 如果 Chrome 缓存下的 `latest` 是目录连接，并且仍指向会被启动清理的 `.tmp` 目录：
   - 先确认它确实是 Junction 或符号连接。
   - 只移除连接本身，不删除连接目标内容。
   - 重新创建连接，使其指向稳定目录中的 Chrome 插件。
14. 不要把插件改成自定义插件 ID，必须保留官方 ID：
   - `computer-use@openai-bundled`
   - `chrome@openai-bundled`

1. 重新执行 Codex 插件列表命令。
2. 确认两个插件均显示：
   - installed
   - enabled
3. 如果插件存在于市场但尚未安装，先查看当前 CLI 的 `plugin add --help`，再按照当前版本支持的语法安装。
4. 检查 Computer Use 插件是否包含：
   `scripts\computer-use-client.mjs`
5. 检查 Chrome 插件是否包含：
   `scripts\browser-client.mjs`
6. 不要直接导入或调用 `@oai/sky`。
7. 必须通过 Computer Use 插件自己的 `computer-use-client.mjs` 连接。
8. 如果出现：
   `Package subpath ... is not defined by "exports"`
   说明插件和运行库可能版本不兼容。
9. 遇到版本不兼容时：
   - 先备份原脚本。
   - 优先使用本机同一 Codex 安装版本附带的兼容文件。
   - 可以检查旧插件缓存或备份中是否存在已经验证可用的官方客户端脚本。
   - 不得从不明网站下载脚本。
   - 不得自行编造协议或直接启动 `codex-computer-use.exe`。
   - 如果本机没有可信的兼容文件，停止脚本替换，建议修复或更新 Codex 安装，并明确报告该阻塞。

修复后必须实际验证，不能只检查文件是否存在。

1. 按官方 Computer Use 插件的技能说明初始化连接。
2. 调用轻量只读接口列出应用或窗口。
3. 成功返回任何应用或窗口列表，才能确认本地电脑操控连接正常。
4. 通过 Chrome 插件自带的 `browser-client.mjs` 连接浏览器。
5. 使用只读方式读取当前打开的 Chrome 标签页。
6. 不要读取 Cookie、密码、Local Storage 或其他敏感信息。
7. 如果能返回标签页标题和地址，则确认 Chrome 插件正常。
8. 检查配置中的官方插件 ID、稳定市场路径和 Chrome `latest` 连接目标。
9. 再次查看最新日志，确认没有新增：
   `bundled_plugins_marketplace_resolve_failed`
   或相同的 `EBUSY` 错误。

只有全部满足以下条件才能报告修复成功：

1. 官方插件市场清单可以正常读取。
2. `computer-use@openai-bundled` 显示已安装、已启用。
3. `chrome@openai-bundled` 显示已安装、已启用。
4. Computer Use 能实际列出本地应用或窗口。
5. Chrome 插件能实际列出当前标签页。
6. 稳定插件源不再位于启动时会被清理的损坏临时目录。
7. 所有修改前的配置和损坏目录都有备份。
8. 没有修改无关的 Codex 配置或 Windows 安全设置。

完成后请给我一份简洁报告，包括：

- 检测到的真正原因
- 修改了哪些路径和配置
- 创建了哪些备份
- Computer Use 实测结果
- Chrome 实测结果
- 是否还需要我完整退出并重新打开 Codex

如果必须重启，请把“完整退出 Codex，包括托盘进程，然后重新打开”作为最后一步。不要在完成诊断和修复前反复让我重启。
```
