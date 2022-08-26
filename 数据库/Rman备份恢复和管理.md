---
abbrlink: 4122458799
alias: 2016/03/18/Rman备份恢复和管理/index.html
categories:
- 数据库
date: '2016-03-18T15:47:01'
description: ''
tags:
- Rman
- 备份
- 恢复
title: Rman备份恢复和管理
---







![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-Rman%E7%BB%84%E4%BB%B6.jpg)

参考资料：

- [Oracle之Rman入门指南](https://www.cnblogs.com/Ronger/archive/2011/12/29/2306367.html)


- [一步一步学Rman](http://www.5ienet.com/note/html/rman/index.shtml)

## Rman简介

Rman-Recover manager恢复管理工具。

Oracle集成了很多环境的一个数据库备份和恢复的工具。

Rman可以做下列事情：

- 数据库热备份
  - 全库备份和恢复
    - 数据库克隆（DG）
  - 增量备份和恢复
  - 表空间备份和恢复
  - 数据文件备份和恢复
  - 归档备份和恢复
  - 控制文件和参数文件备份和恢复
- 数据库冷备份
- 备份集的管理
  - 备份策略
  - 保留和删除备份数据
  - ......

<!--more-->

**冷备份和热备份区别**：

对于oracle数据库只有物理备份和逻辑备份

- 物理备份：是将实际组成数据库的操作系统文件从一处拷贝到另一处的备份过程，通常是从磁盘到磁带
- 逻辑备份：是利用SQL语言从数据库中抽取数据并存于二进制文件的过程。

**物理备份**用于实现数据库的完整恢复，但**数据库必须运行在归挡模式**下（业务数据库在非归挡模式下运行），且需要极大的外部[存储](http://www.storworld.com/)设备，例如磁带库，**具体包括冷备份和热备份。冷备份和热备份是物理备份(也称低级备份)**，它涉及到组成数据库的文件，但不考虑逻辑内容。

- **冷备份发生在数据库已经正常关闭**的情况下，当正常关闭时会提供给我们一个完整的数据库
- 热备份是在数据库运行的情况下，采用archivelog mode方式备份数据库的方法。

热备份和冷备份可以参看：[什么是冷备份和热备份，有什么区别？](http://news.newhua.com/news/2010/0601/93935.shtml)

**应该备份哪些文件？**

- Oracle数据文件
- 控制文件
- 归档日志
- 在线日志
- 参数文件
- 密码文件

## Rman备份实验演示

**备份数据库**

在数据库运行的时候进行Rman备份则是热备份，需要当前数据库处于归档模式

检查数据库是否是归档模式的命令：

```
sqlplus / as sysdba
SQL> archive log list;
```

如果处于archive mod下，则可以进行热备。使用quit退出sqlplus状态，进入恢复管理器。

```
C:\Users\clg>rman target /

恢复管理器: Release 11.2.0.1.0 - Production on 星期一 3月 21 14:46:00 2016

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.

连接到目标数据库: ORCL (DBID=1433387646)
```

备份数据库的命令：（	全备）

```
RMAN> backup database;
```

会备份数据文件和控制文件还有spfile。

**备份表空间**

可以备份某个特定的表空间

```
RMAN> backup tablespace users;
```

**备份文件**

备份制定的文件，根据文件号备份。

```
RMAN> backup datafile 4;
```

**备份归档日志**

```
RMAN> backup archivelog all;
```

**查看备份信息**

```
RMAN> list backup;
```

如果备份的时候恢复区的空间不够，超出了恢复文件数的限制，那么就会出现backup失败。则可以删除之前的备份。

```
RMAN> delete backupset;
或者
RMAN> delete backup;
```

使用这两条命令都会删除备份片段列表。

Rman可以发出一些管理类的SQL语句。

**从备份文件中恢复数据库文件**

先使用restore命令从备份集中拷贝数据库文件到oradata文件夹下（数据库存放数据文件的地方）。

```
restore database;
或者
restore tablespace user;
或者
restore datafile 4;
```

第一个是将整个数据库的数据文件拷贝过来，第二个只是拷贝表空间user的数据文件。

然后使用recover进行介质恢复。

```
recover database;
或者
recover datafile 4;
```

根据拷贝过来的数据文件dbf进行全库恢或者根据具体的文件进行恢复。

## Rman增量备份

[使用Rman 全备份以及增量备份](http://gavinshaw.blog.51cto.com/385947/593340/)

附：

关于数据文件的状态信息（online or offline等等），可以参见v$datafile视图。