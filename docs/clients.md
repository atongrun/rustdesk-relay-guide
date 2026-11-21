# 配置 RustDesk 客户端

先到官网下载客户端：<https://rustdesk.com/zh-cn/>。不要用搜索引擎随便下的安装包。

下面的 `SERVER` 请替换成：

- 有域名：例如 `rd.example.com`
- 没有域名：VPS 的公网 IPv4，例如 `203.0.113.10`（这是文档示例，不要照抄）

`KEY` 替换成 VPS 上 `data/id_ed25519.pub` 的完整内容。

## 通用三项

在「设置 → 网络」或「ID / 中继服务器」里填写：

- ID 服务器：`SERVER`
- 中继服务器：`SERVER:21117`
- API 服务器：留空
- Key：`KEY`

填完后点确认，然后**彻底退出 RustDesk 再打开**。只关掉窗口往往不够，服务可能还在后台用旧配置。

## Windows

1. 按上面三项填写并确认
2. 退出托盘图标里的 RustDesk，必要时在服务里重启 `RustDesk`
3. 如果同时开着多个 `rustdesk.exe`，先停服务，再结束所有进程，然后重新启动，避免旧会话占用配置

干净重启示例（管理员命令提示符）：

```bat
sc stop RustDesk
timeout /t 3
taskkill /F /IM rustdesk.exe
timeout /t 2
sc start RustDesk
```

## macOS

同样填写三项，然后从菜单栏完全退出再打开。确认状态不再是「未就绪」，网络设置里看到的是自建服务器，而不是官方公共服务器。

## Linux（尤其是用 systemd 跑服务时）

如果只改了当前用户的配置，服务启动后可能会从 root 的配置再同步回去，最后又连到官方服务器。必须让两份配置一致。

常见路径（把 `USER` 换成实际用户名）：

- `/home/USER/.config/rustdesk/RustDesk2.toml`
- `/root/.config/rustdesk/RustDesk2.toml`

关键字段示例：

```toml
rendezvous_server = 'SERVER:21116'

[options]
custom-rendezvous-server = 'SERVER'
relay-server = 'SERVER:21117'
key = 'KEY'
```

改完后重启服务：

```bash
systemctl restart rustdesk.service
```

检查两份文件是否一致，并看日志有没有又切回 `rs-ny.rustdesk.com`：

```bash
cat /home/USER/.config/rustdesk/RustDesk2.toml
sudo cat /root/.config/rustdesk/RustDesk2.toml
journalctl -u rustdesk.service -n 100 --no-pager
```

日志里如果先出现自建服务器，随后出现 `sync config from root`，再跳到官方域名，就是两份配置没改齐。

## 安卓与 iOS

在网络 / 服务器设置里填写同样的三项。iPhone 目前主要当主控。改完后强行停止应用再打开。

## 填完怎么确认

- 客户端不再长期显示「未就绪，请检查网络连接」
- 设置里的服务器是自建地址，不是 `rs-ny.rustdesk.com` 这类官方地址
- 用一台设备的 ID，从另一台能连上
