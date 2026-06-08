# 零成本搞定 Obsidian 图床：Cloudflare R2 免费方案

![](assets/QV9TdAoYroBz9DxQPaLcCMFknHh_001.png)

上次那篇图床教程发出去之后，评论区好多人问：

「腾讯云那个免费额度到期了怎么办？」 「有没有完全免费的方案？」

有。今天这篇就是穷鬼版——用 Cloudflare R2 做图床，永久免费。

先说结论： • 免费存储 10 GB，永久有效（总量上限，不是每月额度） • 流量费：零。没有出口带宽费 • 每月 100 万次写入 + 1000 万次读取，个人用根本用不完

唯一门槛：注册时要绑一张 Visa / MasterCard 或 PayPal，不会扣钱，只是验证身份。

整个配的过程还是三步： ① 注册 Cloudflare，开一个 R2 存储桶 — 5 分钟 ② 装 PicList，连上 R2 — 3 分钟 ③ 装 Obsidian 插件，粘图自动上传 — 1 分钟

跟上次腾讯云的教程几乎一样，只是换了个云服务商。不确定选哪个？

• 腾讯云 COS — 50G 免费空间，半年有效，发公众号零障碍。主要发公众号的选这个 • Cloudflare R2 — 10G 免费空间，永久有效，流量也免费。想长期零成本的选这个

确定选 R2？跟着走就行。

## 𝟭. 注册 Cloudflare，开一个 R2 存储桶

Cloudflare 这公司你可能没听过，但你平时上网大概率已经在用它们的服务了。

R2 是它家的对象存储，跟腾讯云 COS、阿里云 OSS 一个东西，但流量免费。

按顺序来，五步：

## 𝗦𝘁𝗲𝗽 𝟭: 注册 Cloudflare 账号

打开 [https://dash.cloudflare.com/sign-up](https://dash.cloudflare.com/sign-up) ，用邮箱注册就行。

## 𝗦𝘁𝗲𝗽 𝟮: 进入 R2 并绑定支付方式

注册完登录后，在左侧菜单找到「R2 对象存储」，点进去。

第一次进来会让你绑定支付方式。Visa / MasterCard 信用卡或者 PayPal 都行。

![](assets/QV9TdAoYroBz9DxQPaLcCMFknHh_002.png)

⚠️ 绑卡不会扣钱，只是验证身份。免费额度内用不花钱。

绑定成功后，你会看到 R2 的免费额度：10 GB 存储上限 + 每月 100 万次写入 + 1000 万次读取，永久有效。

## 𝗦𝘁𝗲𝗽 𝟯: 创建存储桶

点「创建存储桶」，两个东西要填： ❶ 名称：随便起，比如 obsidian-img ❷ 位置：选「自动」就行，Cloudflare 会自动分配离你最近的区域

![](assets/QV9TdAoYroBz9DxQPaLcCMFknHh_003.png)

⚠️ 存储类型选「标准」，只有标准类型才享受免费额度。

![](assets/QV9TdAoYroBz9DxQPaLcCMFknHh_004.png)

点创建，搞定。

![](assets/QV9TdAoYroBz9DxQPaLcCMFknHh_005.png)

## 𝗦𝘁𝗲𝗽 𝟰: 开启公开访问

刚创建的存储桶默认是私有的，外面访问不了。得开一下公开访问，不然图片链接打不开。

进入存储桶 → 设置 → 找到「公共开发 URL」，点击「启用」。

![](assets/QV9TdAoYroBz9DxQPaLcCMFknHh_006.png)

开启后你会得到一个类似这样的域名： [https://pub-xxxxx.r2.dev](https://pub-xxxxx.r2.dev/)

这就是你的图片访问地址。记下来，待会配 PicList 要用。

⚠️ 注意：[R2.dev](https://r2.dev/) 域名发 X、发博客没问题，但**发公众号可能被拦截**（公众号对非腾讯系域名不太友好）。如果你主要发公众号，建议用腾讯云 COS 方案。

![](assets/QV9TdAoYroBz9DxQPaLcCMFknHh_007.png)

接下来拿 API 令牌。这一步相当于上次腾讯云教程里拿密钥那一步

## 𝗦𝘁𝗲𝗽 𝟱: 创建 API 令牌

回到 R2 首页（不是存储桶里面），找到右下角「API Tokens」旁边的「Manage」，点进去，选「创建 Account API 令牌」。

![](assets/QV9TdAoYroBz9DxQPaLcCMFknHh_008.png)

四个东西要填：

❶ 令牌名称：随便填，比如 R2ObsidianToken ❷ 权限：选「对象读和写」 ❸ 指定存储桶：选你刚才创建的那个桶（比如 obsidian-img） ❹ TTL（有效期）：选「永久」就行

![](assets/QV9TdAoYroBz9DxQPaLcCMFknHh_009.png)

![](assets/QV9TdAoYroBz9DxQPaLcCMFknHh_010.png)

点「创建 API 令牌」，会给你三个东西：

• 访问密钥 ID（Access Key ID）— 相当于用户名 • 机密访问密钥（Secret Access Key）— 相当于密码 • 终结点（Endpoint）— 类似 [https://xxxxxx.r2.cloudflarestorage.com](https://xxxxxx.r2.cloudflarestorage.com/)

⚠️ 这三个信息只显示这一次！务必复制保存好。建议直接截图或存到备忘录。

![](assets/QV9TdAoYroBz9DxQPaLcCMFknHh_011.png)

点完成。到这里 Cloudflare R2 搞定了。三个东西记好：访问密钥 ID、机密访问密钥、终结点。

## 𝟮. 装 PicList，连上 R2

跟上次腾讯云教程一样，用 PicList 做上传工具。上次已经装过的，直接跳到下面配置那一步。

没装过的去 GitHub 下载：[https://github.com/Kuingsmile/PicList](https://github.com/Kuingsmile/PicList)

Mac 用户也可以终端敲一行： brew install piclist --cask

装完打开。

打开 PicList，左边找到「图床」，这次不选「腾讯云COS」了，选「Amazon S3」。

为什么选 Amazon S3？因为 R2 兼容 S3 协议，PicList 通过 S3 配置就能直连 R2。

![](assets/QV9TdAoYroBz9DxQPaLcCMFknHh_012.png)

新建一个配置，填这 6 个字段：

❶ 配置名：随便填，比如 obsidian-cloudflare-r2 ❷ Access Key ID：填刚才拿到的「访问密钥 ID」 ❸ Secret Access Key：填刚才拿到的「机密访问密钥」 ❹ Bucket：填你的存储桶名称，比如 obsidian-img ❺ 自定义节点（Endpoint）：直接粘贴创建令牌时给的「终结点」地址 ❻ 自定义域名：填上面 Step 4 里记住的那个「公共开发 URL」域名，比如 [https://pub-xxxxx.r2.dev](https://pub-xxxxx.r2.dev/)

![](assets/QV9TdAoYroBz9DxQPaLcCMFknHh_013.png)

其他字段全部留空，不用管。

填完点「确定」，然后「设为默认图床」。

试一下能不能用。切到 PicList 的「上传」页面，随便拖一张图片进去。

如果上传成功，下面会显示一个链接，类似： [https://pub-xxxxx.r2.dev/xxx.png](https://pub-xxxxx.r2.dev/xxx.png)

复制链接到浏览器打开，图片能显示就说明通了。

![](assets/QV9TdAoYroBz9DxQPaLcCMFknHh_014.png)

## 𝟯. 装 Obsidian 插件

跟上次腾讯云教程一模一样 — 装一个叫 Image auto upload 的插件就行。

装过的直接跳到下面试效果。没装过的看[上篇教程](https://x.com/alin_zone/status/2029762985628160110?s=20)的第三步。

搞定，试试效果。

确保 PicList 在后台开着，然后在 Obsidian 里随便打开一篇笔记，复制一张图片，粘贴。

笔记里会直接变成一个云端链接：

![]([https://pub-xxxxx.r2.dev/xxx.png](https://pub-xxxxx.r2.dev/xxx.png))

![](assets/QV9TdAoYroBz9DxQPaLcCMFknHh_015.png)

以后你写的每篇文章，图片都自动在云端了。永久免费，流量也免费。

就这样，三步搞定：

① Cloudflare 开个 R2 存储桶，拿到 API 令牌 ② PicList 选 Amazon S3，填上令牌连通 R2 ③ Obsidian 插件跟上次一样，装过就不用动

以后粘图片就是上传图片，写完直接发。

💡 几个小贴士：

• PicList 必须开着才能自动上传，建议设成开机启动 • R2 免费额度：10 GB 存储（总量上限） + 零流量费，永久有效。个人图床绰绰有余 • [R2.dev](https://r2.dev/) 域名部分地区可能偏慢。有自己域名的话，可以在存储桶设置的「自定义域」里绑一个，比如 [img.yourdomain.com](https://img.yourdomain.com/)，访问更快更稳 • 已经用腾讯云 COS 的不用换 — 两个方案各有优势：腾讯云发公众号最省心，R2 胜在永久免费 • 同时有腾讯云和 R2 的话，PicList 支持多图床切换，在托盘图标右键就能切

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/BCMSwuliwi9JJzk8HPCcgFK5nvd> ｜ 归档：2026-06-04
