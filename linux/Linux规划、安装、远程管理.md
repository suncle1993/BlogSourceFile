---
categories:
  - linux
date: '2015-12-27T23:43:53'
description: ''
id: 162
tags:
  - CentOS
  - Linux
  - 安装
  - 远程管理
title: Linux规划、安装、远程管理
---



## Linux规划

一定要根据服务项目，才来进行硬盘的规划。

如果是邮件主机，一般/var通常会给个数GB的大小， 如此一来就可以不担心会邮件空间不足！如果是多用户多终端主机,一般/home通常比较大。这些都是和当初预计的主机服务是有关的！

<!--more-->

我的wmware中的CentOS分区如下图(使用df -th可以查看)：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Flowsnow%E6%9F%A5%E7%9C%8BLinux%E7%A3%81%E7%9B%98%E5%88%86%E5%8C%BA.jpg)

/boot 该分区存放Linux的Grub(bootloader)和内核源码,一般给200MB即可> 

swap 该分区没有对应的目录，故用户无法访问，只能由系统访问。Linux下的swap分区即为虚拟内存.虚拟内存用于当系统内存空间不足时，先将临时数据存放在swap分区，等待一段时间后，然后再将数据调入到内存中执行.所以说，虚拟内存只是暂时存放数据，在该空间内并没有执行。swap分区大小一般为物理内存的两倍，物理内存最多给4G足够了，我的是2000MB> 

/home 根据用户分配给即可，我的是2000MB> 

/ 根分区，剩余空间都给/分区

## Linux安装

wmware虚拟机安装Linux操作系统参见我的csdn博客:

[https://blog.csdn.net/u013637931/article/details/49288073](https://blog.csdn.net/u013637931/article/details/49288073)

## Linux远程管理

Linux能够远程管理的前提:

1. 配置静态IP（如果是动态则每次都要配置）
2. 主机开启登陆服务（ssh，telnet等）
3. 登陆工具使用和配置

### 配置静态IP

如何配置静态IP参见我的csdn博客:

[https://blog.csdn.net/u013637931/article/details/49287643](https://blog.csdn.net/u013637931/article/details/49287643)

### 检查主机是否开启登陆服务

#### telnet  (明文传输)

系统默认是没有安装telnet服务的，因此我们先安装telenet-server和telnet服务

* yum install telnet-server #服务端

* yum install telnet #客户端

  查看telnet版本

* rpm -qa | grep telnet

  检查是否开启telnet服务

* rpm -qa telnet

  启动telnet 服务

* vi /etc/xinetd.d/telnet

  > 找到 disable = yes 将 yes 改成 no 。

  启动或者重启telnet服务的守护进程xinetd

* service xinetd start

* service xinetd restart

  修改防火墙，允许端口23通过

* vim /etc/sysconfig/iptables

  > 添加如下一行内容：> 
  >
  > A INPUT -m state --state NEW -m tcp -p tcp --dport 23 -j ACCEPT
  > ​
* service iptables restart #重启防火墙

  本地测试telnet(也可以putty远程测试)

* telnet localhost

  参考：[Centos 开启telnet-service服务](https://www.cnblogs.com/xlmeng1988/archive/2012/04/24/telnet-server.html)

#### ssh  (加密传输)

系统默认安装了ssh

检查CentOS是否安装ssh

* rpm -qa | grep ssh

  检查是否启动ssh服务

* ps -ef | grep ssh

  若出现/usr/bin/sshd进程，则表示启动成功

启动或重启SSH服务

* service sshd start

* service sshd restart

  使用putty或者secureCRT远程ssh登录即可测试

### 远程登录的工具

putty，secureCRT，xmanager都可以登录CentOS系统操作，个人偏爱于putty和secureCRT。

WinSCP可以进行windows和linux之间进行文件传输

各个安装包下载链接：

WinSCP链接：[https://pan.baidu.com/s/1brodEu](https://pan.baidu.com/s/1brodEu) 密码: ar59

secureCRT链接：[https://pan.baidu.com/s/1hrnEbn2](https://pan.baidu.com/s/1hrnEbn2) 密码: 2qgi

putty链接:[https://pan.baidu.com/s/1o6XSKSa](https://pan.baidu.com/s/1o6XSKSa) 密码: 47na