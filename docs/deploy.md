# 在 VPS 上部署 RustDesk 中继

适合一台有公网 IPv4 的 Linux VPS。1 核 1G 通常就能跑中继；2 核 2G 更从容，还可以顺便跑点别的服务。以下以 Ubuntu / Debian 为例。

## 1. 需要开放的端口

云厂商安全组和主机防火墙（如 ufw）都要放行。只开 TCP、漏掉 UDP，是最常见的「看起来端口都通、客户端却显示未就绪」原因。

| 端口 | 协议 | 用途 |
| --- | --- | --- |
| 21115 | TCP | NAT 测试 |
| 21116 | TCP | ID / 打洞服务（hbbs） |
| 21116 | UDP | 心跳与注册（必须开） |
| 21117 | TCP | 中继（hbbr） |
| 21118 | TCP | WebSocket 备用（建议开） |
| 21119 | TCP | WebSocket 备用（建议开） |

其中最关键的是 `21115/TCP`、`21116/TCP`、`21116/UDP`、`21117/TCP`。

用 ufw 的示例：

```bash
ufw allow 21115/tcp
ufw allow 21116/tcp
ufw allow 21116/udp
ufw allow 21117/tcp
ufw allow 21118/tcp
ufw allow 21119/tcp
ufw status
```

安全组里同样要有对应规则。改完后不要只用 TCP 端口探测来判断成功。

## 2. 有域名，或没有域名

### 没有域名

可以直接用 VPS 的公网 IPv4。客户端的 ID 服务器填 IP，中继服务器填 `IP:21117`。后面步骤完全一样，只是不配 DNS。

### 有域名

给这台 VPS 加一条 A 记录，指向公网 IPv4。如果是子域名，主机记录填子域前缀，例如 `rd`，完整名字是 `rd.example.com`。

注意：

- RustDesk 走自定义 TCP/UDP 端口，域名必须是纯 DNS 解析
- 如果域名在 Cloudflare 等平台，代理状态必须关闭（灰云 / DNS only），不能开橙云或同类 HTTP 代理
- 用公共 DNS 验证，解析结果应是这台 VPS 的真实公网 IP：

```bash
# 把 rd.example.com 换成实际域名
dig +short rd.example.com A @1.1.1.1
dig +short rd.example.com A @8.8.8.8
```

## 3. 安装 Docker 与 Compose 插件

```bash
apt update
apt install -y curl ca-certificates

install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" > /etc/apt/sources.list.d/docker.list

apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
docker --version
docker compose version
```

若是 Debian，把上面源里的 `ubuntu` 和 `$VERSION_CODENAME` 按 Debian 官方文档调整。已装好 Docker 则可跳过。

## 4. 用官方 Compose 启动服务

官方开源配置：<https://rustdesk.com/oss.yml>。它会在 host 网络上启动 `hbbs` 和 `hbbr`。

```bash
mkdir -p /opt/rustdesk-server
cd /opt/rustdesk-server
wget https://rustdesk.com/oss.yml -O compose.yml
docker compose -f compose.yml up -d
```

## 5. 部署后检查

```bash
cd /opt/rustdesk-server
docker compose -f compose.yml ps
ls -l data
cat data/id_ed25519.pub
ss -lntup | grep 211
```

成功时应看到：

- `hbbs`、`hbbr` 都是运行中
- `21115`、`21116`、`21117`、`21118`、`21119` 的 TCP 在监听
- `21116/UDP` 在监听
- `data/id_ed25519.pub` 已生成，内容就是客户端要填的 Key

把公钥交给用户保存。不要把 `data/id_ed25519` 私钥提交到 Git 或发到公开场合。

## 6. 告诉用户客户端怎么填

- ID 服务器：有域名填域名，没有填公网 IP
- 中继服务器：`域名或IP:21117`
- API 服务器：留空
- Key：`id_ed25519.pub` 的完整一行

详细填写方式和各系统注意点见 [clients.md](clients.md)。
