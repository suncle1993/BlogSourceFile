---
categories:
  - 数据库
date: '2016-03-21T17:32:01'
description: ''
tags:
  - flashback
  - 闪回
  - Oracle
title: 闪回flashback
---



![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-flashback_pic.png)

**参考资料：**[Using Oracle Flashback Technology](https://docs.oracle.com/cd/E11882_01/appdev.112/e41502/adfns_flashback.htm)

Oracle 11g的新特性闪回操作

- 闪回查询
  - 闪回查询
  - 闪回版本查询
  - 闪回事务查询
- 闪回数据
  - 闪回表
  - 闪回删除
  - 闪回数据
  - 闪回归档

下面会分别介绍这些操作。在介绍这些操作之前先看下闪回特性是否开启。

<!--more-->

## 检查闪回特性是否启用

**参考资料：**

[Configuring Your Database for Oracle Flashback Technology](https://docs.oracle.com/cd/E11882_01/appdev.112/e41502/adfns_flashback.htm#ADFNS01002)

[打开或关闭oracle数据库的闪回功能步骤 ](http://blog.itpub.net/26194851/viewspace-763582/)

确认数据库闪回特性已经启用:v$database

```
SQL> select flashback_on from v$database;

FLASHBACK_ON
------------------
NO
```

如果闪回特性没有启用，则需要先启用闪回。

打开闪回的步骤：

```
SQL> shutdown immediate;
SQL> startup mount;
SQL> alter database flashback on;
SQL> alter database open;
```

打开之后可再次检查闪回特性是否打开。只要打开了闪回特性，就可以进行闪回操作。

## 闪回查询

**参考资料：**[Using Oracle Flashback Query (SELECT AS OF)](https://docs.oracle.com/cd/E11882_01/appdev.112/e41502/adfns_flashback.htm#i1008579)

查询某一个历史时间点的数据。

**查询某个时间点表中的数据——某个时间点表中快照的数据。**

```
SQL> create table t(x int , y int);

表已创建。

SQL> insert into t values(1,2);

已创建 1 行。

SQL> insert into t values(2,3);

已创建 1 行。

SQL>
SQL> commit;

提交完成。

SQL> select * from t;

         X          Y
---------- ----------
         1          2
         2          3

SQL> set time on
21:11:45 SQL> delete from t where x=1;

已删除 1 行。
21:12:11 SQL> commit;

提交完成。

21:12:18 SQL> select * from t;

         X          Y
---------- ----------
         2          3

21:12:21 SQL> select * from t as of timestamp to_timestamp('2016-03-21 21:11:45'
,'yyyy-mm-dd hh24-mi-ss');

         X          Y
---------- ----------
         1          2
         2          3
```

可见删除的数据已经提交，但是还是可以闪回查询到之前时间的数据。其中hh24表示可以用24小时制，否则只能小时不能超过12。至于为什么分钟用mi而不用mm，那是因为规定的格式就是mi，换成mm会显示和之前的月份mm冲突，换成其他的会显示日期格式无法识别。

**基于SCN的闪回查询**

如果有修改时候的SCN，那么就可以基于SCN进行闪回查询

```
21:18:34 SQL> select * from t;

         X          Y
---------- ----------
         2          3

21:20:36 SQL> select current_scn from v$database;

CURRENT_SCN
-----------
    2891887

21:20:53 SQL> insert into t values(3,4);

已创建 1 行。

21:21:13 SQL> commit;

提交完成。

21:21:16 SQL> select * from t;

         X          Y
---------- ----------
         2          3
         3          4

21:21:26 SQL> select * from t as of scn 2891887;

         X          Y
---------- ----------
         2          3
```

可见基于SCN闪回查询得到插入数据之前表中的数据

## 闪回版本查询

**闪回版本查询**也就是flashback versions query。

- See all versions of a row between two times
- See transactions that changed the row

一个时间段内的某一行的所有版本，以及改变该行的所有事务。

**参考资料：**

[Using Oracle Flashback Version Query](https://docs.oracle.com/cd/E11882_01/appdev.112/e41502/adfns_flashback.htm#i1019938)

[闪回版本查询与闪回事务查询](https://blog.csdn.net/laoshangxyc/article/details/12405459)

## 闪回事务查询

**闪回事务查询**也即是flashback trasaction query

- See all changes made by a transaction

闪回一个事务对数据的所有改变。

**参考资料：**

[Using Oracle Flashback Transaction Query](https://docs.oracle.com/cd/E11882_01/appdev.112/e41502/adfns_flashback.htm#i1007455)

[闪回版本查询与闪回事务查询](https://blog.csdn.net/laoshangxyc/article/details/12405459)


## 闪回表

将表闪回到历史的某个时刻。

```
SQL> create table t(id int);

表已创建。

SQL> insert into t values(100);

已创建 1 行。

SQL> insert into t values(200);

已创建 1 行。

SQL> commit;

提交完成。

SQL> select rowid,id from t;

ROWID                      ID
------------------ ----------
AAASaxAABAAAV3xAAA        100
AAASaxAABAAAV3xAAB        200

SQL> select current_scn from v$database;

CURRENT_SCN
-----------
    2960668

SQL> delete from t;

已删除2行。

SQL> commit;

提交完成。

SQL> select * from t;

未选定行

SQL> alter table t enable row movement;

表已更改。

SQL> select * from t;

未选定行

SQL> flashback table t to scn 2960668
  2  ;
flashback table t to scn 2960668
                *
第 1 行出现错误:
ORA-08185: 用户 SYS 不支持闪回


SQL> show recyclebin
SQL>

```

解释1：由于当前是sys用户，会显示：

ORA-08185: 用户 SYS 不支持闪回

闪回技术只适用于普通用户而不适用于sys用户。system用户也不适用。

解释2：必须要启用表的行移动功能，否则不能闪回表。

- alter table t enable row movement;

## 闪回删除

参考资料：[Oracle闪回功能详解](https://blog.csdn.net/heng_ji/article/details/17968279)

由于闪回技术不支持sys用户，所以使用普通用户测试。

Oracle10g以后，当我们删除表时，默认Oracle只是在数据库字典里面对被删的表的进行了重命名，并没有真正的把表删除。
**回收站(recyclebin)**：用来维护表被删除前的名字与删除后系统生成的名字之间的对应关系的数据字典，表上的相关对象（索引、触发器等）也会一并进入回收站。

### 查看回收站的状态

只有dba（sys,system）才有权限执行查询recyclebin的状态。

```
SQL> show parameter recyclebin;

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
recyclebin                           string      on
```

可以发现recyclebin是开启状态，如果发现是关闭状态，可以通过下面命令开启

```
alter system set recyclebin=on;
```

如果要关闭回收站功能，使用下面的命令：

```
alter system set recyclebin=off;
```

禁用后删除的对象将直接删除，不会写到Recyclebin中。

### 闪回删除实验效果

由于sys用户和system用户不支持闪回技术，所以我们需要切换到普通用户进行实验，实验效果如下：

```
SQL> create table t(id int);

表已创建。

SQL> insert into t values(1);

已创建 1 行。

SQL> show recyclebin;
ORIGINAL NAME    RECYCLEBIN NAME                OBJECT TYPE  DROP TIME
---------------- ------------------------------ ------------ -------------------

T                BIN$zqOXt7xBRA2SKyG5nGxb5w==$0 TABLE        2016-03-22:17:18:33


SQL> drop table t;

表已删除。

SQL> commit;

提交完成。

SQL> show recyclebin;
ORIGINAL NAME    RECYCLEBIN NAME                OBJECT TYPE  DROP TIME
---------------- ------------------------------ ------------ -------------------

T                BIN$lpoDEIX2SSGAQt9TL3gUTA==$0 TABLE        2016-03-22:17:22:24

T                BIN$zqOXt7xBRA2SKyG5nGxb5w==$0 TABLE        2016-03-22:17:18:33

SQL> select * from "BIN$lpoDEIX2SSGAQt9TL3gUTA==$0";

        ID
----------
         1

SQL> flashback table t to before drop;

闪回完成。

SQL> select * from t;

        ID
----------
         1
```

### 清空回收站操作

可以全部清空回收站，也可只清空回收站的部分文件。

**全部清空回收站**

```
SQL> purge recyclebin;
```

**部分清空回收站**

```
##假定回收站中有一个文件的ORIGINAL NAME是t1，永久删除他t1表
SQL> purge table t1;
```

### 回收站的空间管理

虽然Oracle并没真正删除被回收的表，但是在Oracle看来这一块空间已经是自由的（free）。使用下面的演示可以查看效果：（在普通用户下验证即可）

```
SQL> create table t as select * from user_objects;

表已创建。

SQL> select sum(blocks) from user_free_space where tablespace_name='USERS';

SUM(BLOCKS)
-----------
         96

SQL> drop table t;

表已删除。

SQL> select sum(blocks) from user_free_space where tablespace_name='USERS';

SUM(BLOCKS)
-----------
        104

SQL> show recyclebin;
ORIGINAL NAME    RECYCLEBIN NAME                OBJECT TYPE  DROP TIME
---------------- ------------------------------ ------------ -------------------

T                BIN$kMskKvD5SJqmwaC5WDQiAg==$0 TABLE        2016-03-22:18:31:43

SQL> select blocks from user_segments where segment_name='BIN$kMskKvD5SJqmwaC5WDQiAg==$0';

    BLOCKS
----------
         8
```

实验效果表明删除表之后用户自由空间增大了，并且大小刚好就是回收站中的对应的表的大小。可见这一块空间在Oracle看来已经是free_space了。

## 闪回归档

**参考资料：**

[Using Flashback Data Archive (Oracle Total Recall)](https://docs.oracle.com/cd/E11882_01/appdev.112/e41502/adfns_flashback.htm#BJFFDCEH)

[Oracle 11g 闪回数据归档](https://blog.csdn.net/summerycool/article/details/5925266)

闪回归档：Flashback Data Archive。**Oracle Total Recall**，**也即Oracle全面回忆功能。**

闪回数据归档可以和我们一直熟悉的日志归档类比，日志归档记录的是Redo的历史状态，用于保证恢复的连续性；而闪回归档记录的是UNDO的历史状态，可以用于对数据进行闪回追溯查询；后台进程LGWR用于将Redo信息写出到日志文件，ARCH进程负责进行日志归档；在Oracle 11g中，新增的后台进程FBDA（Flashback Data Archiver Process）则用于对闪回数据进行归档写出。

具体操作见参考资料。

## 闪回数据库

参考资料：[Oracle DB闪回（Flashback database）开启笔记](https://www.linuxidc.com/Linux/2014-09/107257.htm)

数据库的闪回

- 是Oracle不同于查询闪回和归档闪回的另外一种闪回机制
- Oracle 10g引入
- 需要配置闪回区域
- 记录数据块的修改，称为flashback logs（闪回日志）
- 通过后台恢复写入进程RVWR（Recovery Writer）来工作
- 就像一个向后转的按钮，让数据库向后回退。
- 可以用于人为失误操作或者业务的需要。

**闪回数据库架构Flashback database architecture**

![Flashback database architecture](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-Flashback%20database%20architecture.png)

开启闪回数据库功能之后，会在SGA中开辟内存Flashback buffer，会记录buffer cache中的部分改变然后后台恢复写入进程RVWR将记录写入闪回日志Flashback logs中。FBDA进程（Flashback Data Archive ）则会将Flashback logs进行归档。

这个过程和重做日志非常非常的类似。