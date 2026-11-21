# 排障

先不要重装。按下面的顺序查，很多「连不上」其实是端口、配置不一致或本地代理，而不是中继程序挂了。

## 1. 客户端全部显示未就绪

先在 VPS 上确认服务还在、端口还在听：

```bash
cd /opt/rustdesk-server
docker compose -f compose.yml ps
ss -lntup | grep 211
```

如果 TCP 探测 `21116`、`21117` 都成功，客户端却一直未就绪，优先查 **21116/UDP**：

- 云安全组有没有放行 `21116/UDP`
- 主机防火墙（ufw / firewalld / iptables）有没有放行 `21116/UDP`
- `ss` 里能不能看到 `21116/UDP` 在监听

TCP 通、UDP 不通时，设备注册和心跳会失败，表现就是未就绪。把 UDP 放行后再彻底重启客户端。

## 2. Linux 重启后又连回官方服务器

典型现象：刚改完能用，重启 `rustdesk` 服务或重启系统后，又连不上，或日志跳回 `rs-ny.rustdesk.com`。

原因通常是用户目录里的 `RustDesk2.toml` 已经改成自建，但 root 下还是默认官方地址。服务启动后会 `sync config from root`，把运行态覆盖回去。

处理：把 `/home/用户/.config/rustdesk/RustDesk2.toml` 和 `/root/.config/rustdesk/RustDesk2.toml` 改成一样，然后 `systemctl restart rustdesk.service`。详见 [clients.md](clients.md)。

升级或重装客户端后，再检查一次这两份文件。

## 3. 有域名却解析不对

确认 A 记录指向 VPS 的真实公网 IP，并且没有开 HTTP 代理 / CDN（例如 Cloudflare 必须是 DNS only）。用公共 DNS 查询：

```bash
dig +short your.domain.example A @1.1.1.1
dig +short your.domain.example A @8.8.8.8
```

结果应该是 VPS 的公网 IPv4。如果是代理节点的地址，RustDesk 的自定义端口会被破坏。

刚改完 DNS 时，先等解析生效，再重启客户端。

## 4. 本机代理把去中继的流量拐走

如果电脑上开着带虚拟网卡或 fake-ip 的代理，域名可能被解析成 `198.18.0.x` 这类虚假地址，RustDesk 看起来像端口能通，实际上注册和心跳是异常的。

处理原则：去中继服务器的流量要直连，不要再进代理。

- 在代理规则里，把中继的域名和公网 IP 设为直连
- 不要把 `198.18.0.x` 这类假地址当成 RustDesk 的本机局域网地址写进配置
- 临时排查时，可先关掉代理或在 hosts 里把域名指到真实公网 IP，验证通了再改成长期的直连规则

改完后刷新 DNS，并彻底退出或重启 RustDesk。

## 5. Windows 上改完配置却不生效

可能有多个 `rustdesk.exe` 残留。停掉服务，结束所有 `rustdesk.exe`，再启动服务。见 [clients.md](clients.md) 里的干净重启命令。

## 6. 日志里出现 API heartbeat 失败

如果 API 服务器本来就留空，日志里可能周期性出现类似 `HTTP POST ... :21114/api/heartbeat failed` 的提示。开源自建场景下，这条单独出现通常不影响远控。

不要只凭这一条判断中继不可用。要看设备是否已注册、能否真正建立连接、是否出现视频画面。

## 7. 建议的确认命令

VPS：

```bash
cd /opt/rustdesk-server
docker compose -f compose.yml ps
cat data/id_ed25519.pub
ss -lntup | grep 211
```

有域名时，在自己的电脑上确认解析：

```bash
dig +short your.domain.example A @1.1.1.1
```

Linux 被控：

```bash
journalctl -u rustdesk.service -n 100 --no-pager
```

重点看有没有连到自建服务器，有没有 `sync config from root`，有没有跳回官方域名。
