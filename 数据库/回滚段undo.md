---
categories:
  - 数据库
date: '2016-03-16T14:42:45'
description: ''
tags:
  - Undo
  - Oracle
title: 回滚段undo
---


---

## Undo的作用

- 数据的回滚
- 一致性读
- 表的闪回（事务，查询的闪回..）
- 失败会话的恢复

## 回滚rollback操作

```
SQL> archive log list;
ORA-01031: 权限不足
SQL> conn /as sysdba
已连接。
SQL> archive log list;
数据库日志模式            存档模式
自动存档             启用
存档终点            USE_DB_RECOVERY_FILE_DEST
最早的联机日志序列     45
下一个存档日志序列   47
当前日志序列           47
SQL> create table t1(id int);

表已创建。

SQL> select * from t1;

未选定行

SQL> insert into t1 values('1');

已创建 1 行。

SQL> insert into t1 values('2');

已创建 1 行。

SQL> rollback;

回退已完成。

SQL> select * from t1;

未选定行

SQL> desc t1;
 名称                                      是否为空? 类型
 ----------------------------------------- -------- ----------------------------

 ID                                                 NUMBER(38)

SQL> select * from t1;

未选定行

SQL> insert into t1 values(1);

已创建 1 行。

SQL> insert into t1 values(2);

已创建 1 行。

SQL> select * from t1;

        ID
----------
         1
         2

SQL> rollback;

回退已完成。

SQL> select * from t1;

未选定行

SQL> archive log list;
数据库日志模式            存档模式
自动存档             启用
存档终点            USE_DB_RECOVERY_FILE_DEST
最早的联机日志序列     45
下一个存档日志序列   47
当前日志序列           47
SQL> shutdown immediate;
数据库已经关闭。
已经卸载数据库。
ORACLE 例程已经关闭。
SQL> startup mount;
ORACLE 例程已经启动。

Total System Global Area 3307048960 bytes
Fixed Size                  2180264 bytes
Variable Size            1828719448 bytes
Database Buffers         1459617792 bytes
Redo Buffers               16531456 bytes
数据库装载完毕。
SQL> alter database noarchivelog;
alter database noarchivelog
*
第 1 行出现错误:
ORA-38774: 无法禁用介质恢复 - 闪回数据库已启用


SQL> alter database flashback off;

数据库已更改。

SQL>
SQL> alter database noarchivelog;

数据库已更改。

SQL> alter database open;

数据库已更改。

SQL> archive log list;
数据库日志模式             非存档模式
自动存档             禁用
存档终点            USE_DB_RECOVERY_FILE_DEST
最早的联机日志序列     45
当前日志序列           47
SQL> select * from t1;

未选定行

SQL> insert into t1 values(1);

已创建 1 行。

SQL> insert into t1 values(2);

已创建 1 行。

SQL> select * from t1;

        ID
----------
         1
         2

SQL> rollback;

回退已完成。

SQL> select * from t1;

未选定行

SQL> insert into t1 values(1);

已创建 1 行。

SQL> insert into t1 values(2);

已创建 1 行。

SQL> commit;

提交完成。

SQL> rollback;

回退已完成。

SQL> select * from t1;

        ID
----------
         1
         2
```

可见rollback操作和当前数据库 归档模式并没有关系，只和commit操作有关，一旦commit就无法回滚。

如果没有指定 rollback 到哪一个保存点savepoint上，就意味着全部Rollback，而不是只是rollback一条操作。

<!--more-->

关于savepoint的操作见下面的命令：

```
SQL> drop table t1;

表已删除。

SQL> select * from t1;
select * from t1
              *
第 1 行出现错误:
ORA-00942: 表或视图不存在


SQL> create table t1(id int);

表已创建。

SQL> insert into t1 values(1);

已创建 1 行。

SQL> savepoint s1;

保存点已创建。

SQL> insert into t1 values(2);

已创建 1 行。

SQL> insert into t1 values(3);

已创建 1 行。

SQL> rollback to s1;

回退已完成。

SQL> select * from t1;

        ID
----------
         1
```

虽然可以rollback到保存点，但是一旦commit，所有的保存点就都没用了。



## undo的逻辑结构

回滚段的空间是可以循环利用的，就像是分块的圆盘，这个圆盘可以增加块，也可以回收块。

**undo的空间使用机制-增长**

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-undo%E7%A9%BA%E9%97%B4%E5%A2%9E%E9%95%BF%E6%9C%BA%E5%88%B6.png)

如图中所示，块4填满后需要继续向前填充，虽然块2是inactive的，但是中间隔着一个active的块1，所以不能向前覆盖。这个时候空间就必须要增长了，则会加入新的块5，然后就可以继续向块5中写入undo信息。

**undo的空间使用机制-回收**

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-undo%E7%A9%BA%E9%97%B4%E5%9B%9E%E6%94%B6%E6%9C%BA%E5%88%B6.png)

当块4块5块6连续并且都是inactive的时候，此时空间回收机制，可以将这几个块合并成单独的块，块6。



## 一致性读

回滚段解决了写操作不会阻塞读操作的问题。

一致性读并非总要去读回滚段。

**实现的一致性读产生的代价——ORA-01555**

ORA-01555: "snapshot too old: rollback segment number string with name "string" too small"

Cause: rollback records needed by a reader for consistent read are overwritten by other writers;

Action: if in Automatic Undo Management mode, increase undo_retention setting.otherwise,use larger rollback segments.

快照太久，回滚段太小，回滚记录被覆盖

具体可以参见：[ORA-01555 原因与解决](http://www.dbtan.com/2010/01/ora-01555-reason-and-solution.html)



## 自动管理Undo-AUM

Automatic Undo Management

查看undo配置信息：

```
SQL> show parameter undo;

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
undo_management                      string      AUTO
undo_retention                       integer     900
undo_tablespace                      string      UNDOTBS1
```

Undo配置参数含义

-DNDO_MANAGEMENT     undo的管理模式，分自动和手动

-UNDO_TABLESPACE      当前正在被使用的undo表

-UNDO_RETENTION        规定多长时间内，数据不能被覆盖。

\----------------------------------------- 

AUTO             表示undo 为自动管理模式。

900               表示在900秒内，undo上的数据不能被覆盖。

UNDOTBS1    是当前正在使用的undo表空间。

注意：undo_retention是一个动态调整的参数，同时，Oracle无法保证在这个保留时间内的undo数据不被覆盖，当undo空间不足时，Oracle将覆盖即使未过保留期的数据以释放空间。

强制保留undo_retention时间内的数据

- 设置undo tablespace guarantee属性
- 设置该属性之后也可以取消

```
SQL> alter tablespace undotbs1 retention guarantee;

表空间已更改。

SQL> alter tablespace undotbs1 retention noguarantee;

表空间已更改。
```



## Undo调优

Undo的设置取决于我们实际的生产系统。如何设置undo更合理地为我们工作呢？

**Undo表空间的大小**：

　　我们在创建一个undo表空间的使用，就指定了它的大小，这个大小一旦创建是不可变更的。设置过大，是一种浪费，设置过小，例如删除100万条记录，这些删除的记录都要临时存放到undo表空间中，如果undo的大小不能存储100万条记录，那么就会出问题。

**Undo数据的存放时间**：

　　也就是undo_retention 参数所对应的时间，undo上有数据存放时间与undo大小的密切关系。存放时间越长，需要的表空间越大。就像理发师的数量与理发师的效率的关系一样。理发师效率很高，一秒钟解决一个客户，那么就不需要太多的理发师傅。

**Undo表空间的历史信息**：

如何合理设置undo表空间的大小和存放时间呢？那么就需要参考历史记录

关于如何设置undo表空间的大小可以参见：

[【技术分享】如何确定或调整undo表空间的大小](http://support.huawei.com/ecommunity/bbs/10180041.html)

关于如何设置undo表空间的存放时间可以参见：

[undo_retention：确定最优的撤销保留时间](https://blog.csdn.net/zq9017197/article/details/14446165)







参考资料：[oracle undo 解析](https://www.cnblogs.com/fnng/archive/2012/09/23/2699110.html)