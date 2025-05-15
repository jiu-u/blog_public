# FRPC安装与配置


# 文档

[文档 | frp](https://gofrp.org/zh-cn/docs/)

# 安装

## Windows

- [GitHub - luckjiawei/frpc-desktop: frp 跨平台桌面客户端，可视化配置，轻松实现内网穿透！ 支持所有 frp 版本](https://github.com/luckjiawei/frpc-desktop)
- [GitHub - koho/frpmgr: Windows 平台的 FRP GUI 客户端 / A user-friendly desktop GUI client for FRP on Windows.](https://github.com/koho/frpmgr)
- [GitHub - VaalaCat/frp-panel: a multi node frp webui and for https://github.com/fatedier/frp server and client management, which makes this project a Cloudflare Tunnel/Tailscale Funnel/Ngork platform and agent open source alternative](https://github.com/VaalaCat/frp-panel)
- [Frpee-Gui-frp 客户端-内网穿透软件-免费内网穿透](https://frpee.com/gui/#)
- [GitHub - q1285514609/frp-client-gui: 可视化配置 frp 客户端（支持修改新增配置，阿里云解析），主要针对需要映射 http 请求的需求编写,如映射 http 的接口地址或者 web 服务地址](https://github.com/q1285514609/frp-client-gui)

## Linux

下载并安装

```bash
mkdir -p /opt/frp
wget "https://ghfast.top/https://github.com/fatedier/frp/releases/download/v0.61.2/frp_0.61.2_linux_amd64.tar.gz" -o frp.tar.gz
tar -zxf frp.tar.gz

mv frp_0.61.2_linux_amd64/* /opt/frp/
chmod +x /opt/frp/frpc

rm -rf frp_0.61.2_linux_amd64/
rm frp.tar.gz
```

配置 `service` 文件
`/etc/systemd/system/frpc.service`

```toml
[Unit]
Description = frp server
After = network.target syslog.target
Wants = network.target

[Service]
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/opt/frp/frpc -c /opt/frp/frpc.toml
LimitNOFILE=1048576

[Install]
WantedBy=multi-user.target
```

# 配置示例(Frp 端配置 SSL，不使用 Nginx)

```toml
serverAddr = "frpc.example.com"
serverPort = 7000
auth.method = "token"
auth.token = "JxqUk3d8dajkHVQJ26"

transport.heartbeatInterval = 30
transport.heartbeatTimeout = 90

log.to = "frpc.log"
log.level = "info"
log.maxDays = 3
webServer.addr = "0.0.0.0"
webServer.port = 57400
transport.tls.enable = false

[[proxies]]
name = "df_http_cert"
type = "http"
localIP = "127.0.0.1"
localPort = 18001
customDomains=["public.example.com"]
subdomain=""

[[proxies]]
name = "domain2_https"
type = "https"
customDomains = ["public.example.com"]
[proxies.plugin]
type = "https2http"
localAddr = "127.0.0.1:18001"
crtPath = "/data/cert/public.example.com/fullchain.pem"
keyPath = "/data/cert/public.example.com/key.pem"
hostHeaderRewrite = "127.0.0.1"
requestHeaders.set.x-from-where = "frp"


[[proxies]]
name = "df_http_nx1"
type = "http"
localIP = "127.0.0.1"
localPort = 18002
customDomains=["xxx.example.com"]
subdomain=""
```

# 启动

```bash
systemctl restart frpc
systemctl enable frpc
systemctl status frpc
```

# 查看日志

```bash
# 查看最新日志
sudo journalctl -u frpc

# 查看最新日志并实时刷新
sudo journalctl -u frpc -f

# 查看指定时间段的日志
sudo journalctl -u frpc --since "2023-01-01" --until "2023-01-02"

# 查看最近100行日志
sudo journalctl -u frpc -n 100
```

# 可视化面板

http://ip:57400/static/#/

# 脚本

```bash
curl -fsSL https://ghfast.top/https://raw.githubusercontent.com/jiu-u/amongst/refs/heads/main/scripts/frpc.sh | bash
```

# 进阶使用 Frpc 搭配 Nginx(SSL 终止端)

## Frpc 配置

```toml
serverAddr = "frps.example.com"
serverPort = 7000
auth.method = "token"
auth.token = "12345678"

transport.heartbeatInterval = 30
transport.heartbeatTimeout = 90

log.to = "frpc.log"
log.level = "info"
log.maxDays = 3
webServer.addr = "0.0.0.0"
webServer.port = 57400
transport.tls.enable = false


[[proxies]]
name = "http_nginx"
type = "http"
localIP = "127.0.0.1"
localPort = 80
customDomains = ["www.example.com"]
subdomain = ""


[[proxies]]
name = "https_nginx"
type = "https"
localIP = "127.0.0.1"
localPort = 443
customDomains = ["www.example.com"]
```

## Nginx 端配置

```nginx
server {
    listen       80;
    listen  [::]:80;
    server_name  www.example.com;

    rewrite ^(.*)$ https://$host$1 permanent;
}

server {
    charset utf-8;
    listen       443 ssl;
    listen  [::]:443;
    server_name  www.example.com;

    ssl_certificate    /etc/nginx/ssl/www.example.com/fullchain.pem;
    ssl_certificate_key   /etc/nginx/ssl/www.example.com/key.pem;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }


    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

