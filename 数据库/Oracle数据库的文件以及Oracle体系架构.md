---
categories:
  - 数据库
date: '2016-02-06T21:27:16'
description: ''
tags:
  - Oracle
  - 体系架构
  - CentOS
  - 文件
title: Oracle数据库的文件以及Oracle体系架构
---



## 第一部分、Oracle数据库的文件

### 1、参数文件:控制实例的行为的参数的集合

#### 参数文件的作用

- 设定数据库的限制
- 设置用户或者进程的限制
- 设定数据库资源的限制
- 调整系统的性能

#### 主要的参数文件

SGA_TARGET：Oracle在SGA区（SGA是Oracle最重要的一块内存区域，存放各种各样的数据、SQL解析以及redo日志等等）需要分配多大的内存。

PGA_AGGREGATE_TARGET：此参数用来指定所有session总计可以使用最大PGA（程序全局区，会话分配的内存）内存。SGA和PGA基本就是oracle使用的内存的总和。

DB_CACHE_SIZE：数据块缓冲缓存区大小

DB_FILES：db_files参数限制了数据库数据文件总的个数,datafiles数目达到db_files指定后数据库不能添加新的数据文件

LOG_ARCHIVE_DEST_n：此参数可以设置最多10(n=[1..10])个不同的归档路径

USER_DUMP_DEST：specifies the pathname for a directory where the server will write debugging trace files on behalf of a user process.



查看参数文件初始化值可以在oracle的官方文档上去看。在[Basic Initialization Parameters](https://docs.oracle.com/cd/E11882_01/server.112/e40402/initparams002.htm#REFRN00101)就可以查看各项初始化参数了。每个基本参数点进去就有了详细的说明。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-Oracle%E5%9F%BA%E6%9C%AC%E7%9A%84%E5%88%9D%E5%A7%8B%E5%8F%82%E6%95%B0.jpg)

<!--more-->

其中SGA_TARGET的初始化参数如下：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-OracleSGA_TARGET%E5%8F%82%E6%95%B0%E6%96%87%E4%BB%B6.jpg)



查看当前数据库的参数文件

``` 
show parameter SGA; /*显示和sga相关的参数*/

select name,value from v$parameter; /*显示所有的参数*/

show parameter spfile; /*显示spfile参数文件*/
```



spfile对应的SPFILEORCL.ORA是二进制文件，用show parameter spfile;可以显示该二进制文件的路径（D:\Oracle11g\Administrator\product\11.2.0\dbhome_1\database）。

使用create pfile from spfile;可以生成INITorcl.ORA文件，这是一个文本格式的文件，里面显示的参数可以直接修改。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-Oracle%E7%9A%84pfile%E7%9A%84%E4%BD%BF%E7%94%A8.jpg)

### 2、控制文件

#### 控制文件包含的信息

- 数据库名字（DBID）
- 数据库建立时间
- 数据文件，在线日志文件和归档文件三中文件的信息
- 表空间的信息
- Rman的备份信息

#### 控制文件的作用

- 它包含数据文件，在线数据文件，归档文件的信息，这些文件用于数据库open时的 文件验证。当数据库的架构改变时，比如增减，删除文件时，会更新数据文件。
- 包含了数据库恢复时候需要的一些信息，用于数据库的恢复。

#### 控制文件的结构

- 空间允许重用区

> 这个区域的信息是可以被重用的（覆盖的），当空间不足或者规则满足时，允许覆盖以前的信息，比如归档日志和Rman备份集的信息。

- 空间不允许重用区

> 这个区域的信息是不允许重用的（覆盖的），因为他们是数据库必须的信息，比如表空间，数据文件，在线日志文件等。

#### 控制文件丢失了怎么办？

- 备份控制文件

- 重建控制文件

  ​

> 参数文件和控制文件的丢失都不是致命的，都不会导致数据库的损坏。

### 3、数据文件

存放实际的数据，隶属于某个表空间。

查看表空间及对应的数据文件信息：

``` 
select file_name,tablespace_name from dba_data_files;
select file_name,tablespace_name from dba_temp_files;
```

#### 数据文件的损坏

需用通过备份恢复

- 还原备份文件
- 用归档+在线redo恢复

使用Redo信息恢复

- 创建新数据文件
- 用归档+在线Redo恢复

### 4、重做日志文件-Redo Log File

#### 重做日志文件的作用

- 保护数据的安全
- 恢复数据


- 数据同步和分析 -golden gate，Data guard

#### 日志文件损坏

活动日志损坏：数据丢失，数据库损坏

非活动日志损坏：数据不会丢失，可以重建日志文件

Oracle日志文件的状态可参见：[Oracle日志文件的状态current/active/inactive/unused](https://www.linuxidc.com/Linux/2014-09/106795.htm)



## 第二部分、Oracle体系架构

### Oracle整体架构图

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-Oracle%E6%95%B4%E4%BD%93%E6%9E%B6%E6%9E%84%E5%9B%BE.jpg)

由上图可知，主要分为三个部分：实例instance，数据库database，其他组成部分。上半部分中的Instance就是实例，有内存加进程构成，内存分为SGA（System Global Area）和PGA（Program Global Area）。下方的Database就是数据库，包含数据文件，控制文件，重做日志文件。数据库文件是一种相对静止的东西。

下面分别介绍SGA区，PGA区和后台进程。

### SGA区

http://blog.itpub.net/25264937/viewspace-694917/

### PGA区

http://blog.itpub.net/25264937/viewspace-694917/











<!--more-->