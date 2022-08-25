---
categories:
  - linux
date: '2017-03-30T14:29:45'
description: ''
tags:
  - CentOS
  - Nginx
  - 安装
title: CentOS7-Nginx编译安装
---



# 安装编译环境

需先安装好编译环境make，gcc和g++ 开发库

```shell
yum -y install gcc automake autoconf libtool make
yum install gcc gcc-c++
```

# 安装pcre

pcre(Perl Compatible Regular Expressions)： perl 兼容的正则表达式库。

以下各编译安装的源码包均放在`/usr/local/src`下，Nginx依赖pcre是为了重写rewrite。

从ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/下载pcre包，不宜太新，推荐使用pcre8.39或8.40，太新的pcre版本Nginx不支持。

```shell
cd /usr/local/src
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.39.tar.gz
tar -zxvf pcre-8.39.tar.gz
cd pcre-8.39
./configure
make
make install
```

<!--more-->

# 安装zlib

zlib是为了Nginx压缩

从http://zlib.net/出下载当前最新源码http://zlib.net/zlib-1.2.11.tar.gz

```shell
cd /usr/local/src
wget http://zlib.net/zlib-1.2.11.tar.gz
tar -zxvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure
make
make install
```

# 安装ssl

```shell
cd /usr/local/src
wget https://www.openssl.org/source/openssl-1.0.1t.tar.gz
tar -zxvf openssl-1.0.1t.tar.gz
cd openssl-1.0.1t
./config 	# 不是./Configure
make
make install
```

# 安装Nginx

Nginx 一般有两个版本，分别是稳定版和开发版，您可以根据您的目的来选择这两个版本的其中一个，下面是把 Nginx 安装到 /usr/local/nginx 目录下的详细步骤：

```shell
cd /usr/local/src
wget https://nginx.org/download/nginx-1.10.2.tar.gz
tar -zxvf nginx-1.10.2.tar.gz
cd nginx-1.10.2

./configure --sbin-path=/usr/local/nginx/nginx --conf-path=/usr/local/nginx/nginx.conf --pid-path=/usr/local/nginx/nginx.pid --with-http_ssl_module --with-pcre=/usr/local/src/pcre-8.39 --with-zlib=/usr/local/src/zlib-1.2.11 --with-openssl=/usr/local/src/openssl-1.0.1t --with-http_v2_module

make
make install
```

- `--with-pcre=/usr/local/src/pcre-8.39` 指的是pcre-8.39 的源码路径。


- `--with-zlib=/usr/local/src/zlib-1.2.11` 指的是zlib-1.2.11 的源码路径。
- `--with-openssl=/usr/local/src/openssl-1.0.1t`指的是openssl-1.0.1t 的源码路径。


- 启用 https,时如需使用 http/2 协议，则会依赖`ngx_http_v2_module`模块，可以使用`--with-http_v2_module`配置参数来启用。

# 启动Nginx

测试nginx.conf是否正确

```shell
/usr/local/nginx/nginx -t
```

确保系统的 80 端口没被其他程序占用，运行以下命令来启动 Nginx

```shell
/usr/local/nginx/nginx 
```

打开浏览器访问此机器的 IP，如果浏览器出现 Welcome to nginx! 则表示 Nginx 已经安装并运行成功。



---

参考

1. [Nginx安装](http://www.nginx.cn/install)