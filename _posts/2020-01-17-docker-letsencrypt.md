---
title: letsencrypt申请以及过期后重新申请https证书
date: 2020-01-17 13:30:09
categories:
- https
tags:
- certbot
- letsencrypt
- docker
---

## 背景

用docker版本的caddy安装过，中间申请证书一步各种报错。于是改用cerbort手工申请证书。



## 前言

Let’s Encrypt 的通配符证书 (Wildcard Certificate) 于 2018 年 3 月中旬上线，可以免费申请，安装和使用，本次记录一下基本的步骤。

申请通配符证书需要 ACME v2 协议的客户端，官方推荐使用 [Certbot](https://certbot.eff.org/)。

[Certbot](https://certbot.eff.org/) 官网也有安装步骤供您参考，本文只记录安装通配符证书的基本操作步骤，与其他软件集成和配置步骤不在本文范围



### 安装 Certbot

先安装 EPEL 仓库（因为 certbot 在这个源里，目前还没在默认的源里）

```shell
$ sudo yum install epel-release
```

安装 certbot

```shell
$ sudo yum install certbot
```

查看 certbot 版本，因为 ACME v2 要在 `certbot 0.20.0` 以后的版本支持。

```shell
$ certbot --version
certbot 0.37.2
```

本文时， certbot 版本时 0.37.2，可以支持 ACME v2 协议。



### 申请证书



#### 成功实例(非通配符)

算了通配符先改成www，也就是对单域名证书（一级域名和2级域名www.）

```shell
$ sudo certbot certonly -d yourdomain.xyz -d www.yourdomain.xyz --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
```

主要参数说明：

1. certonly 是 certbot 众多插件之一，您可以选择其他插件。
2. -d 为那些主机申请证书，如果是通配符，输入 *.yourdomain.com。
   `注意：本文还申请了 yourdomain.com 这是为了避免通配符证书不匹配`。
3. –preferred-challenges dns，使用 DNS 方式校验域名所有权。
   `注意：通配符证书只能使用 dns-01 这种方式申请`。



~~~shell
$ sudo certbot certonly -d yourdomain.xyz -d www.yourdomain.xyz --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory

Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): xxxx@gmail.com  # 1.首先输入邮件地址
Starting new HTTPS connection (1): acme-v02.api.letsencrypt.org

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(A)gree/(C)ancel: A     # 2.服务协议，输入同意

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: N   # 3. 是否共享你的email，输入N
Obtaining a new certificate
Performing the following challenges:
dns-01 challenge for yourdomain.xyz
dns-01 challenge for www.yourdomain.xyz

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
NOTE: The IP of this machine will be publicly logged as having requested this
certificate. If you're running certbot in manual mode on a machine that is not
your server, please ensure you're okay with that.

Are you OK with your IP being logged?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y    # 4.IP记录选择同意Y

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# 第一个TXT记录_acme-challenge
Please deploy a DNS TXT record under the name
_acme-challenge.yourdomain.xyz with the following value:

79MbLIbz1frxB5YJd5cL-RbvfL_Cch90_2HBQgDgHpQ

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue    # 5.等dns记录生效后按回车

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# 第二个TXT记录_acme-challenge.www
Please deploy a DNS TXT record under the name
_acme-challenge.www.yourdomain.xyz with the following value:

nyUHptibTcfwPXVqyfTCL4LNtIAaB8Alna45oQddW5Q

Before continuing, verify the record is deployed.
(This must be set up in addition to the previous challenges; do not remove,
replace, or undo the previous challenge tasks yet. Note that you might be
asked to create multiple distinct TXT records with the same name. This is
permitted by DNS standards.)

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
Waiting for verification...
Cleaning up challenges
#总算成功了。
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/yourdomain.xyz/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/yourdomain.xyz/privkey.pem
   Your cert will expire on 2019-12-24. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
~~~



此时去 DNS 服务商那里，

配置 `_acme-challenge.yourdomain.com` 和`_acme-challenge.www.yourdomain.com`类型为 TXT 的记录。**在没有确认 TXT 记录生效之前不要回车执行。**

```
注意：要输入当时提示的值，本例为 hkh6BT7jERHEDzPORxBQ*************4ebhA
```



**namecheap域名商**

In the advanced DNS panel for your domain in Namecheap we need to create a new `TXT Record` and add `_acme-challenge` as the host and `yB0AXXXXXXORZXTwzeXXXXXXXXXXXXXXXXmOoA1-XXX` as the value:

只需要`_acme-challenge` 不需要后面的域名

建议ttl修改为`1min`，尽快生效。



PS : 在namecheap里TXT 记录要增加2个( **重要**)

- _acme-challenge
- _acme-challenge.www



确认DNS是否生效工具

安装`bind-utils`工具包，`nslookup`和`dig`都可以用了

```shell
#需要安装BIND Utilities工具包
$ sudo yum install bind-utils

$ nslookup -type=TXT _acme-challenge.yourdomain.xyz

[centos@ip-172-26-11-163 ~]$ nslookup -type=TXT _acme-challenge.yourdomain.xyz
Server:         172.x.x.x
Address:        172.x.x.x#53
##没有生效 TXT记录
** server can't find _acme-challenge.yourdomain.xyz: NXDOMAIN
```

```shell
#成功  输出里能看到值yUHptibTcfwPXVqyfTCL4LNtIAaB8Alna45oQddW5Q
$  nslookup -type=TXT _acme-challenge.www.yourdomain.xyz
Server:         172.x.x.x
Address:        172.x.x.x#53

Non-authoritative answer:
_acme-challenge.www.yourdomain.xyz  text = "nyUHptibTcfwPXVqyfTCL4LNtIAaB8Alna45oQddW5Q"

Authoritative answers can be found from:
```

或

```shell
$ dig -t txt _acme-challenge.yourdomain.xyz @8.8.8.8
```



TXT记录一直没有生效。很长时间才生效。。。。。



可以校验一下证书信息

~~~shell
$ sudo openssl x509 -in /etc/letsencrypt/live/yourdomain.xyz/cert.pem -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            03:e3:dc:db:43:ce:12:f6:ce:f4:5c:25:0c:07:b4:74:2b:ba
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=US, O=Let's Encrypt, CN=Let's Encrypt Authority X3
        Validity
            Not Before: Sep 25 15:14:27 2019 GMT
            Not After : Dec 24 15:14:27 2019 GMT
        Subject: CN=yourdomain.xyz
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
:::::::::::省略
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage:
                TLS Web Server Authentication, TLS Web Client Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Subject Key Identifier:
                2A:72:34:41:52:CE:E5:08:98:FC:45:E8:AC:33:D5:48:39:19:CF:41
            X509v3 Authority Key Identifier:
                keyid:A8:4A:6A:63:04:7D:DD:BA:E6:D1:39:B7:A6:45:65:EF:F3:A8:EC:A1

            Authority Information Access:
                OCSP - URI:http://ocsp.int-x3.letsencrypt.org
                CA Issuers - URI:http://cert.int-x3.letsencrypt.org/

            X509v3 Subject Alternative Name:
                DNS:yourdomain.xyz, DNS:www.yourdomain.xyz
:::::::::::省略

~~~

可以看到证书的 SAN 扩展里包含了 `yourdomain.xyz`， 说明申请的证书的匹配范围。



### 证书更新

(有坑，见错误4),  暂时采用重新申请规避。也比较简单。直接重新申请就可以。

> 从证书申请成功的输出信息里可以看到，每三个月就要更新下证书
>
> /etc/letsencrypt/live/yourdomain.xyz/privkey.pem
>    Your cert will expire on `2019-12-24`. To obtain a new or tweaked

~~~

certbot renew
~~~



### 查看证书有效期的命令

```shell
$ sudo openssl x509 -noout -dates -in /etc/letsencrypt/live/yourdomain.xyz/cert.pem
notBefore=Sep 25 15:14:27 2019 GMT
notAfter=Dec 24 15:14:27 2019 GMT

```



### 设置定时任务自动更新证书

letsencrypt证书的有效期是90天，但是可以用脚本去更新。

默认的情况，证书到期前`30天`，执行certbot renew 证书才会更新，每个证书有个配置文件。默认在文件夹 /etc/letsencrypt/renewal 下，有每个证书的信息。

如果时间没到就去执行更新。是不会成功的。letsencrypt会提示你错误消息

~~~
sudo cat /etc/letsencrypt/renewal/yourdomain.xyz.conf
~~~



#### 查看证书到期时间

~~~shell
$  sudo certbot certificates
Saving debug log to /var/log/letsencrypt/letsencrypt.log
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Found the following certs:
  Certificate Name: yourdomain.xyz
    Domains: yourdomain.xyz www.yourdomain.xyz
    Expiry Date: 2020-01-08 07:50:44+00:00 (VALID: 79 days)
    Certificate Path: /etc/letsencrypt/live/yourdomain.xyz/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/yourdomain.xyz/privkey.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
~~~

#### 更新证书 - 非docker方式

```shell
# 更新证书
certbot renew --dry-run 
# 如果不需要返回的信息，可以用静默方式：
certbot renew --quiet
```



#### 更新证书 - docker方式

**注意：更新证书时候网站必须是能访问到的**



##### Crontab

1. 输入以下命令`crontab -e`创建 Cron 文件：

相当于打开一个vi编辑器。把crontab的配置粘贴进来，

保存。

会在`/var/spool/cron`下生成一个root的文件

```
#crontab -e
no crontab for root - using an empty one
#可以先随意输入点内容
#编辑后保存，会生成一个root文件
[root@ip- ~]# cd /var/spool/cron
[root@ip- cron]# ll
total 4
-rw-------. 1 root root 71 Oct 21 10:49 root
```



2. 添加编辑 Certbot 的自动续期命令
   在`/var/spool/cron/root` cron 文件中，复制以下代码，粘贴，保存，上传。

```shell
## 2. docker方式
# 分 时 日 月 周
#推荐 只重启nginx容器 （未实测）
0 3 */70 * 0 docker restart nginx-container > /dev/null
```

以上含义是：每隔 70 天，夜里 3 点整自动执行检查续期命令一次。续期完成后，重启 nginx容器。



3. 重启 Cron 服务，使之生效

```shell
# 2019/10/21 11:05:06 CST 2019 操作 检查时间
service crond restart
```



```shell
# 可以使用crontab定时更新，例如：
# 每月1号5时执行执行一次更新，并重启nginx服务器
00 05 01 * * /usr/bin/certbot renew --quiet && /bin/systemctl restart nginx
```









## NGINX配置SSL

主要参考django配置

目录结构

```
nginx
     my-nginx.conf
     nginx.conf
     Dockerfile
docker-compose.yml
```



docker-compose.yml

**切记** 要把 `- /etc/letsencrypt:/etc/letsencrypt`映射到容器里

~~~
version: '3'
services:
    nginx:
        container_name: nginx-container
        # build命令使用Dockerfile进行镜像构建
        build: ./nginx
        restart: always
        ports:
        - "80:80"
        - "443:443"
        volumes:
        - nginx_data:/nginx_data
        - ./log:/var/log/nginx #容器里的log被映射到宿主机的当前文件的log下面
        - /etc/letsencrypt:/etc/letsencrypt
volumes:
    nginx_data:
~~~



my-nginx.conf

~~~properties
server {
	listen 80;
	server_name  www.yourdomain.xyz;
	# http重定向到https
	#rewrite ^ https://$server_name$request_uri? permanent;
	#感觉上面server-name不固定的方式反而不好。
	rewrite ^(.*)$  https://www.yourdomain.xyz$1 permanent;
}


server {
	listen 443 ssl;
	# index  index.html;
	server_name  www.yourdomain.xyz;
	charset     utf-8;

	client_max_body_size 75M;   # adjust to taste
	
	#也可以再增加URL redirect Record来重定向。推荐nginx里rewrite
	if ($host = 'yourdomain.xyz' ) {
		rewrite ^/(.*)$ https://www.yourdomain.xyz/$1 permanent;
	}
	# 配置ssl相关
	# letsencrypt生成的文件
	##ssl_certificate /etc/letsencrypt/live/yourdomain.xyz/fullchain.pem;
	##ssl_certificate_key /etc/letsencrypt/live/yourdomain.xyz/privkey.pem;

	# 由于这个是链接。映射到容器里不识别。换成archive下的实际文件
	ssl_certificate /etc/letsencrypt/archive/yourdomain.xyz/fullchain1.pem;
	ssl_certificate_key /etc/letsencrypt/archive/yourdomain.xyz/privkey1.pem;

	# 启用 session resumption 提高HTTPS性能
	ssl_session_timeout 1d;
	ssl_session_cache shared:SSL:50m;
	ssl_session_tickets on;

	# DHE密码器的Diffie-Hellman参数, 推荐 2048 位
	##ssl_dhparam /etc/ssl/private/dhparam.pem;

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	# 一般推荐使用的ssl_ciphers值: https://wiki.mozilla.org/Security/Server_Side_TLS
	ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128:AES256:AES:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK';
	ssl_prefer_server_ciphers on;

	# 欢迎页面
	location / {
		root   /usr/share/nginx/html;
		index  index.html index.htm;
		# 不缓存
		add_header Cache-Control no-cache;
		add_header Pragma no-cache;
		add_header Expires 0;
	}
}
~~~

简单说明

ssl_dhparam 这是个非必要的选项，加上后，相对可以安全一点，生成dhparam.pem，需要用到 openssl

方法1

```
openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048
```


方法2

```
openssl dhparam -dsaparam -out /etc/nginx/ssl/dhparam.pem 2048
```


方法2 相对于 方法1 速度快一点，但是安全性较低，/etc/nginx/ssl/dhparam.pem 这个路径可以自定义





nginx.conf

~~~properties
user  root;
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;
events {
    worker_connections  1024;
}
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    #tcp_nopush     on;
    keepalive_timeout  65;
    #gzip  on;
    # include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-available/*;
}
~~~

Dockerfile

~~~properties
FROM nginx:latest
ENV TZ=Asia/Shanghai
COPY nginx.conf /etc/nginx/nginx.conf
COPY my_nginx.conf /etc/nginx/sites-available/

RUN mkdir -p /etc/nginx/sites-enabled/\
    && ln -s /etc/nginx/sites-available/my_nginx.conf /etc/nginx/sites-enabled/

CMD ["nginx", "-g", "daemon off;"]
~~~



访问https://www.yourdomain.xyz显示欢迎页面

点击证书可以看到是安全的。



## 测试ssl证书安全等级

~~~
www.ssllabs.com
最简配置。测试等级为B

加上配置ssl_protocols，ssl_ciphers，ssl_prefer_server_ciphers后
再测试等级就变成A
~~~



## 常用命令

### 1 查看证书

~~~shell
$ sudo certbot certificates

Saving debug log to /var/log/letsencrypt/letsencrypt.log
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Found the following certs:
  Certificate Name: yourdomain.xyz
    Domains: yourdomain.xyz www.yourdomain.xyz
    Expiry Date: 2020-01-08 05:07:23+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/yourdomain.xyz/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/yourdomain.xyz/privkey.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

~~~



### 2 吊销证书

相当于通知 Let's Encrypt，该证书已经无效了。

```shell
$ sudo certbot revoke --cert-path /etc/letsencrypt/live/yourdomain.xyz/cert.pem --reason superseded 
```

-- reason 表示是吊销证书的原因，命令运行成功后，

/etc/letsencrypt/renewal、/etc/letsencrypt/archive、/etc/letsencrypt/live 下对应的文件都会被删除。



如果没过期。重新申请还是老的证书。都不会有提示。直接生成到本地。



## 问题

### 错误1: 不带www域名不支持https

1.配置好以后发现`https://yourdomain.xyz/`不带www的是无法访问的。

带www就可以访问. （nginx.conf里故意注释掉了rewrite）



在namecheap里设置了URL redirect record里设置了"@"为https://www.xxxx.xyz也没用

> @ 记录指没有主机名的域名部份，如 foo.com 



[参考]: https://qizhanming.com/blog/2019/04/23/how-to-install-let-s-encrypt-wildcards-certificate-on-centos-7





### 出错2: Another instance of Certbot is already running

~~~
Another instance of Certbot is already running.
~~~



~~~
$ sudo find / -type f -name ".certbot.lock"
/etc/letsencrypt/.certbot.lock
/var/lib/letsencrypt/.certbot.lock
/var/log/letsencrypt/.certbot.lock
// 删除后重试
$ sudo find / -type f -name ".certbot.lock" -exec rm {} \;
~~~



### 错误3: 启动报错

启动报找不到证书文件。是因为没有把证书映射到docker容器里导致。

```
django-container | #######2==== .
nginx-container | nginx: [emerg] cannot load certificate "/etc/letsencrypt/archive/yourdomain.xyz/fullchain1.pem": BIO_new_file() failed (SSL: error:02001002:system library:fopen:No such file or directory:fopen('/etc/letsencrypt/archive/yourdomain.xyz/fullchain1.pem','r') error:2006D080:BIO routines:BIO_new_file:no such file)
nginx-container exited with code 1
```



### 错误4: 手动续期报错



关键错误信息



```
The error was: PluginError('An authentication script must be provided with --manual-auth-hook when using the manual plugin non-interactively.',).
```



以下为实际错误输出

> [root@ip-172-26-7-109 cron]# certbot renew --dry-run
> Saving debug log to /var/log/letsencrypt/letsencrypt.log
>
> - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
> Processing /etc/letsencrypt/renewal/yourdomain.xyz.conf
> - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
> Cert is due for renewal, auto-renewing...
> Could not choose appropriate plugin: The manual plugin is not working; there may be problems with your existing configuration.
> The error was: PluginError('An authentication script must be provided with --manual-auth-hook when using the manual plugin non-interactively.',)
> Attempting to renew cert (yourdomain.xyz) from /etc/letsencrypt/renewal/yourdomain.xyz.conf produced an unexpected error: The manual plugin is not working; there may be problems with your existing configuration.
> The error was: PluginError('An authentication script must be provided with --manual-auth-hook when using the manual plugin non-interactively.',). Skipping.
> All renewal attempts failed. The following certs could not be renewed:
>   /etc/letsencrypt/live/yourdomain.xyz/fullchain.pem (failure)



> 背景知识

使用 certbot 工具，为不能自动给 letencrypt 通配符证书自动续期（renew）而烦恼吗？这个工具能够帮忙！

不管是申请还是续期，只要是通配符证书，只能采用 dns-01 的方式校验申请者的域名，也就是说 certbot 操作者必须手动添加 DNS TXT 记录。

如果你编写一个 Cron (比如 1 1 */1 * * root certbot-auto renew)，自动 renew 通配符证书，此时 Cron 无法自动添加 TXT 记录，这样 renew 操作就会失败，如何解决？

certbot 提供了一个 hook，可以编写一个 Shell 脚本，让脚本调用 DNS 服务商的 API 接口，动态添加 TXT 记录，这样就无需人工干预了。

在 certbot 官方提供的插件和 hook 例子中，都没有针对国内 DNS 服务器的样例，所以我编写了这样一个工具，目前支持阿里云 DNS、腾讯云 DNS、GoDaddy（certbot 官方没有对应的插件）。



***namecheap也是不支持的。***

暂定手动申请。



**切记** 申请前先吊销掉老的证书，否则即使申请了新证书，使用的还是老的过期的

~~~shell
$ sudo certbot revoke --cert-path /etc/letsencrypt/live/yourdomain.xyz/cert.pem --reason superseded 
~~~



申请新证书

~~~shell
 sudo certbot certonly -d yourdomain.xyz -d www.yourdomain.xyz --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
~~~



按下回车前先确认dns是否生效。以下为生效ok的输出

~~~
# nslookup -type=TXT _acme-challenge.yourdomain.xyz
Server:         172.x.x.x
Address:        172.x.x.x#53

Non-authoritative answer:
_acme-challenge.yourdomain.xyz  text = "uQ0w86kWKq5EmdKRWRK0jtwYasTydzDsr9GHZ4J550Q"

Authoritative answers can be found from:

# nslookup -type=TXT _acme-challenge.www.yourdomain.xyz
Server:         172.x.x.x
Address:        172.x.x.x#53

Non-authoritative answer:
_acme-challenge.www.yourdomain.xyz      text = "sixnxEeTJtU6Q9umrNq3JG3piGPS2weHIRsEGDX686I"

Authoritative answers can be found from:

~~~



确认证书已经更新到2020-04-15

~~~
# certbot certificates
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Found the following certs:
  Certificate Name: yourdomain.xyz
    Domains: yourdomain.xyz www.yourdomain.xyz
    Expiry Date: 2020-04-15 01:48:52+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/yourdomain.xyz/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/yourdomain.xyz/privkey.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
~~~



最后一步，

记得重启nginx（ 重启nginx容器即可`docker restart nginx-container`），生效。



《新坑》

一切就绪后，发现浏览器打开网站还是老的证书信息，清除浏览器缓存也没有用。

解决方法

要先把老的证书吊销掉。`certbot revoke` 



### 错误5  证书过期后，重新生效后浏览器打开还是老证书。



要先把老的证书吊销掉， 相当于通知letsencypt改证书已无效。

~~~
$ sudo certbot revoke --cert-path /etc/letsencrypt/live/yourdomain.xyz/cert.pem --reason superseded 
~~~



会清空live下的所有信息，

然后再重新申请证书。这一步申请的话由于前面申请过了。中间的询问步骤都是没有了的。就是直接吧上次申请有效的证书又给你生成到本地

