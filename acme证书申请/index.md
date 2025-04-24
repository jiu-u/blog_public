# ACME证书申请

# Wiki

- [Home · acmesh-official/acme.sh Wiki · GitHub](https://wiki.acme.sh)
	- [说明 · acmesh-official/acme.sh Wiki · GitHub](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)
	- [How to issue a cert · acmesh-official/acme.sh Wiki · GitHub](https://github.com/acmesh-official/acme.sh/wiki/How-to-issue-a-cert)
	- [dnsapi · acmesh-official/acme.sh Wiki · GitHub](https://github.com/acmesh-official/acme.sh/wiki/dnsapi)

# 安装

```bash
yum update ca-certificates # centos
apt install ca-certificates # debian

curl https://get.acme.sh | sh -s email=echo@echo.xyz
wget -O -  https://get.acme.sh | sh -s email=echo@echo.xyz

# 国内使用
# https://github.com/acmesh-official/acme.sh/wiki/Install-in-China
git clone https://gitee.com/neilpang/acme.sh.git
cd acme.sh
./acme.sh --install -m email=echo@echo.xyz

source ~/.bashrc
source ~/.zshrc
```

开启自动更新
```bash
acme.sh --upgrade --auto-upgrade
acme.sh --upgrade --auto-upgrade  0 # 关闭自动更新
```

# 常见命令

```bash
acme.sh --help # help
acme.sh --list  # 列出证书
acme.sh --upgrade # 检查更新
acme.sh --issue # 申请证书
acme.sh --renew # 更新证书
acme.sh --revoke # 吊销证书
acme.sh --remove # 删除证书
acme.sh --install-cert # 安装证书
```
# 切换CA

```bash
# 切换为letsencrypt
acme.sh --set-default-ca --server letsencrypt
# 切换为zerossl
acme.sh --set-default-ca --server zerossl

apt-get install socat
```

![image-20241203010111740](https://s2.loli.net/2025/04/24/MDvlbhJzqg4EYnT.png)

[Server · acmesh-official/acme.sh Wiki · GitHub](https://github.com/acmesh-official/acme.sh/wiki/Server)

# 证书申请

```bash
sh acme.sh  --issue -d renalio.eu.org  --standalone
acme.sh --issue -d rsync.157077.xyz  --standalone
```

## HTTP认证

### 使用独立服务模式

如果服务器上没有运行任何 Web 服务，**80** 端口是空闲的，那么 **acme.sh** 还能假装自己是一个 WebServer，临时监听 **80** 端口，完成验证:

```bash
acme.sh --issue --standalone -d example.com -d www.example.com -d cp.example.com
```
### 直接签发(HTTP-01 验证)

>需要你的服务器上面已经部署了网站环境。（被申请的域名可以正常被打开）
> Acme 自动在你的网站根目录下放置一个文件, （这个文件可以被互联网访问）来验证你的域名所有权,完成验证. 然后就可以生成证书了.

只需要指定域名，并指定域名所在的网站根目录. **acme.sh** 会全自动的生成验证文件，并放到网站的根目录，验证完成后会聪明的删除验证文件，整个过程没有任何副作用。

```bash
acme.sh --issue -d mydomain.com -d www.mydomain.com --webroot /home/wwwroot/mydomain.com/
```
### TLS-ALPN-01 验证

通过在服务器上使用自定义的 TLS 握手来验证域名的所有权。这种方式适用于没有 Web 服务器的情况。

如果您没有网络服务器，可能您正在 smtp 或 ftp 服务器上， `443` 端口是空闲的。您可以使用独立的 TLS ALPN 模式。acme.sh 内置了独立的 TLS 网络服务器，它可以监听 443 端口以颁发证书。

```bash
acme.sh --issue -d yourdomain.com --alpn
acme.sh  --issue  -d example.com  --alpn --tlsport 8443
```

### 使用 Nginx 模式

如果你用的 **Nginx** 服务器，或者反代，**acme.sh** 还可以智能的从 **Nginx** 的配置中自动完成验证，你不需要指定网站根目录:

```bash
acme.sh --issue --nginx -d example.com -d www.example.com -d cp.example.com
```
### 使用 Apache 模式

如果你用的 **Apache** 服务器，**acme.sh** 还可以智能的从 **Apache** 的配置中自动完成验证，你不需要指定网站根目录:

```bash
acme.sh --issue --apache -d example.com -d www.example.com -d cp.example.com
```

**注意，无论是 Apache 还是 Nginx 模式，acme.sh 在完成验证之后，会恢复到之前的状态，都不会私自更改程序本身的配置. 好处是你不用担心配置被搞坏，也有一个缺点，你需要自己配置 SSL 项，否则只能成功生成证书，你的网站还是无法正常使用 HTTPS。**
## DNS认证
如果你没有服务器，没有公网 IP，只需要 DNS 的解析记录即可完成验证。
### 手动验证

这需要你手动在域名上添加一条 TXT 解析记录，验证域名所有权。
注意，如果使用手动验证，acme.sh 将无法自动更新证书，每次都需要手动添加解析来验证域名所有权。**如果有自动更新证书的需求，请使用自动验证（DNS API）。**

```bash
acme.sh --issue --dns -d example.com -d www.example.com -d cp.example.com
```

然后，**acme.sh** 会生成相应的解析记录显示出来，你只需要在你的域名管理面板中添加这条 TXT 记录即可。
等待解析完成之后，执行以下命令重新生成证书：

``` bash
acme.sh --renew -d mydomain.com
```
注意这里现在用的是 `--renew` 参数

### 自动验证(DNS API)(DNS-01 验证)

DNS 方式的真正强大之处在于可以使用域名解析商提供的 API 自动添加 TXT 记录，且在完成验证后删除对应的记录。
**acme.sh** 目前支持超过一百家的 DNS API。
以 DNSPod.cn 为例，你需要先登录到 [DNSPod.cn](https://dnspod.cn/)，拿到你的 DNSPod API Key 和 ID 并设置：

```bash
export DP_Id="1234"
export DP_Key="sADDsdasdgdsf"
```

现在我们可以签发通配符证书了：

```shell
acme.sh --issue --dns dns_dp -d example.com -d *.example.com
```

`DP_Id` 和 `DP_Key` 将保存在 `~/.acme.sh/account.conf` 中，并在需要时自动获取，无需手动再设置。
更详细的 DNS API 用法: https://github.com/acmesh-official/acme.sh/wiki/dnsapi

支持的 DNS API 包括但不限于：
- `dns_cf`：Cloudflare
- `dns_dp`：DNSPod
- `dns_cx`：CloudXNS
- `dns_ali`：阿里云 DNS
- `dns_aws`：AWS Route 53
- `dns_gd`：GoDaddy
- `dns_linode`：Linode
- `dns_nsupdate`：BIND DNS
- `dns_ovh`：OVH
- `dns_pdns`：PowerDNS
- `dns_gcore`：G-Core Labs
#### Cloudflare示例

[请稍候…](https://dash.cloudflare.com/profile/api-tokens)
[dnsapi · acmesh-official/acme.sh Wiki · GitHub](https://github.com/acmesh-official/acme.sh/wiki/dnsapi#dns_cf)
##### 使用限制性令牌

多个区域DNS
![image.png](https://s2.loli.net/2025/04/24/Z5ogcsyxAI7qtB6.png)

```bash
export CF_Account_ID="763eac4f1bcebd8b5c95e9fc50d010b4"
export CF_Token="Y_jpG9AnfQmuX5Ss9M_qaNab6SQwme3HWXNDzRWs"
```

##### 使用全局API密钥

![image.png](https://s2.loli.net/2025/04/24/XKVLBiZHOMPRad2.png)

```bash
export CF_Key="763eac4f1bcebd8b5c95e9fc50d010b4"
export CF_Email="alice@example.com"
```



# 查看已安装证书信息

```bash
acme.sh --info -d example.com
```
# 安装证书

证书生成好以后，我们需要把证书复制给对应的 Apache、Nginx 或其他服务器去使用。
**必须使用** `--install-cert` 命令来把证书复制到目标文件，请勿直接使用 `~/.acme.sh/` 目录下的证书文件，这里面的文件都是内部使用，而且目录结构将来可能会变化。
## Nginx示例

```bash
acme.sh --install-cert -d example.com \
	--key-file       /path/to/keyfile/in/nginx/key.pem  \
	--fullchain-file /path/to/fullchain/nginx/cert.pem \
	--reloadcmd     "service nginx reload"
```

对应的参数：
- `--key-file` ：私钥文件安装地址
- `--cert-file` ：证书文件安装地址
- `--fullchain-file` ：证书链文件安装地址
- `--reloadcmd` ：重启命令内容

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name www.exmaple.com;
    rewrite ^(.*)$ https://$host$1 permanent;
}

server {
    charset utf-8;
    listen       443 ssl;
    listen  [::]:443;
    server_name  www.exmaple.com;

    ssl_certificate    /etc/nginx/ssl/www.exmaple.com/fullchain.pem;
    ssl_certificate_key   /etc/nginx/ssl/www.exmaple.com/key.pem;

    location / {
        # rewrite ^/api-server/(.*)$ /$1 break;
        proxy_pass http://172.24.64.224:19999/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # location / {
    #     root   /usr/share/nginx/html;
    #     index  index.html index.htm;
    # }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```
## Apache示例

```bash
acme.sh --install-cert -d example.com \
	--cert-file      /path/to/certfile/in/apache/cert.pem  \
	--key-file       /path/to/keyfile/in/apache/key.pem  \
	--fullchain-file /path/to/fullchain/certfile/apache/fullchain.pem \
	--reloadcmd     "service apache2 force-reload"
```
# 更新证书

目前证书每 60 天自动更新，你无需任何操作。
但是你也可以强制续签证书：
```bash
acme.sh --renew -d example.com --force
```
# 吊销证书

```bash
acme.sh --revoke -d dnomd343.top
```
# 删除证书

```bash
acme.sh --remove -d dnomd343.top
```
# 链接🔗
## FreeSSL
[FreeSSL.cn - 一个提供免费HTTPS证书申请的网站](https://freessl.cn/)
## 项目地址
[GitHub - acmesh-official/acme.sh: A pure Unix shell script implementing ACME client protocol](https://github.com/acmesh-official/acme.sh)
