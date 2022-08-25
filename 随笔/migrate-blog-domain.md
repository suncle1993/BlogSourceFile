---
categories:
  - 随笔
date: '2019-03-31T01:00:56'
description: ''
tags:
  - Hexo
  - Nginx
title: 博客域名迁移
---




突然就想给本站换个域名了，那么就动手了

目标：

1. 网站使用Git Hooks自动部署到VPS，
2. nginx解析域名到VPS，并开启https和http2
3. 老域名 `flowsnow.net` 301永久重定向到新域名 `suncle.me`
4. 更改Google收录和Baidu收录索引，尽可能少的影响权重
5. 使用valine评论系统，迁移disqus评论数据到valine

<!--more-->

## 网站自动部署到VPS

首先需要建立好本地到VPS的ssh链接，开启互信

在VPS上建立git裸库

```bash
cd ~
mkdir blog.git    # 创建git仓库文件夹
cd blog.git       # 进入仓库目录
git init --bare   # 使用--bare参数初始化为裸仓库，这样创建的仓库不包含工作区
```

配置Git Hooks，创建post-receive文件：

```bash
cd ~/blog.git/hooks  # 切换到hooks目录下
vim post-receive     # 创建文件
```

写入一下内容实现基于复制的自动部署

```
#!/bin/bash
GIT_REPO=/root/blog.git
TMP_GIT_CLONE=/tmp/blog
PUBLIC_WWW=/data/www
rm -rf ${TMP_GIT_CLONE}
git clone $GIT_REPO $TMP_GIT_CLONE
rm -rf ${PUBLIC_WWW}/*
cp -rf ${TMP_GIT_CLONE}/* ${PUBLIC_WWW}
```

保存退出后，执行：`chmod +x post-receive` 赋予可执行权限。

在hexo博客站点配置文件_config.yml文件中，修改部署配置：

```yaml
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  - type: git
    repo:
      # github: git@github.com:Flowsnow/Flowsnow.github.io.git,master
      # coding: git@git.coding.net:Flowsnow/Flowsnow.git,coding-pages
      linode: root@139.162.108.44:blog.git
```

## nginx解析域名到VPS

在VPS上使用 `apt-get install -y nginx` 安装好 `nginx` 之后，新增配置 `/etc/nginx/sites-available/suncle.me`：

```nginx
server {
    server_name suncle.me;
    root /data/www;
    index index.html;

    # Media: images, icons, video, audio, HTC
    location ~* \.(?:jpg|jpeg|gif|png|ico|cur|gz|svg|mp4|ogg|ogv|webm|htc)$ {
        access_log off;
        add_header Cache-Control "max-age=2592000";
    }

    # CSS and Javascript
    location ~* \.(?:css|js)$ {
        add_header Cache-Control "max-age=31536000";
        access_log off;
    }


    listen 443 ssl http2; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/suncle.me/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/suncle.me/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = suncle.me) {
        return 301 https://suncle.me$request_uri;
    } # managed by Certbot: redirect http to https

    listen 80;
    server_name suncle.me;
    return 404; # managed by Certbot
}
server {
    listen 80;
    server_name www.suncle.me;
    return 301 https://suncle.me$request_uri;  # redirect www to non-www
}
```

创建软连接

```
ln -s /etc/nginx/sites-available/suncle.me /etc/nginx/sites-enabled/
```

切换用户为 `root`，开启 `gzip_static` 压缩，关闭日志，修改 `nginx.conf` 如下：

```nginx
user root;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections 768;
    # multi_accept on;
}

http {

    # Basic Settings

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    # server_tokens off;

    # server_names_hash_bucket_size 64;
    # server_name_in_redirect off;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # SSL Settings

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
    ssl_prefer_server_ciphers on;

    # Logging Settings

    # access_log /var/log/nginx/access.log;
    # error_log /var/log/nginx/error.log;

    # Gzip Settings

    gzip_static on;
    gzip on;
    gzip_disable "msie6";

    # gzip_vary on;
    # gzip_proxied any;
    # gzip_comp_level 6;
    # gzip_buffers 16 8k;
    # gzip_http_version 1.1;
    # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    # Virtual Host Configs

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

然后使用一下命令启动/重启 `nginx` ：

```bash
systemctl start nginx
systemctl restart nginx
```

配置https和http2，先安装certbot

```bash
apt-get install software-properties-common
add-apt-repository ppa:certbot/certbot
apt-get update
apt-get install python-certbot-nginx
```

运行certbot

```bash
certbot --nginx
```

设置好之后重启nginx即可。

设置定时更新Https证书，在crontab中加入以下定时任务，每天凌晨3点执行

```
0 3 * * * certbot renew
```

## Nginx 实现永久重定向

使用nginx将老域名永久重定向到新域名，site-enabled目录下新增以下配置

```
server {
    listen 80 default_server;
    server_name flowsnow.net;
    index index.html index.htm index.php;
    root /home/wwwroot/default;

    if ( $scheme = "http" ) {
        return 301 https://suncle.me$request_uri;
    }
    location / {
        rewrite ^(.*)$  https://suncle.me$1 permanent;
    }
}
```

## 迁移Google和Baidu收录索引

Baidu收录可以直接使用Baidu网站改版工具实现，Google收录可以使用Google更改网站地址工具实现，但是前提是需要配置好301重定向。Google提交更换地址申请页面如下：
![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/image/tech/migrate-blog-domain/google-replace-domain.jpg)

## 迁移Disqus到Valine

觉得valine评论系统看着还不错，而且disqus由于被墙之前一直是加了一个disqus-proxy的反向代理才能使访客看到评论，比较麻烦而且需要一台VPS。

Valine基于LeanCloud，不得不说，LeanCloud的服务一直都是比较稳定的，而且有免费额度可以使用。

迁移Disqus评论数据到Valine可以使用Disqus2LeanCloud这个工具，见下图，具体使用可以参考后面的参考链接

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/image/tech/migrate-blog-domain/Disqus2LeadCloud.jpg)

---

参考：

1. [知乎-网站更换域名收录怎么办？](https://www.zhihu.com/question/270176706) 
2. [How to Host a Static Website with Nginx – Jason Rigden – Medium](https://medium.com/@jasonrigden/how-to-host-a-static-website-with-nginx-8b2dd0c5b301)
3. [Hexo建站使用Git部署到VPS](https://zhuanlan.zhihu.com/p/37312476) 
4. [Nginx 在线美化器](http://www.html580.com/tool/nginx/index.php)
5. [Google 网站改版](https://www.google.com/webmasters/tools/change-address)
6. [博客更换域名后利用Nginx实现完美301跳转 - 运维学习笔记博客](https://www.imydl.tech/lnmp/315.html)
7. [使用valine评论系统](https://ioliu.cn/2017/add-valine-comments-to-your-blog/)
8. [disqus评论迁移到valine](http://disqus.panjunwen.com/)

