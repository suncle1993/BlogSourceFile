---
categories:
  - 数据库
date: '2016-03-21T10:08:50'
description: ''
tags:
  - ASM
  - 管理
  - Oracle
title: 自动存储管理ASM
---



参考资料：

[https://docs.oracle.com/cd/B19306_01/server.102/b14231/storeman.htm#ADMIN036](https://docs.oracle.com/cd/B19306_01/server.102/b14231/storeman.htm#ADMIN036)

## 什么是ASM？

ASM是Automatic Storage Management（自动存储管理）的缩写。ASM是一个集成的高性能的文件系统和卷管理器。Oracle将所有的存储分为disk groups，我们只需要管理这些disk groups，而不用去管具体的数据文件。

In the SQL statements that you use for creating database structures such as tablespaces, control files, and redo and archive log files, you specify file location in terms of disk groups. ASM then creates and manages the associated underlying files（底层文件） for you.

## 为什么使用ASM？

- 提供高效率的存储管理
- 提供完整的集群文件系统和卷管理能力

**ASM的优点：**

**Mirroring and Striping**（镜像化和条带化）

条带化是一种用于在多个磁盘驱动器之间分散数据的技术。一个大的数据段被分为较小的单元，这些单元分布在可用设备之间。分隔数据的单元称为“数据单元大小”或“条带大小”，是指向每个磁盘写入这些条带的大小。可以同时读写的并行条带数量称为“条带宽度”。分条可以加快从磁盘存储中获取数据的操作，这是因为它扩展了总I/O带宽的能力。这样就优化了性能和磁盘利用率，从而不再需要手动I/O 性能调优。

ASM镜像化选项:

| Mirroring Option | Description                              |
| ---------------- | ---------------------------------------- |
| 2-way mirroring  | Each extent has 1 mirrored copy.         |
| 3-way mirroring  | Each extent has 2 mirrored copies.       |
| Unprotected      | ASM provides no mirroring. Used when mirroring is provided by the disk subsystem itself. |

<!--more-->

**Dynamic Storage Configuration**（动态存储配置）

可以在数据库运行时更改数据库的配置，ASM会自动Rebalance。

**ASM Instance**（Oracle实例）

ASM实例是一种Oracle实例，它为磁盘组、ADVM(ASM动态卷)和ACFS(ASM集群文件系统)管理元数据。所有元数据修改都是由ASM实例完成的，以隔离故障。数据库实例连接到一个ASM实例，以创建、删除、打开、关闭文件或者改变其大小，数据库实例直接读写由ASM实例管理的磁盘。Oracle在内部使用自动内存管理，很少需要对一个Oracle ASM实例进行调优。

**Interoperability with Existing Databases**

已存在数据库的互操作性。ASM并不会消除已存在系统的功能。

**Single Instance and Clustered Environments**

ASM支持单实例和集群环境。

## ASM的Components 

ASM的五项组成：disk groups, disks, failure groups, files, and templates。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-ASM%E7%BB%84%E4%BB%B6%E7%9A%84%E5%85%B3%E7%B3%BB.png)

### disk groups

ASM的首要组成就是disk groups，一组disk作为一个单元构成disk groups。

三种disk group类型对应的镜像选择：

| Disk Group Type     | Supported Mirroring Levels   | Default Mirroring Level |
| ------------------- | ---------------------------- | ----------------------- |
| Normal redundancy   | 2-way3-wayUnprotected (none) | 2-way                   |
| High redundancy     | 3-way                        | 3-way                   |
| External redundancy | Unprotected (none)           | Unprotected             |

### disks

在windows操作系统上，disk可能是一个分区（partition），在其他的平台上可能是：

- A partition of a logical unit number (LUN)
- A network-attached file

### failure groups

故障组定义了一些ASM磁盘，它们可能共用一种潜在的故障装置。故障组是磁盘组中的一个磁盘子集，这个子集内的磁盘依赖于一个必须容忍其故障的公共硬件资源。只有对于普通冗余（Normal redundancy）或高冗余（High redundancy）配置，它才非常重要。相同数据的冗余副本被放置在不同的故障组中。

### files

写到ASM磁盘中的文件称为ASM文件。每个ASM文件都完全包含在单个磁盘组中，平均分布在这个组中的所有ASM磁盘上。一个ASM文件就是一个数据盘区集，每个数据盘区是分配单元的一个集合。由于在ASM文件变大时Oracle会自动增大数据盘区的大小，因此我们不能改变数据盘区的大小。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-%E6%95%B0%E6%8D%AE%E5%9C%A8%E7%A3%81%E7%9B%98%E4%B8%AD%E7%9A%84%E5%88%86%E5%B8%83%E7%AE%A1%E7%90%86.png)

### templates

Templates是文件属性值的集合。用来给每一种类型的数据库文件设置镜像化和条带化的属性的。

关于templates：

[Managing Disk Group Templates](https://docs.oracle.com/cd/B19306_01/server.102/b14231/storeman.htm#i1019485)

## ASM架构

ASM支持单实例架构和集群架构。

在一个数据库服务器中，可以存在多个数据库实例，一个数据库实例可以对应一个ASM实例，也可以多个数据库实例对应一个ASM实例。单实例架构如下图：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-ASM%E5%8D%95%E5%AE%9E%E4%BE%8B%E6%9E%B6%E6%9E%84.png)

ASM集群架构如下：多个ASM实例共同管理数据文件。数据库实例和ASM实例最常常见的还是一对一。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-ASM%E7%9A%84RAC%E6%9E%B6%E6%9E%84.png)



## Rebalance

- ASM rebalance 操作不会影响数据库的正常使用
  - 会影响I/O效率
- 能通过数据的重新分布，使得系统的I/O得到最大的提升
- 从Oracle 10R2之后，如果关闭数据库实例，Rebalance操作会更快。