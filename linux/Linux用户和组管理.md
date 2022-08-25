---
categories:
  - linux
date: '2016-01-09T00:16:04'
description: ''
id: 174
tags:
  - CentOS
  - Linux
  - 命令
  - 总结
  - 用户
  - 管理
  - 组
  - 配置
title: Linux用户和组管理
---



* * *

## 概述

只有root用户才能管理用户和组，所以一切命令都是root执行。

### 用户分类

1. 超级用户：root，UID=0
   
2. 普通用户：具有操作系统有限的权限，500&lt;=UID&lt;=65535(2^32-1),有限个
   
3. 伪用户：为了安全,1&lt;=UID&lt;=499
   
   > 伪用户解释：linux中任何一个命令的操作都必须有一个用户的身份。伪用户一般和系统或者程序服务相关，比如bin,daemon,shutdown ,halt等，linux默认都有这些伪用户，伪用户通常不需要或无法登陆系统（nologin），可以没有宿主目录

<!--more-->

### 用户和组的配置文件

#### 用户信息文件：/etc/passwd

* vim /etc/passwd
  
  ``` 
  root:x:0:0:root:/root:/bin/bash
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  adm:x:3:4:adm:/var/adm:/sbin/nologin
  linzhiling:x:500:500::/home/linzhiling:/bin/bash
  lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
  ```
  
  在该文件中，每一行用户记录的各个数据段用“：”分隔，分别定义了用户的各方面属性。各个字段的顺序和含义如下：

**LOGNAME**:注册名，用于区分不同的用户

**PASSWORD**:口令，系统用口令来验证用户的合法性口令不再直接保存在passwd文件中，通常将passwd文件中的口令字段使用一个“x”来代替，将/etc /shadow作为真正的口令文件，用于保存包括个人口令在内的数据。当然shadow文件是不能被普通用户读取的，只有超级用户才有权读取。

**UID**:Linux系统中惟一的用户标识，用于区别不同的用户。在系统内部管理进程和文件保护时使用 UID字段。在Linux系统中，注册名和UID都可以用于标识用户，只不过对于系统来说UID更为重要；而对于用户来说注册名使用起来更方便。在某些特 定目的下，系统中可以存在多个拥有不同注册名、但UID相同的用户，事实上，这些使用不同注册名的用户实际上是同一个用户。

**GID**:用户组id

**USERINFO**:用户名，包含有关用户的一些信息，如用户的真实姓名、办公室地址、联系电话等。

**HOME**:用户主目录，home_directory

**SHELL**:命令解释程序(Shell),Shell是当用户登录系统时运行的程序名称，通常是一个Shell程序的全路径名，比如/bin/bash

参考网址：[http://luzl.iteye.com/blog/564404](http://luzl.iteye.com/blog/564404)

#### 用户密码文件：/etc/shadow

* vim /etc/shadow
  
  在该文件中每一行密码记录的各个数据段也是用‘:’分隔，一共有九位， 这九个位的含义参考网址：
  
  [https://www.cnblogs.com/zhousir1991/archive/2011/07/25/2116520.html](https://www.cnblogs.com/zhousir1991/archive/2011/07/25/2116520.html)

#### 组配置文件

* vim /etc/group
  
  字段解释参考如下网址：
  
  [https://www.cnblogs.com/peida/archive/2012/12/05/2802419.html](https://www.cnblogs.com/peida/archive/2012/12/05/2802419.html)

## Linux用户管理命令

### 增加用户

* useradd liuyifei
* passwd 123456 #创建一个用户之后必须设置一个密码才能登陆，也是用来修改密码的

### 删除用户

* userdel liuyifei
  
  如果只是暂时停用用户，可以用root用户直接在/etc/passwd文件中在该用户前面加上注释符就行，再次启用时去掉就行。

### 查看用户

* w #查看当前登录的所有用户
* who #查看当前登录的所有用户
* whoami #查看当前登录用户名
* finger liuyifei #查看单个用户信息
* yum install finger #finger命令需要自行安装

## Linux组管理命令

### 创建组

* groupadd mingxing

### 修改组

* groupmod [-g gid [-o]] [-n group_name] group
* groupmod -n mingxing1 mingxing #将mingxing组的名称改为mingxing1

### 删除组

* groupdel mingxing

### 查看组

* cat /etc/group #查看所有组
* groups liuyifei #查看用户所在的组

### gpasswd将用户从组中添加删除

参考：[https://man.linuxde.net/gpasswd](https://man.linuxde.net/gpasswd)

* gpasswd -a linzhiling mingxing #-a=add 添加
* gpasswd -d linzhiling mingxing #-d=del 删除
* gpasswd mingxing #为mingxing组修改密码

## 附

id命令：显示用户的uid和gid

- id liuyifei