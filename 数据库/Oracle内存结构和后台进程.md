---
abbrlink: 943732234
alias: 2016/03/16/Oracle内存结构和后台进程/index.html
categories:
- 数据库
date: '2016-03-16T11:22:28'
description: ''
tags:
- Oracle
- 内存结构
- 后台进程
title: Oracle内存结构和后台进程
---









![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-Oracle%E5%86%85%E5%AD%98%E5%92%8C%E5%90%8E%E5%8F%B0%E8%BF%9B%E7%A8%8B.png)

**Oracle实例=内存+后台进程**

**Oracle数据库=实例+物理存储结构**

由上图可知Oracle实例（一个Instance）由内存结构和程序结构组成，内存结构主要是SGA，程序结构主要是后台进程。

物理存储结构主要是数据库文件。

这次仔细学习了Oracle实例的具体内容。

## 为什么Oracle数据库做的这么复杂？

**内存**

- 数据查询的速度
- 更好的提升数据处理的速度

**后台进程**

- 为了完成特定的任务的服务进程

**数据文件**

- 数据的永久性保存
- 也是数据库性能下降的主要原因

## 实例之内存结构

### System global area(SGA)

SGA区包括Oracle实例需要的一系列内存组件，用于存放数据信息和数据控制信息，这些**内存信息被所有进程所共享**。SGA的各个组成包括：

| 组成                        | 描述                                       |
| :------------------------ | :--------------------------------------- |
| **Database buffer cache** | 数据库高速缓冲区，里面存放着从磁盘读取到内存的数据块，这些数据块可以被所有的会话访问，是全局共享的。buffer cache分为三个部分，分别是default pool，keep pool，recycle pool。default pool是正常情况下，数据块存放的内存区域，default pool会根据一个过期算法（LRU，Least Recently Used，近期最少使用）将过期的脏数据（修改过的数据，没有修改的数据可以不写到磁盘上）写到磁盘上。keep pool这个区域用于将一些数据始终固定在内存中。recycle pool存放一些不经常使用的数据块，避免这些数据块在default pool中占据空间。 |
| **Shared pool**           | 共享池缓存着一些用户可以共用的信息：1、可以重新使用的SQL语句  2、存放一些数据字典的信息，包括用户账户数据，表，索引，权限等等。  3、存储存放在数据库中的可执行代码。 |
| **Redo log buffer**       | 重做日志缓冲区，存放着数据库操作产生的redo数据，Redo log buffer以循环的方式写入，当redo log已经写到磁盘后，就可以被后续的日志数据覆盖。 |
| **Large pool**            | 对数据的处理不适用LRU算法，比shared pool更搞笑的内存收取方式。并行执行时会使用large pool。Rman备份时启动并行备份方式时，使用large pool。 |
| **Java pool**             | 这块内存区域用来存放所有特定会话的JVM（Java Virtual Machine）中的java代码和数据。 |
| **Streams pool**          | 里面存放着流相关的信息，比如流队列，其中也会流复制中capture进程提供进程内存空间。Streams pool只为流复制提供内存空间，如果没有手工配置，也没有配置流复制，这个空间将设置为0。 |
| **Result cache**          | 结果缓存，当表的访问方式以读为主前提下，从一张大表中过滤出少量的记录作为结果集，把查询结果集放入result cache，后续相同的查询语句可以直接从result cache里获取想要的结果，省去了CPU、I/O上的开销。这个SGA组件加速了频繁运行的查询语句的执行速度。 |

<!--more-->

### Program global area(PGA)

不同于SGA，PGA属于独占式内存区，它的数据和控制信息为某个会话所独有，当一个会话产生时，Oracle会为这个会话分配一个PGA内存区域。可以理解为操作系统在一个进程启动时，为他分配的内存空间，是一个操作系统含义上的内存区。

### User global Area(UGA)

UGA中保存和当前会话相关的信息，比如会话登录的信息，pl/sql的变量，绑定变量的值等等。UGA随着连接方式不一样可以在SGA中也可以在PGA中。

### Software code areas

Oracle存放自身软件代码的一部分内存区，不允许其他会话访问

## 后台进程

**Oracle的进程**

用户进程	user process

服务器进程 server process

实例后台进程 background process

**windows查看Oracle有哪些后台进程**

```
SQL> select program from v$session where program like 'ORACLE.EXE%';

PROGRAM
----------------------------------------------------------------
ORACLE.EXE (GEN0)
ORACLE.EXE (DIA0)
ORACLE.EXE (CKPT)
ORACLE.EXE (MMNL)
ORACLE.EXE (RVWR)
ORACLE.EXE (ARC0)
ORACLE.EXE (QMNC)
ORACLE.EXE (ARC1)
ORACLE.EXE (DIAG)
ORACLE.EXE (MMAN)
ORACLE.EXE (SMON)

PROGRAM
----------------------------------------------------------------
ORACLE.EXE (Q001)
ORACLE.EXE (SMCO)
ORACLE.EXE (PMON)
ORACLE.EXE (DBRM)
ORACLE.EXE (DBW0)
ORACLE.EXE (RECO)
ORACLE.EXE (ARC2)
ORACLE.EXE (Q002)
ORACLE.EXE (CJQ0)
ORACLE.EXE (W000)
ORACLE.EXE (VKTM)

PROGRAM
----------------------------------------------------------------
ORACLE.EXE (PSP0)
ORACLE.EXE (LGWR)
ORACLE.EXE (MMON)
ORACLE.EXE (ARC3)

已选择26行。
```

下面重点看一些Oracle后台进程（链接内存和磁盘的桥梁）

### 系统监控进程SMON

Oracle数据库至关重要的一个后台进程，SMON 是System Monitor 的缩写，意即：系统监控。

**SMON的主要工作：**

- 数据库启动时的实例恢复，在RAC环境下，一个节点的SMON可以对另外一个节点做实例恢复


- 清理和释放临时段上的数据（排序、临时表…）


- 对于DMT（字典管理表空间），SMON可以合并连续空闲的extent


- 维护回滚段的online，offline以及空间的回收

### 进程监控进程PMON

PMON是Process Monitor的缩写，PMON主要有下面的用途：

- 在进程非正常中断后，做清理工作


- 在进程abort后，PMON进行清理工作。


- PMON的第三个用途是，向Oracle TNS listener注册实例信息。

### 数据库写进程DBWn

DBWn是Database writer的缩写，n代表可以设置多个写进程。

DBWn负责把缓冲区的脏数据写到磁盘上，DBW进程是分散地把数据写到磁盘上的。而LGWR是连续写redo log。分散写要比连续写耗时的多。

**DBWn触发条件：**

- 当buffer cache空间不足时触发。


- DBWn接到checkpoint的指令时触发。

### 日志写进程LGWR

LGWR是把SGA中redo log buffer的信息写到redo log file的进程。LGWR是顺序写入到redo log file中，因此速度很快。LGWR会在下面情况发生：

1，每隔3秒钟，进行一次LGWR

2，任何事务进行了commit操作

3，当redo log buffer是1/3满，或者里面有1MB的数据

基于以上的原因，**把redo log buffer设置的很大就没必要的**。

### 检查点进程CKPT

CKPT是checkpoint的缩写，根据checkpoint信息和DBW向磁盘写数据块的信号，CKPT更新控制文件和数据文件头。 Checkpoint information includes the checkpoint position, SCN, location in online redo log to begin recovery, and so on. CKPT 既不向数据文件中写数据块，也不向online redo log files写redo块。

### 归档进程ARCn

ARCn，Archive归档进程。ARCn的工作是在LGWR把onlone redo log填满后，ARCn把redo log file的内容copy到其他的地方。（也就是说是把联机重做日志变成归档日志）。online redo log 是被用来为实例失败的时候，恢复数据文件。而归档日志是被用来在media recovery的时候，恢复数据文件。

### checkpoint和commit的区别

**commit**的作用是提交那些事务修改的数据产生的日志，即触发LGWR将redo log buffer中的内容写到redo log files，此时并没有把真正的数据写到磁盘上。**commit的目的就是为了写到redo log files中去保护数据。**

**checkpoint**会触发DBWn进程，将脏数据块写到数据文件中。如果DBWR进程要将事务的结果写入数据文件，但发现要写入的脏数据块相关的重做信息仍然处于重做日志缓存中，它将通知oracle启动LGWR进程，先将这些重做信息写入重做日志文件，直到重做信息全部被写入后，DBWR进程才开始将脏缓存写入数据文件。所以**checkpoint的目的是保证数据一致性。**



参考：

[Oracle官网-Managing the Oracle Instance](https://docs.oracle.com/cd/E25054_01/server.1111/e10897/instance.htm)

[Oracle官网-Process Architecture](https://docs.oracle.com/cd/E29597_01/server.1111/e25789/process.htm)