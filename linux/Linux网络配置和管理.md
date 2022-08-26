---
abbrlink: 506153548
alias: 2016/01/22/Linux网络配置和管理/index.html
categories:
- linux
date: '2016-01-22T19:59:15'
description: ''
tags:
- Linux
- 网络
- CentOS
title: Linux网络配置和管理
---








## 网络设备信息查看和配置

### 文件配置

在CentOS中，系统网络设备的配置文件保存在“/etc/sysconfig/network-scripts”目录下，ifcfg-eth0包含第一块网卡的配置信息，ifcfg-eth1包含第二块网卡的配置信息。

``` 
vi /etc/sysconfig/network-scripts/ifcfg-eth0

```

可以得到配置信息

``` 
DEVICE=eth0
HWADDR=00:0C:29:35:50:5B
TYPE=Ethernet
UUID=e9e3400b-1e20-49c1-a12c-de3b6b221048
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=static
IPADDR=192.168.118.10
NETMASK=255.255.255.0
NETWORK=192.168.118.0
BROADCAST=192.168.118.255
```

若希望手工修改网络地址，可以通过修改对应的文件（ifcfg-ethN）或创建新的文件来实现。

具体对应修改方法参见我的csdn博客：[CentOS6.7配置静态IP](https://blog.csdn.net/u013637931/article/details/49287643)

<!--more-->

每次修改网卡之后需要重启对应网卡才能生效，重启命令如下：

``` 
service network restart
```

命令配置

ifconfig，参见[ifconfig命令](https://man.linuxde.net/ifconfig)

ifconfig命令被用于配置和显示Linux内核中网络接口的网络参数。用ifconfig命令配置的网卡信息，在网卡重启后或机器重启后，配置就不存在。

``` 
[root@localhost ~]# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0C:29:35:50:5B
          inet addr:192.168.118.10  Bcast:192.168.118.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fe35:505b/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:433 errors:0 dropped:0 overruns:0 frame:0
          TX packets:214 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:41843 (40.8 KiB)  TX bytes:22969 (22.4 KiB)
          Interrupt:19 Base address:0x2000

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:14 errors:0 dropped:0 overruns:0 frame:0
          TX packets:14 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1410 (1.3 KiB)  TX bytes:1410 (1.3 KiB)

```

配置ip地址：

``` 
[root@localhost ~]# ifconfig eth0 192.168.2.10 
[root@localhost ~]# ifconfig eth0 192.168.2.10 netmask 255.255.255.0 
[root@localhost ~]# ifconfig eth0 192.168.2.10 netmask 255.255.255.0 broadcast 192.168.2.255
```



启动关闭指定网卡：

``` 
ifconfig eth0 up
ifconfig eth0 down
```

ssh登陆linux服务器操作要小心，关闭了就不能开启了，除非有多网卡（即有多ip）。



配置虚拟ip：

``` 
ifconfig eth0:1 192.168.118.2
```

如果要用就配置eth0:1需要在/etc/sysoconfig/network-scripts/下面创建一个eth0:1文件，并且设定ip为192.168.118.2，设定device名称为eth0:1，然后重启网卡，就可以永久配置了。



## 网络配置信息

相关的配置文件是/etc/sysconfig/network

``` 
vi /etc/sysconfig/network
```

得到如下结果：

``` 
NETWORKING=yes
NETWORKING_IPV6=no
HOSTNAME=localhost.localdomain
GATEWAY=192.168.118.1
```

解释：

> NETWORKING=yes 网络是否可用。



> HOSTNAME=xxxxxxxx为新设置的主机名。



## 域名解析配置/etc/resolv.conf

/etc/resolv.conf文件是由域名解析器（resolver，一个根据主机名解析ip地址的库）使用的配置文件

``` 
[root@localhost ~]# vi /etc/resolv.conf
nameserver 114.114.114.114
nameserver 8.8.8.8

```

“nameserver”表示解析域名时使用该地址制定的主机为域名服务器。



## 主机名配置/etc/hosts

/etc/hosts, 包括主机名的用途、配置文件的操作方法等。用来把主机名字映射到IP地址

需要修改主机名时可用hostname命令修改，如下所示：

``` 
hostname flowsnow #将主机名称暂时改为flowsnow
```



## 网络服务配置/etc/services

/etc/services文件是记录网络服务名和它们对应使用的端口号及协议。端口号和标准服务之间的对应关系在RFC 1700中有详细的定义。这个文件使得程序能够把服务的名字转换成端口号，这张表在每一台计算机上都存在。只有root用户才能修改这个文件。而且通常情况下这个文件不需要修改。

``` 
vi /etc/services
...
mysql           3306/tcp                        # MySQL
mysql           3306/udp                        # MySQL
...
```

## 主机查找方法配置

指定主机名查找方法，通常指先查找文件/etc/hosts,找不到时再向DNS服务器请求。对于大多数用户不用改动此文件内容。



## netstat命令

参见：[netstat命令](https://man.linuxde.net/netstat) [netstat命令详解](https://www.cnblogs.com/ggjucheng/archive/2012/01/08/2316661.html)

netstat命令用来打印Linux中网络系统的状态信息。

常见选项：

``` 
-a (all)显示所有选项，默认不显示LISTEN相关
-t (tcp)仅显示tcp相关选项
-u (udp)仅显示udp相关选项
-n 拒绝显示别名，能显示数字的全部转化成数字。
-l 仅列出有在 Listen (监听) 的服務状态

-p 显示建立相关链接的程序名
-r 显示路由信息，路由表
-e 显示扩展信息，例如uid等
-s 按各个协议进行统计
-c 每隔一个固定时间，执行该netstat命令。

注：LISTEN和LISTENING的状态只有用-a或者-l才能看到
```