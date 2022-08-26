---
abbrlink: 4012305162
alias: 2017/03/30/CentOS7-Nginx配置Lets-Encrypt-SSL证书/index.html
categories:
- linux
date: '2017-03-30T15:49:46'
description: ''
tags:
- CentOS
- Nginx
- Let's-Encrypt
- SSL
title: CentOS7-Nginx配置Let's-Encrypt-SSL证书
---








# Let's-Encrypt

为http站点添加https支持，需要从证书发行机构获取SSL/TLS 证书。常见的免费证书有两种：

- [Let's-Encrypt](https://letsencrypt.org/)，本文即将介绍，Let's-Encrypt大法好。
- [caddy](https://caddyserver.com/)，原生支持 HTTP/2，自动创建 [Let’s Encrypt](http://www.appinn.com/use-letsencrypt-with-nginx/) 证书，非常简单易用。

## 安装

```shell
yum install epel-release -y
yum install certbot -y
```

## 配置

```shell
certbot certonly --webroot -w /www/html -d suncle.me -d www.suncle.me
```

- `--webroot`表示以webroot模式运行，此处我们不选择standalone模式
- `-w /www/html`表示站点根目录在/www/html
- `-d suncle.me -d www.suncle.me`表示为@主机和www主机生成共同的证书

按照提示操作成功后，提示：

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at
   /etc/letsencrypt/live/suncle.me/fullchain.pem. Your cert will
   expire on 2017-06-28. To obtain a new or tweaked version of this
   certificate in the future, simply run certbot again. To
   non-interactively renew *all* of your certificates, run "certbot
   renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```

<!--more-->

## 证书自动更新

Let’s Encrypt 只有3个月的有效期，所以我们需要定时去更新证书。

可以通过运行：`certbot renew --dry-run` 来测试自动生成是否能够正常运行。如果正确执行，我们就可以通过以下命令更新证书:

```shell
certbot renew 
```

如果要达到自动更新证书，可以借助`crontab`或者`systemd`定时执行上面的更新命令。Let’s Encrypt官方建议一天更新两次最好。因为证书没有到期之前是不会更新，因此即使一天执行两次也不会有什么影响。

```shell
# 每天3:00和19:00点更新证书
0 3,19 * * * certbot renew
```

具体执行时间可以参考以下crontab格式进行修改：

```
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```

# 配置Nginx SSL证书

修改/usr/local/nginx/nginx.conf文件如下（最好先备份）

```nginx
# Nginx以root用户启动
user root;
# Nginx开启的进程数
worker_processes 2;

events {
    # Nginx连接的最大个数
    worker_connections  65535;
}

http {
    # 如果路径中出现通配符，mime.types可配置多个文件
    include mime.types; 
    # 默认文件类型
    default_type application/octet-stream;
    # 日志格式  
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    # 记录访问日志
    access_log logs/access.log main;
    # 开启sendfile，提升文件传输效率
    sendfile on;
    # 死链判断：客户端连接保持活动的超时时间
    keepalive_timeout 65;

    #设置非安全连接永久跳转到安全连接
    server{
        listen 80;
        server_name www.suncle.me suncle.net blog.suncle.net;
        #告诉浏览器有效期内只准用 https 访问
        add_header Strict-Transport-Security max-age=15768000;
        #永久重定向到 https 站点
        return 301 https://$server_name$request_uri;
    }
    server {
        #启用 https, 使用 http/2 协议, nginx 1.9.11 启用 http/2 会有bug, 已在 1.9.12 版本中修复.
        listen 443 ssl http2;
        server_name www.suncle.me suncle.net blog.suncle.net;
        #告诉浏览器当前页面禁止被frame
        add_header X-Frame-Options DENY;
        #告诉浏览器不要猜测mime类型
        add_header X-Content-Type-Options nosniff;
        root /www/html;

        #证书路径
        ssl_certificate /etc/letsencrypt/live/suncle.me/fullchain.pem;
        #私钥路径
        ssl_certificate_key /etc/letsencrypt/live/suncle.me/privkey.pem;
        #安全链接可选的加密协议
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        #可选的加密算法,顺序很重要,越靠前的优先级越高.
        ssl_ciphers ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-RC4-SHA:!ECDHE-RSA-RC4-SHA:ECDH-ECDSA-RC4-SHA:ECDH-RSA-RC4-SHA:ECDHE-RSA-AES256-SHA:HIGH:!RC4-SHA:!aNULL:!eNULL:!LOW:!3DES:!MD5:!EXP:!CBC:!EDH:!kEDH:!PSK:!SRP:!kECDH;
        #在 SSLv3 或 TLSv1 握手过程一般使用客户端的首选算法,如果启用下面的配置,则会使用服务器端的首选算法.
        ssl_prefer_server_ciphers on;
        #储存SSL会话的缓存类型和大小
        ssl_session_cache shared:SSL:10m;
        #缓存有效期
        ssl_session_timeout 60m;
    }
}
```

以上配置文件nginx.conf中需要修改的字段主要有：

- server_name www.suncle.me suncle.net blog.suncle.net;
- ssl_certificate /etc/letsencrypt/live/suncle.me/fullchain.pem;
- ssl_certificate_key /etc/letsencrypt/live/suncle.me/privkey.pem;

> `listen 443 ssl http2;`这一句中，如果Nginx编译安装的时候没有安装`ngx_http_v2_module`模块，则需要先安装。或者不采用http2协议，直接`listen 443 ssl`即可

保存配置，检查是否报错，重启Nginx

```shell
/usr/local/nginx/nginx -t
/usr/local/nginx/nginx -s reload
```

Nginx 的 SSL 证书到此配置完成。

---

参考

1. [CentOS 7 Nginx Let’ s Encrypt SSL 证书安装配置](https://blog.itnmg.net/letsencrypt-ssl/)
2. [开启Https之旅](http://blog.lzuer.net/2016/10/25/https/)
3. [nginx+https+http2搭建(二)](https://www.arayzou.com/2016/08/12/nginx+https+http2%E6%90%AD%E5%BB%BA(%E4%BA%8C)/)
4. [Linux Crontab使用总结](http://xianglong.me/article/linux-crontab/)