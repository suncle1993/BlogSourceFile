---
categories:
  - 大数据
date: '2018-04-16T19:08:21'
description: ''
tags:
  - Hadoop
  - MapReduce
  - HDFS
title: Hadoop、MapReduce、HDFS介绍
---




对于入门hadoop的初学者，首先需要了解一下三个部分：

- hadoop的生态环境
- MapReduce模型
- HDFS分布式文件系统

依次介绍这三个部分。

<!--more-->

# 初识hadoop

Hadoop数据存储与分析

hadoop提供了一个可靠的共享存储和分析系统。HDFS实现数据的存储，MapReduce实现数据的分析和处理。虽然Hadoop还有其他功能，但HDFS和MapReduce是核心价值。

Hadoop项目：

- Common：一系列组件和接口，用于分布式文件系统和通用I/O（序列化，Java RPC和持久化数据结构）
- Avro：一种序列化系统，用于支持高效、跨语言的RPC和持久化数据存储
- MapReduce：分布式数据处理模型和执行环境
- HDFS：分布式文件系统
- Pig：数据流语言和运行时环境，运行在MapReduce和HDFS集群上
- Hive：一种分布式的、按列存储的数据仓库。Hive管理HDFS中存储的数据，并提供基于SQL的查询语言（由运行时引擎翻译成MapReduce作业）用以查询数据
- HBase：一种分布式的、按列存储的数据库。HBase使用HDFS作为底层存储，同时支持MapReduce的批量式计算和点查询（随机读取）
- ZooKeeper：一种分布式的、可用性高的协调服务。ZooKeeper提供分布式锁之类的基础服务用于构建分布式应用
- Sqoop：该工具用于在结构化数据存储（如关系型数据库）和HDFS之间高效批量传输数据
- Oozie：该服务用于运行和调度hadoop作业（如MapReduce，Pig，Hive及Sqoop作业）



# MapReduce模型

分为MapReduce的定义和MapReduce的工作方式两个部分进行说明

## MapReduce的定义

MapReduce是一个适用于处理大量数据的编程模型。 Hadoop能够运行用各种语言编写的MapReduce程序：Java，Ruby，Python和C ++。 MapReduce程序本质上是并行的，因此对于使用群集中的多台机器执行大规模数据分析非常有用。

MapReduce程序分两个阶段工作：

- Map阶段
- Reduce阶段

每个阶段的输入都是**key-value**对。 另外，每个程序员都需要指定两个函数：map函数和reduce函数。

## MapReduce的工作方式

让我们用一个例子来理解MapReduce的工作方式。考虑你的MapReduce程序有以下输入数据（示例数据来自[这里](https://www.guru99.com/introduction-to-mapreduce.html)）：

```
Welcome to Hadoop Class
Hadoop is good
Hadoop is bad
```

需要经过MapReduce以下几个步骤的处理：

![https://cdn.guru99.com/images/Big_Data/061114_0930_Introductio1.png](https://cdn.guru99.com/images/Big_Data/061114_0930_Introductio1.png)

MapReduce任务的最终输出是：

| 单词     | 数量  |
| ------- | ---- |
| bad     | 1    |
| Class   | 1    |
| good    | 1    |
| Hadoop  | 3    |
| is      | 2    |
| to      | 1    |
| Welcome | 1    |

数据经历了以下几个阶段：

**Input Splits**

对MapReduce作业的输入分为固定大小的片段，称为**Input Splits**。**Input Splits**是由单个map消费的输入块。

**Mapping**

这是执行map-reduce程序的第一个阶段。 在这个阶段中，每个分割中的数据被传递给一个mapping 函数以产生输出值。 在我们的例子中，映射阶段的工作是计算来自输入分割的每个词的出现次数，并且提供`<word, frequency>`形式的列表。

**Shuffling**

此阶段消费Mapping阶段的输出。 其任务是整合Mapping阶段输出的相关记录。 在我们的例子中，同样的词汇与各自的频率一起组合在一起。

此阶段除了进行Shuffling操作还可以进行sorting操作。

**Reducing**

在这个阶段，会汇总来自Shuffling阶段的输出值。 该阶段结合Shuffling阶段的值并返回单个输出值。 总之，这个阶段得出了完整的数据集。

# HDFS分布式文件系统

分以下六个部分说明。

## HDFS的定义

HDFS（Hadoop Distributed FileSystem）是Apache Software Foundation项目和Apache Hadoop项目的子项目。 Hadoop非常适合存储大量数据（如TB和PB），并使用HDFS作为其存储系统。 你可以通过HDFS连接到数据文件分发集群中的任意节点。 然后可以像一个无缝的文件系统一样访问和存储数据文件。 访问数据文件是以流式方式处理的，这意味着应用程序或命令可以直接使用MapReduce处理模型执行。

## HDFS的应用接口

您可以通过许多不同的方式访问HDFS。 HDFS为Java API提供本地Java应用程序编程接口（API）和对java api封装的本地C语言包装器。 另外，您可以使用Web浏览器浏览HDFS文件。下表是可以与HDFS接口的应用程序：

| 应用                      | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| FileSystem (FS) shell     | 类似于常见Linux和UNIX shell（bash，csh等）的命令行界面，允许与HDFS数据交互。 |
| DFSAdmin                  | 可以用来管理HDFS群集的命令集。                               |
| fsck                      | Hadoop命令的子命令。 可以使用fsck命令检查文件是否存在不一致，如缺少块，但不能使用fsck命令纠正这些不一致。 |
| Name nodes and data nodes | 内置Web服务器可让管理员检查群集的当前状态。                  |

由于其简单而强大的体系结构，HDFS具有非凡的功能集和高期望值。

## HDFS架构

HDFS由文件和目录所在节点的互连集群组成。 HDFS群集包含一个称为NameNode的单个节点，该节点管理文件系统命名空间并管理客户端对文件的访问。 另外，DataNode将数据作为块存储在文件中。

在HDFS中，NameNode节点管理文件系统命名空间操作，如打开，关闭和重命名文件和目录。 NameNode还将数据块映射到DataNode，DataNode处理来自HDFS客户端的读取和写入请求。 DataNode还根据NameNode的指示信息创建，删除和复制数据块。

HDFS架构图如下：

![https://www.ibm.com/developerworks/library/wa-introhdfs/fig1.gif](https://www.ibm.com/developerworks/library/wa-introhdfs/fig1.gif)

每个群集都包含一个NameNode。 这种设计方便了管理每个命名空间和判断数据分配的简化模型。

**NameNode和DataNode之间的关系：**

NameNode和DataNode是用于在异构操作系统上的商用机器上以分离方式运行的软件组件。 HDFS是使用Java编程语言构建的；因此，任何支持Java编程语言的机器都可以运行HDFS。 典型的安装集群有一台运行NameNode的专用机器，这台机器上也可能有一个DataNode。 集群中的其他机器每台都运行一个数据节点。

DataNode不断循环的向NameNode询问指令。NameNode不能直接连接到DataNode；它只是返回来自DataNode调用的函数的值。 每个DataNode维护一个开放的服务器套接字，以便客户端代码或其他DataNode可以读取或写入数据。 NameNode持有该服务器套接字的主机或端口，该NameNode将信息提供给感兴趣的客户端或其他数据节点。

NameNode维护并管理对文件系统命名空间的更改。

> 文件系统命名空间（File system namespace）：
>
> HDFS支持传统的分层文件组织，用户或应用程序可以在其中创建目录并存储文件。 文件系统命名空间层次与大多数其他现有文件系统类似; 您可以创建，重命名，重定位和删除文件。

## 数据复制

数据复制：Data replication

HDFS复制文件块以实现容错。 应用程序可以指定文件在创建时的副本数量，并且此后可以随时更改此数字。 NameNode会做出关于块复制的所有决定。

HDFS使用智能副本放置模型来提高可靠性和性能。 优化的副本放置功能使得HDFS独特于大多数其他分布式文件系统。

大型HDFS环境通常在多台计算机上安装。 不同机器上的两个数据节点之间的通信通常比同一机器上的数据节点慢。 因此，NameNode会尝试优化数据节点之间的通信。

## 数据组织方式

HDFS的一个主要目标是支持大文件。 一个典型的HDFS块的大小是64MB（HDFS默认数据块大小为64MB（最小化寻址开销），小于一个块大小的文件不会占据整个块的空间）。 因此，每个HDFS文件都由一个或多个64MB块组成。 HDFS会尝试将每个块放置在单独的数据节点上。

## 数据存储可靠性

HDFS的一个重要目标是可靠地存储数据，即使在NameNode、DataNode或者网络分区内出现故障时也是如此。

检测是HDFS克服故障的第一步。 HDFS使用心跳消息来检测NameNode和DataNode之间的连接。

HDFS采用以下手段保证数据存储的可靠性：

- HDFS心跳：HDFS heartbeats
- 数据块重新平衡：Data block rebalancing
- 数据的完整性：Data integrity，通过事务
- 同步元数据更新：Synchronous metadata updating
- 用户，文件和目录的HDFS权限：HDFS permissions
- 快照：Snapshots

# 总结

- 了解hadoop的生态：整体环境
- 理解MapReduce模型：计算过程
- 熟悉HDFS分布式文件系统：存储方式

了解以上3个部分之后就可以去搭建hadoop环境，开发MapReduce应用。

---

参考：

- https://www.guru99.com/introduction-to-mapreduce.html
- https://www.ibm.com/developerworks/library/wa-introhdfs/