---
categories:
  - 数据库
date: '2016-03-16T15:08:49'
description: ''
tags:
  - Oracle
  - 存储
  - 数据结构
title: Oracle数据结构
---



## Oracle的数据存储结构

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-Oracle%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8%E7%BB%93%E6%9E%84.png)

- 表空间（tablespace）--Oracle中最大的逻辑存储单位
- 数据文件（data file）--表空间物理存储载体
- 段（segment）--Oracle中所有占用空间的对象的总称
- extend--段的组成单位
- 数据块（data block）--extend的组成单位，是Oracle存储和数据操作的最小单位。

## 数据块

数据块是Oracle存储和数据操作的最小单位，但不一定和操作系统的os块相同，一个数据块可能有多个os块构成。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-%E6%95%B0%E6%8D%AE%E5%9D%97%E5%92%8Cos%E5%9D%97.png)

<!--more-->

**数据块的存储属性**

*PCTFREE (percent free)*

为一个块**保留的空间百分比**，表示数据块在什么情况下可以被insert，默认是10，表示当数据块的可用空间低于10%后，就不可以被insert了，只能被用于update；即：当使用一个block时，在达到pctfree之前，该block是一直可以被插入的，这个时候处在上升期。

*PCTUSED (percent used)*

当数据块的剩余空间达到PCTFREE之后就不可以insert了，但是进行的delete操作和update操作会释放数据块的空间，如果数据块的空间释放到了PCTUSED之后就可以开始insert数据了。

**行链接和行迁移**

***行迁移*——update操作引起的**

当一条记录被更新时，数据库引擎首先会尝试在它保存的数据块中寻找足够的空闲空间，如果没有足够的空闲空间可用，这条记录将被拆分为两个部分，第一个部分包括指向第二个部分的rowid，该部分任然保留在原来的数据块中，第二个部分包含所有的具体数据，将保存到另外一个新的数据块中，这个就成为行迁移。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-%E8%A1%8C%E8%BF%81%E7%A7%BB%E5%9B%BE%E8%A7%A3.png)

为什么不将整行都放到新的数据块中？
原因是这样会导致该行数据rowid发生变化，而rowid被存储在索引中，也有可能被客户端临时保存在内存中，rowid的变化可能导致查询错误。该行不仅存了本行id还有新行的id。相当于存入了指针，并且保留了头指针。

***行链接*——insert操作或者update操作引起的**

行链接和行迁移不同，行链接是当一条记录太大，在一个数据块中无法存入，这时会被拆分为2个或以上的部分，存储在多个块中，这多个块之间会构造一个链

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-%E8%A1%8C%E9%93%BE%E6%8E%A5%E5%9B%BE%E8%A7%A3.png)

行链接的原因则可能为：

- 直接插入大的记录；


- 更新记录导致记录大于一个数据块，在这时，这样记录可能会同时变为行迁移和行链接。

行迁移和行链接的参考资料：

[http://www.2cto.com/database/201410/344139.html](http://www.2cto.com/database/201410/344139.html)

**数据块空闲空间**

- 可以手工对数据块进行空闲空间合并，数据会被再次使用
  - 当一个插入或者更新操作哦的行在一个数据块中有足够的空闲空间
  - 并且这个空闲空间是碎片状态，无法满足一行数据的使用


- oracle不总是自动整碎片的原因是，这会导致一定的系统资源开销

**索引数据块的整理**

Alter index coalesce——合并同一个branch的数据块（coalesce，合并，聚合）

Alter index REBUILD——重构整个索引段

```
SQL> create table t1(id int,score int);

表已创建。

SQL> create index idx_st on t1(id);

索引已创建。

SQL> alter index idx_st coalesce;

索引已更改。

SQL> alter index idx_st rebuild;

索引已更改。
```

**Oracle的读操作**

①逻辑读：从内存中读取数据块

②物理读：从磁盘读取数据块到内存

单块读：每次从磁盘读取一个数据块

多块读：每次从磁盘读取多个数据块

## Extent-区间

是由一组连续的数据块组成，多个extent构成一个段（segment）。

Oracle11g中创建表之后并不会立刻就分配extent，只有在插入一个数据之后才会分配extent。

关于extent的具体实验效果可以参见如下资料：

[http://blog.itpub.net/12798004/viewspace-1250248/](http://blog.itpub.net/12798004/viewspace-1250248/)

## Segment-段

### 什么是段？

在Oracle表中，凡是分配了空间的对象，都称之为段。

- 表，表分区
- 索引，索引分区
- 大对象（LOB，large object）

### 段的分类

- 数据段
- 临时段
- 回滚段

### 临时段

也成为临时表空间。存在临时表空间中的数据成为临时段。

- 排序，hash，merge...（需要一个中间数据处理区域）
- 只有在内存空不足时，Oracle才会在临时表空间上创建临时段。
- 临时段上的操作并不记录redo log

#### 临时表

**临时表的概念**

临时表就是用来暂时保存临时数据（亦或叫中间数据）的一个数据库对象。

- Oracle的临时表只存在于某个会话或者事务的生命周期里，此时临时表中的数据只对这个会话可见。
- 临时表经常被用于存放一个操作的中间数据（数据处理的中间环节）
- 临时表由于不产生redo，能够提高数据操作的性能。

**临时表的分类**

根据on commit的设定，可以将临时表分为两类，会话级的临时表和事务级的临时表。

- on commit delete rows

> 临时表的默认参数，表示临时表中的数据仅在事务（transaction）过程中有效，当事务提交（commit），临时表的临时段将被自动截断（truncate），**但是临时表的结构以及元数据还存在用户的数据字典中，如果临时表完成使命之后，最好删除临时表，否则数据库会残留很多临时表的表结构和元数据。**

- on commit preserve rows

> 它表示临时表的内容可以跨事物而存在，不过，当该会话结束时，临时表的暂时段将随着会话的结束而被丢弃，临时表中的数据自然也就随之丢弃。**但是临时表的结构以及元数据还存储在用户的数据字典中。如果临时表完成它的使命后，最好删除临时表，否则数据库会残留很多临时表的表结构和元数据。**

**临时表-on commit delete rows**

```
SQL> create global temporary table t_temp on commit delete rows as select * from
 dba_objects;

表已创建。

SQL> select count(*) from t_temp;

  COUNT(*)
----------
         0

SQL> insert into t_temp select * from dba_objects;

已创建72635行。

SQL> select count(*) from t_temp;

  COUNT(*)
----------
     72635

SQL> commit;

提交完成。

SQL> select count(*) from t_temp;

  COUNT(*)
----------
         0
```

> 解释：
>
> 创建临时表空的create语句属于DDL语句，虽然创建的时候有初始数据，但是创建之后就相当于进行了一次commit，所以t_temp中并没有数据。insert插入数据之后t_temp表中就有了72635条数据。经过commit操作，数据就直接truncate掉了，但是表还存在着。

**临时表-on commit preserve rows**

```
SQL> create global temporary table t_temp on commit preserve rows as select * from dba_objects;

表已创建。

SQL> select count(*) from t_temp;

  COUNT(*)
----------
     72675

SQL> insert into t_temp select * from dba_objects;

已创建72675行。

SQL> select count(*) from t_temp;

  COUNT(*)
----------
    145350

SQL> quit;
从 Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options 断
开

C:\Users\clg>sqlplus / as sysdba

SQL*Plus: Release 11.2.0.1.0 Production on 星期三 3月 16 20:34:08 2016

Copyright (c) 1982, 2010, Oracle.  All rights reserved.


连接到:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

SQL> select count(*) from t_temp;

  COUNT(*)
----------
         0

SQL> drop table t_temp;

表已删除。
```

关于临时表的使用可以参见：

[https://www.cnblogs.com/kerrycode/p/3285936.html](https://www.cnblogs.com/kerrycode/p/3285936.html)

**临时表-索引**

临时表也是可以创建索引的，基本使用和表-索引差不多。



### 段的压缩

Oracle允许对段进行压缩。oracle的数据段压缩技术可以理解为"块级压缩"技术，也就是说是针对block级别的数据压缩。它是在block中引入记号表(symbol表)，block中的重复数据在symbol中用一个项(指针)表示，即块中相同的row只存储一条，从而节约了空间。

优点：

- 减少存储空间
- 减少处理的数据块
  - 减少内存占用
  - 提高I/O效率
  - 提高查询效率

缺点：

- 因为要额外对数据做处理，在数据插入时，会消耗更多的资源和时间。

**创建压缩表**

```
create table t_comp compress;
```

或者是：下创建表，后激活压缩

```
SQL> create table t1(id int);

表已创建。

SQL> alter table t1 compress;

表已更改。
```

**段压缩的级别**

- 表空间级


- 表级


- 分区


- 子分区

### 段的存储管理

MSSM--Manual Segment Space Management

- 手工设定对象的存储参数：PCTFREE，PCTUSED......

ASSM--Automatic Segment Space Management

- Oracle自动设定对象的存储参数：只可以手工设定PCTFREE参数，其他参数由Oracle自动设定。

## Tablespace-表空间

### 大文件表空间：bigfile字段

普通的数据文件，收到数据块的限制

- 每个数据文件最多只能包含2^22-1（4M）个数据块
  - 2k--8G
  - 4k--16G
  - 8k--32G
  - ...

大数据文件,可以使用2^32（4G）个数据块

- 2k--8T
- 4k--16
- ...T

大数据表空间的优势：

- 减少数据库的数据个数限制（每个数据库64k个数据文件）
- 方便文件的管理，不需要人工干预表空间的文件大小。
- 减少数据库对文件头同步的开销。

### 表空间的管理方式

**字典管理表空间**（Dictionary-managed tablespaces）

所有表空间存储在数据字典中，统一调配。现在基本不用

**本地管理表空间**（locally managed tablespace）

本地管理表空间不是在数据词典里存储表空间的，由自由区管理的表空间。用位图来自由的管理区间。一个区间对一个位，如果这个位是1表示已经被占用，0表示未被占用。

词典管理空间表示“中央集权治”，本地管理表空间表示“省市自治区”，一个databases表示中国，tablespaces表示一个省或直辖市。词典管理统一由中央调配。而**本地管理表示有高度的自治权利，自已各种资源的分配不用上报中央。**

### 表空间的存储属性

每一个级别的都有自己的管理方式。

数据管理方式

- local
- dictionary

段管理

- ASSM
- MSSM

extent管理

- AutoAllocate（自动分配）
- Uniform（统一大小分配）