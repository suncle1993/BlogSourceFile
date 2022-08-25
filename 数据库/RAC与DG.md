---
categories:
  - 数据库
date: '2016-03-16T10:35:05'
description: ''
tags:
  - Oracle
  - RAC
  - DG
title: RAC与DG
---


## RAC

RAC: real application clustersrac	

RAC: real application clustersrac	

单节点数据库：数据文件和示例文件一一对应

> 实例损坏时数据库就损坏了

RAC架构数据库：数据文件和多个实例对应

> RAC最根本的初衷是实例级的容错，并不是基于数据的
>
> 实例都是基于数据的。
>
> dataguard是基于数据容错的。
>
> Oracle数据库支持网格计算环境的核心技术
>
> SAN网络存储（Storage Area Network）：集中式管理的高速存储网络
> ![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-RAC%E6%9E%B6%E6%9E%84.png)



RAC的目的

- 提供实例级别的冗余
- 提供更多的系统资源
- 增加更多的并行处理

RAC的优点和缺点

  优点

- 提供系统冗余

- 更多的系统资源

- 业务分割处理

  缺点

- 内存共享和资源竞争（cache fusion）

- 底层技术复杂，对DBA技术要求高

什么时候需要使用RAC？

- 实例冗余——第一考虑的目的
- 处理能力和性能的提升

<!--more-->

## DG

DataGuard，数据卫士，一种数据库级别的高可用性（HA）方案，用作数据[容灾](http://baike.baidu.com/view/1088749.htm)解决方案。对于联机事务处理（OLTP，数据量不太大）非常合适，对于联机分析处理（OLAP，数据量太大），只能选择关键数据创建DG，常规数据，选择其他方式备份。



容灾级别的DG：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-%E5%AE%B9%E7%81%BE%E7%BA%A7%E5%88%AB%E7%9A%84DG.png)

本地，同城，异地，多种容灾，创建很多standby



***DataGuard的保护模式***

**最大保护模式**

最安全的模式，这种模式主备库之间数据是同步的。即主库提交的同时，备库会做相应的恢复。最大限度的保证了[数据完整性](http://baike.baidu.com/view/702953.htm)。不允许数据的丢失。

如果主备库之间网络，或者备库出现问题会直接影响主库操作。导致主库宕机。因此一般不会选择最大保护模式。

**最大性能模式**

这种模式保证主库性能最大化，主备库之间数据是[异步传输](http://baike.baidu.com/view/817251.htm)的。即，主备日志归档以后才会传输到备用库，在备库上使用归档日志文件做恢复操作。

**最高可用性模式**

这种模式和"最大保护"基本上差不多。正常情况下，主备库之间是同步的。

当网络或者备库出现问题时，不会影响到主库的宕机，主库会自动转换到"最大性能"模式，等待备库可用时，将归档传输到备库做恢复。

可以把这种模式理解为"最大保护"和"最大性能"两种模式的中间体。



***如何选择DG的保护模式***

影响DG保护模式选择的最大因素就是网络质量，如果网络质量比较好，比如本地的局域网，则可以选择最高可用模式。如果网络质量一般，则选择最大性能模式。一般不会选择最大保护模式，最大保护模式损害了系统的可用性。



***DG中standby数据库的类型***

**物理standby数据库：physical standby databases**

物理Standby与Primary数据库完全一模一样，在物理数据库磁盘上具有主库相同架构的块，**通过REDO应用（属于块对块的应用）来维护物理Standby数据库**。

**逻辑standby数据库：logical standby databases**

逻辑Standby也要通过Primary数据库（或其备份，或其复制库，如物理Standby）创建，因此在创建之初与物理Standby数据库类似。不过由于**逻辑Standby通过SQL应用的方式应用REDO数据**，因此逻辑Standby的物理文件结构，甚至数据的逻辑结构都可以与Primary不一致。



***附：***

关于Oracle11gR2 之  DataGuard_03   三种保护模式的探索可见下面这篇blog

[探索Oracle11gR2 之  DataGuard_03   三种保护模式](https://blog.csdn.net/wuweilong/article/details/9989785)