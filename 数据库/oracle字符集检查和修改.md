---
abbrlink: 1725623659
alias: 2016/12/01/oracle字符集检查和修改/index.html
categories:
- 数据库
date: '2016-12-01T22:51:15'
description: ''
tags:
- Oracle
- 字符集
title: Oracle字符集检查和修改
---








# Oracle字符集检查和修改

在部署重构版测试环境时，需要创建Oracle数据库，使用dbca创建数据库之后没有注意数据库本身的字符集，导致后续所有的数据库脚本执行后中文乱码。最后的解决办法是清掉全库数据，再修改字符集，重启数据库。

## 1、Oracle字符集概述

系统或者程序运行的环境就是一个我们常见的locale。而设置数据库locale最简单的方法就是设置NLS_LANG这个环境参数。在linux中NLS_LANG是一个环境变量，在windows中NLS_LANG是写在注册表中的。NLS_LANG这个参数由三个组成部分，分别是语言（language）, 区域（territory）和字符集（character set），格式如下：

```shell
NLS_LANG = language_territory.charset
```

我们平时最常见的就是：`AMERICAN_AMERICA.ZHS16GBK`和`SIMPLIFIED CHINESE_CHINA.ZHS16GBK`

NLS_LANG的作用官网是这样说的：

- It sets the language and territory used by the client application and the database server. It also sets the client's character set, which is the character set for data entered or displayed by a client program

<!--more-->

意思就是说：

- NLS_LANG设置了客户端应用程序和数据库服务器使用的语言和区域。它还设置了客户端的字符集，这是**客户端程序用于数据输入或者显示的字符集**。也就是说如果客户端字符集和NLS_LANG中的charset不同，则会乱码。

## 2、检查Oracle Server字符集

检查Oracle Server字符集最常用的方法有两种

### ▶查询nls_database_parameters

```sql
select * from nls_database_parameters;
```

### ▶使用userenv函数

userenv函数返回当前会话（session）的相关信息。以下sql语句可以查询当前会话连接的数据库字符集

```sql
select userenv('language') from dual;
```

有关`userenv('parameter')`返回值的官网介绍如下

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/oracle/2016-11-29_194757.jpg)

意思就是：返回的是当前会话使用的language和territory。characterset是数据库的字符集。

userenv函数的具体使用和当前会话字符集的取值详见以下链接

[oracle的userenv和nls_lang详解][1]

## 3、修改Oracle Server字符集

一旦数据库创建后，数据库的字符集理论上讲是不能改变的。因此，在设计和安装之初考虑使用哪一种字符集十分重要。根据Oracle的官方说明，字符集的转换是从子集到超集受支持,反之不行。如果两种字符集之间根本没有子集和超集的关系，那么字符集的转换是不受oracle支持的。对数据库server而言，错误的修改字符集将会导致很多不可测的后果，可能会严重影响数据库的正常运行，所以在修改之前一定要确认两种字符集是否存在子集和超集的关系。一般来说，除非万不得已，我们不建议修改oracle数据库server端的字符集。

以下是修改server端字符集的方法——不建议使用

```sql
SQL> conn /as sysdba 
SQL> shutdown immediate; 
SQL> startup mount 
SQL> ALTER SYSTEM ENABLE RESTRICTED SESSION; 
SQL> ALTER SYSTEM SET JOB_QUEUE_PROCESSES=0; 
SQL> ALTER SYSTEM SET AQ_TM_PROCESSES=0; 
SQL> alter database open; 
SQL> ALTER DATABASE CHARACTER SET ZHS16GBK; 
ALTER DATABASE CHARACTER SET ZHS16GBK 
* 
ERROR at line 1: 
ORA-12712: new character set must be a superset of old character set 
提示我们的字符集：新字符集必须为旧字符集的超集，这时我们可以跳过超集的检查做更改： 
SQL> ALTER DATABASE character set INTERNAL_USE ZHS16GBK; 
SQL> select * from v$nls_parameters; 
重启检查是否更改完成： 
SQL> shutdown immediate;  
SQL> startup 
SQL> select * from v$nls_parameters; 
```

具体使用方法参见：[oracle服务器和客户端字符集的查看和修改][2]

## 4、检查Oracle Client字符集

windows查看nls_lang

```shell
set NLS_LANG
```

linux查看nls_lang

```shell
echo $NLS_LANG
```

## 5、修改Oracle Client字符集

修改客户端字符集只需要修改上述检查结果中的NLS_LANG即可。

## 6、整理补充 

**▶数据库字符集** 

```sql
select * from nls_database_parameters ;
select userenv('language') from dual;
```

- 以上两种方法取得的都是数据库字符集，来源于props$，是表示数据库的字符集。

**▶实例字符集** 

```sql
select * from nls_instance_parameters; 
```

- 主要涉及NLS_LANGUAGE、NLS_TERRITORY的值. NLS_INSTANCE_PARAMETERS其来源于v$parameter

**▶会话字符集** 

```sql
select * from nls_session_parameters; 
```

- 来源于v$nls_parameters，表示会话自己的设置，可能是会话的环境变量或者是alter session完成，如果会话没有特殊的设置，将与nls_instance_parameters一致。

详见oracle官网：[NLS Database Parameters][3]

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/oracle/NLS%20Data%20Dictionary%20Views.jpg)

--- 

[1]: https://suncle.me/2016/12/01/oracle%E7%9A%84userenv%E5%92%8Cnls-lang%E8%AF%A6%E8%A7%A3/
[2]: https://blog.csdn.net/dream19881003/article/details/6800056
[3]: https://docs.oracle.com/cd/E11882_01/server.112/e10729/ch3globenv.htm#NLSPG194