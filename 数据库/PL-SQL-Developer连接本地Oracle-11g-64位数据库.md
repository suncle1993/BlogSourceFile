---
categories:
  - 数据库
date: '2016-03-22T11:02:40'
description: ''
tags:
  - Oracle
  - pl/sql
  - 数据库
title: PL/SQL Developer连接本地Oracle 11g 64位数据库
---


## 登陆PL/SQL

假定本地电脑中已经安装了Oracle 11gR2数据库和PL/SQL developer。

如果没有安装可以在一下地址下载安装：

Oracle 11gR2数据库：[https://www.oracle.com/technetwork/database/enterprise-edition/downloads/112010-win64soft-094461.html](https://www.oracle.com/technetwork/database/enterprise-edition/downloads/112010-win64soft-094461.html)

PL/SQL developer(含注册机)：[https://pan.baidu.com/s/1kUfY8GB](https://pan.baidu.com/s/1kUfY8GB) 密码: 1ky8

首先打开PL/SQL，会发现没有database可以选择，我们可以以非登录方式登陆PL/SQL（直接点cancel即可）。

开始设置：Tools->Preferences，进入后点击Oracle下面的connection，设置Oracle home和oci library。

由于我已经设置过了，如下图：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-PLSQL%E6%9C%AC%E5%9C%B0%E9%85%8D%E7%BD%AE.png)

图中路径可能不一样，具体的看你的Oracle Home目录，如果不知道自己的Oracle Home目录的，可以去自己的环境变量中看一下。

点击Apply->OK，退出PL/SQL Developer，再次登录。

<!--more-->

尝试登陆数据库。出现以下错误：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-PLSQL%E9%94%99%E8%AF%AF%E5%8E%9F%E5%9B%A0.png)

显示初始化错误：Make sure you have the 32 bits Oracle Client installed.

这个意思就是我们没有安装32位的Oracle客户端。虽然安装的是64位的Oracle，但是我们plsql不能识别，只能识别32的客户端。那么我们就去下载安装32位的客户端。

## 安装32 bits Oracle Client

首先确定下自己电脑上装的Oracle11g的具体版本：

```
C:\Users\clg>sqlplus / as sysdba

SQL*Plus: Release 11.2.0.1.0 Production on 星期二 3月 22 10:40:19 2016

Copyright (c) 1982, 2010, Oracle.  All rights reserved.


连接到:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
```

可以看到版本是11.2.0.1.0，那么就需要下载对应版本的客户端，不过应该是下载11.2的应该都可以。保险起见，我们还是下载11.2.0.1.0版本的。

32位的Oracle client下载地址：

官网：[https://www.oracle.com/technetwork/topics/winsoft-085727.html](https://www.oracle.com/technetwork/topics/winsoft-085727.html)

CSDN：[http://download.csdn.net/download/ss123sswe/7166681](http://download.csdn.net/download/ss123sswe/7166681)

百度云：[https://pan.baidu.com/s/1kTS1hif](https://pan.baidu.com/s/1kTS1hif) 密码: a8zr

下载下来的 Oracle Client是解压版的，因此只要需要解压了。将下载的Oracle Client文件instantclient-basic-win32-11.2.0.1.0.zip（这是客户端，必须是32位）解压到d:\app\（解压到别的地方也可以，只是后面的配置需要按照这个进行）。然后在解压后的D:\app\instantclient_11_2目录下新建NETWORK\ADMIN目录，在ADMIN目录下新建tnsnames.ora文件，添加数据库TNS。

```
ORCL =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl.servyou.local)
    )
  )
```

tnsnames.ora文件也可以从Oracle数据库HOME目录的NETWORK\ADMIN目录D:\app\clg\product\11.2.0\dbhome_1\NETWORK\ADMIN拷贝过来，还要把sqlnet.ora拷贝过来。由于是连接本地数据库，所以host写成localhost即可。

个人觉得**采用拷贝的方式比较好**，手写的时候前面一定不能有空格，否则无法识别。

## 配置PL/SQL的Oracle Home和OCI Libaray

以非登录模式进入PL/SQL，按照同样的方法设置路径，将Oracle Home路径指定为Oracle Client目录（D:\app\instantclient_11_2），OCI Libaray路径为Oracle Client目录下面的oci.dll (D:\app\instantclient_11_2\oci.dll)。具体配置情况如下：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-PLSQL32%E4%BD%8D%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%85%8D%E7%BD%AE.png)

配置完成之后，保存并推出PL/SQL。

## 验证PL/SQL是否可以识别新的Oracle Client

打开PL/SQL，会发现：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-PLSQL%E6%AD%A3%E5%B8%B8%E7%99%BB%E9%99%86%E7%95%8C%E9%9D%A2.png)

下方出现了Connect as选项，可以选择Normal，SYSDBA等等。

输入用户名和密码，就可以登录。

登录进去之后我们可以检查一下能否查询数据：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-PLSQL%E6%9F%A5%E8%AF%A2%E7%95%8C%E9%9D%A2.png)

查询成功，dual表中确实只有一个记录X。

查询没有问题，也就是实现了PL/SQL Developer连接本地Oracle 11g 64位数据库。