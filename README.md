# RustDesk 自建中继教程（给人和 Agent）

把下面这段话完整复制给任意 Agent，再提供一台有公网 IP 的 VPS（有没有域名都行）。Agent 会来读这个仓库，并按里面的步骤把中继搭好。

```text
请阅读 GitHub 仓库 https://github.com/atongrun/rustdesk-relay-guide ，并严格按照仓库说明，在我已有的 VPS 上部署并配置好 RustDesk 自建中继（hbbs + hbbr）。这台 VPS 可能有域名，也可能只有公网 IP、没有域名。开始前先向我确认 SSH 登录方式和域名（或公网 IP），不要编造地址或密钥。完成后把客户端应填写的 ID 服务器、中继服务器和 Key 给我，并按仓库中的标准验证连通。
```

English (optional):

```text
Read https://github.com/atongrun/rustdesk-relay-guide and follow it to deploy a self-hosted RustDesk relay (hbbs + hbbr) on my existing VPS. The VPS may or may not have a domain. Ask me for SSH access and the domain or public IP first. Do not invent addresses or keys. When finished, give me the client ID server, relay server, and Key, and verify connectivity using the repo's checklist.
```

## 这个仓库是做什么的

用一台普通 Linux VPS，按官方开源方案部署 RustDesk 的 ID 服务器（`hbbs`）和中继服务器（`hbbr`），并配好各端客户端。适合跨省、手机热点、没有公网 IP、直连经常失败的情况：两端都连到你自己的中继上，国内就能稳定转起来。

人和 Agent 都读这些文件：

- [AGENTS.md](AGENTS.md)：给 Agent 的操作手册，拿到任务后按这个执行
- [docs/deploy.md](docs/deploy.md)：在 VPS 上安装、开放端口、处理有域名和没有域名两种情况
- [docs/clients.md](docs/clients.md)：Windows / macOS / Linux / 移动端怎么填服务器
- [docs/troubleshoot.md](docs/troubleshoot.md)：未就绪、重启后又跳回官方服务器、DNS 和本地代理等常见坑

客户端请到官网下载：[https://rustdesk.com/zh-cn/](https://rustdesk.com/zh-cn/)，不要用来路不明的安装包。


## 还没有服务器？

这套中继我是在腾讯云国内轻量上跑的，2 核 2G 就够用，只跑中继很闲。套餐和购买入口在我的小站 [aibuy · 腾讯云轻量](https://aibuy.bid/vps/tencent/)：

- 国内轻量 2 核 2G：活动价 99 元/年（2026 年 10 月 7 日核对），还写明可以同价续费 1 次，适合打算用久一点
- 大促期间官网还有 4 核 4G 新客秒杀。我 2026 年 11 月 21 日在双 11 活动页看到过 38 元/年；能抢到很值，抢不到用 99 这档也够

这是返利链接，你还是在腾讯云下单，售后也走腾讯云。活动资格和成交价以结算页为准。

本仓库只提供搭建方法，不包含任何真实服务器地址、私钥或公钥。密钥必须从你自己的 VPS 上读取。
