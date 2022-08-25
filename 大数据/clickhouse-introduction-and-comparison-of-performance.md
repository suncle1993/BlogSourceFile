---
categories:
  - 大数据
date: '2019-07-27T14:13:26'
description: ''
tags:
  - clickhouse
  - 性能
title: Clickhouse简介和性能对比
---




# Clickhouse简介和性能对比

ClickHouse是一个用于联机分析(OLAP)的列式数据库管理系统(DBMS)。

常见的列式数据库有： Vertica、 Paraccel (Actian Matrix，Amazon Redshift)、 Sybase IQ、 Exasol、 Infobright、 InfiniDB、 MonetDB (VectorWise， Actian Vector)、 LucidDB、 SAP HANA、 Google Dremel、 Google PowerDrill、 Druid、 kdb+。

<!--more-->

不同的存储方式适合不同的场景，这里的查询场景包括： 

- 进行了哪些查询
- 多久查询一次
- 各类查询的比例
- 每种查询读取多少数据————行、列和字节
- 读取数据和写入数据之间的关系
- 使用的数据集大小以及如何使用本地的数据集
- 是否使用事务,以及它们是如何进行隔离的
- 数据的复制机制与数据的完整性要求
- 每种类型的查询要求的延迟与吞吐量

系统负载越高，根据使用场景进行定制化就越重要，并且定制将会变的越精细。没有一个系统同样适用于明显不同的场景。如果系统适用于广泛的场景，在负载高的情况下，所有的场景可以会被公平但低效处理，或者高效处理一小部分场景。

# OLAP场景的关键特征

* 大多数是读请求
* 数据总是以相当大的批(> 1000 rows)进行写入
* 不修改已添加的数据
* 每次查询都从数据库中读取大量的行，但是同时又仅需要少量的列
* 宽表，即每个表包含着大量的列
* 较少的查询(通常每台服务器每秒数百个查询或更少)
* 对于简单查询，允许延迟大约50毫秒
* 列中的数据相对较小： 数字和短字符串(例如，每个URL 60个字节)
* 处理单个查询时需要高吞吐量（每个服务器每秒高达数十亿行）
* 事务不是必须的
* 对数据一致性要求低
* 每一个查询除了一个大表外都很小
* 查询结果明显小于源数据，换句话说，数据被过滤或聚合后能够被盛放在单台服务器的内存中

# Clickhouse优缺点

优点

- 数据压缩
- 多核并行处理
- 支持数据复制和数据完整性
  - shard分片
  - replica副本
- 多服务器分布式处理。其他列式数据库管理系统中，几乎没有一个支持分布式的查询处理
- 支持sql
  - 大部分情况下是与SQL标准兼容的。 支持的查询包括 GROUP BY，ORDER BY，IN，JOIN以及非相关子查询。 不支持窗口函数和相关子查询。
- 向量引擎
- 实时数据插入
- 稀疏索引
- 适合在线查询

缺点

- 没有完整的事务支持。
- 缺少高频率，低延迟的修改或删除已存在数据的能力。仅能用于批量删除或修改数据，但这符合 [GDPR](https://gdpr-info.eu/)。
- 稀疏索引使得ClickHouse不适合通过其键检索单行的点查询。

# 性能对比

官方的性能测试对比报告参见：https://clickhouse.yandex/benchmark.html

知乎上的一篇OLAP引擎比较：https://zhuanlan.zhihu.com/p/54907288

在一张有44个字段的大表中做单表查询并且和Amazon RedShift做对比，结果如下：

Clickhouse 测试环境：单CPU 2核 4G内存
```
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l  # # 查看物理CPU个数
cat /proc/cpuinfo| grep "cpu cores"| uniq  # 查看每个物理CPU中core的个数(即核数)
```
数据量：1580万

clickhouse engine和index：

```
ENGINE = ReplacingMergeTree(insert_time)
order by (membership_uid, business_group_uid, calendar_date, insert_time);
```
Sql执行速度取决于：执行时间execution和数据拉取时间fetching

Clickhouse:

```sql
select * from dm.delphi_membership_properties t where t.membership_id=666; -- 160ms
select * from dm.delphi_membership_properties t where t.membership_uid='3ec723abeffc470ea42593f0d1e9d279'; -- 120ms
select count(*) from dm.delphi_membership_properties t where t.business_group_id=44; --  120ms
select account_phone from dm.delphi_membership_properties t where t.membership_id=666; -- 58ms
select account_phone from dm.delphi_membership_properties t where t.membership_uid='3ec723abeffc470ea42593f0d1e9d279'; -- 44 ms
select account_phone from dm.delphi_membership_properties t where t.business_group_id=44; -- 190ms
```

RedShift: 机器配置高于clickhouse单机数倍

```sql
select * from dm.delphi_membership_properties t where t.business_group_id=44; --  1286条数据一次取出来时间较长
select * from dm.delphi_membership_properties t where t.business_group_uid='7a68b4a7350c4d7288e8befef91f8581'; -- 1286条数据一次取出来时间较长
select count(*) from dm.delphi_membership_properties t where t.business_group_uid='fa26f456030940b8b6ec4b56e256aee2'; -- 250ms
select account_phone from dm.delphi_membership_properties t where t.business_group_uid='7a68b4a7350c4d7288e8befef91f8581'; -- 360ms
select count(distinct t.membership_uid) from dm.delphi_membership_properties t where t.business_group_uid='fa26f456030940b8b6ec4b56e256aee2'; -- 450ms
```

将clickhouse 表engine的order by key修改如下：颗粒度小的在后
```
ENGINE = ReplacingMergeTree(insert_time)
order by (business_group_uid, calendar_date, created_at, insert_time, membership_uid);
```
修改order by key之后含有bguid的查询速度大幅提升，均低于60ms。这个速度提升主要是clickhouse的稀疏索引导致的，关于索引会在其他文章中介绍到。
