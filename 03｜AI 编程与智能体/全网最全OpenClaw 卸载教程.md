# 全网最全OpenClaw 卸载教程

前面教程里面教会了大家如何安装Openclaw和使用高阶用法，让它成为了你工作的好帮手，甚至是你的`好女友`。但是更多的小伙伴都是为了蹭热度，去安装使用的。再发下巨量的使用Token之后无力承担，或者是完全就不知道从和下手，玩了两下就腻了。

这也就是我为什么出篇教程的意义，很多人手足无措的时候不知道怎么卸载。这里我就把官方的、Win、Mac、Linux的卸载方式都集齐了，不管是什么设备都可以找到自己的卸载方案。

![](assets/b8b9ef832f9202d01fad1a1015b4b951_MD5.webp)

> 这里顺便告诉大家一个商机，现在闲鱼还没有大批量Openclaw卸载服务，学了我这篇教程就可以去给人家收费卸载了。

---

## 一、官方卸载方式

这里如果你是使用官方脚本安装的，最推荐的还是官方的自动卸载脚本（因为最简单也是最方便的）：

```Plaintext
# 交互式
openclaw uninstall

# 非交互
openclaw uninstall --all --yes --non-interactive

# npx 直接跑
npx -y openclaw uninstall --all --yes --non-interactive
```

> 说明：这条命令成功，只是移除了很多核心组件会被移除（比如：gateway service、agent runtime、config/state、workspace 等），**但如果你想彻底删除干净** 还需要做接下来的“扫尾检查”，才能确保你的电脑或者服务器的干净整洁。

---

## 二、按步骤卸载（如果对自己足够自信）

按顺序做，确保服务先停止，再删除文件，最后移除 CLI/应用程序。

### 1) 停止 gateway

```Plaintext
openclaw gateway stop
```

### 2) 卸载 gateway 启动项

```Plaintext
openclaw gateway uninstall
```

### 3) 删除状态、配置与工作区

```Plaintext
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
rm -rf ~/.openclaw/workspace
# 若用过 profile：
rm -rf ~/.openclaw-*
```

> 这里如果遇到权限问题可以在命令的前面加上`sudo`，这样就可以拥有最高权限轻松删除文件残留。但是也要注意删除命令的文件夹，别手贱点击过快可能让你的服务器或电脑变成砖，一定一定要小心。

### 4) 根据安装方式卸载 CLI / GUI

- npm / pnpm / bun
- npm rm -g openclaw  
pnpm remove -g openclaw  
bun remove -g openclaw
- Homebrew / cask（macOS）
- brew uninstall openclaw-cli # CLI 包  
brew uninstall --cask openclaw # 带 GUI 的 App

> 如果是源码安装的方式（git clone）**先用** `openclaw gateway uninstall` **卸服务，确保服务停掉然后再把源码目录删除**（否则可能服务继续引用已经被删除的路径，造成删除错误）。

---

## 三、踩坑分享

如果你删掉了 CLI，然后发现后台网关仍然在跑（或重启后又回来了，怎么都无法关闭卸载），那就只能按系统执行强制卸载：

### macOS

```Plaintext
# 强制停止并移除用户 launch agent（替换 $UID）
launchctl bootout gui/$UID/ai.openclaw.gateway
rm -f ~/Library/LaunchAgents/ai.openclaw.gateway.plist
```

> 有时也可能是 com.openclaw.gateway 或 com.clawdbot.gateway，这里需要仔细检查对应的包名的名字别搞错了。而且`rm -f`也要小心使用。

### Linux

```Plaintext
# 通常名称：openclaw-gateway.service，也有可能不一样需要看具体服务名称
systemctl --user disable --now openclaw-gateway.service
rm -f ~/.config/systemd/user/openclaw-gateway.service
systemctl --user daemon-reload

# 若安装为 system 服务
sudo systemctl disable --now openclaw-gateway.service
sudo rm -f /etc/systemd/system/openclaw-gateway.service
sudo systemctl daemon-reload
```

### Windows

以管理员运行 PowerShell：

```Plaintext
# 计划任务
schtasks /Delete /F /TN "OpenClaw Gateway"

# 删除gateway 启动脚本
Remove-Item -Force "$env:USERPROFILE\.openclaw\gateway.cmd"
```

> 在不同发行或者版本不同的情况下 unit/plist 名称，有可能是 `openclaw` / `clawdbot` / `gateway` ，主要是改名太多次了，也有人安装的是老版本，没有升级所以可以都查一遍然后确定具体名称。

---

## 四、深度清理

1. **最后查找残留进程确认监听端口**
2. macOS / Linux
3. ps aux | grep -i openclaw  
lsof -iTCP -sTCP:LISTEN -P -n | grep -i openclaw  
ss -lptn | grep -i openclaw
4. **检查 systemd / launchctl / crontab / scheduled-tasks**

- `systemctl --user list-units | grep -i openclaw`
- `launchctl list | grep -i openclaw`
- `crontab -l | grep -i openclaw`

1. **如果是使用Docker 容器那就是最简单的，只需要几行命令。因为环境是隔离的docker不会污染到宿主机，所以也不需要后续清理垃圾的操作**
2. docker ps -a | grep openclaw  
docker rm -f  
docker rmi

## 五、最后的提醒

- **服务器篇**：如果是购买的云服务，检查是否后续还要使用，如果不需要就要取消服务器防止二次收费。如果是活动办理的，查看云服务厂商开通的时候有没有附带其他服务；比如Openclaw优惠套餐、Token大礼包是否会二次收费。
- **模型API篇**：如果决定后期不再使用模型或者厂商的API，防止泄露我建议去对应厂商的API KEY管理后台去删除不要使用的API KEY，防止二次流失造成不必要的经济损失。
- **备份篇**：如果想下次还要玩Openclaw，可以在卸载之前配置Openclaw的配置文件和workspace，如果是有意义的聊天记录也可以选择导出聊天记录，这样在下次安装的时候想要继续玩，就可以使用官方的恢复功能。原来那只小龙虾它就这样回来了。