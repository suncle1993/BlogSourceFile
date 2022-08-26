---
abbrlink: 1499000154
alias: 2019/08/02/Comparison-of-the-Open-Source-OLAP-Systems-for-BigData/index.html
categories:
- 大数据
date: '2019-08-02T17:05:54'
description: ''
tags:
- Aliyun
- BigData
title: 大数据OLAP系统比较
---









大数据OLAP系统比较

# 结论

选择presto和clickhouse配合使用

- 对实时性要求不严格的数据用presto查询

- 对于实时性有要求的数据查询clickhouse

理由：

1. 核心原因：clickhouse相对于Apache Kylin等预计算方案非常省机器，成本最关键(比较穷，没办法)
2. clickhouse的单表查询非常非常快
3. 目前再惠的数据仍然处于并将长期处于小规模阶段(集群内存少于1T，Cpu少于200vCore)，clickhouse在小规模集群上表现优于Druid和Pinot
4. presto的综合性能好，在join操作时表现较好，保持目前数仓的这一套不变

<!--more-->

# OLAP整体情况

1. 目前的大数据OLAP系统都是部分优化的，偏向于定制化系统，典型的是Clickhouse的不同表级engine
2. 建立一个能够cover绝大多数情况的通用的大数据OLAP系统预计还需要100年
3. 目前所有的OLAP系统都是基于两种思路设计
   1. 列式数据库加索引，典型是Clickhouse
   2. 预计算空间换时间：典型是Apache Kylin，所有结果预先计算好放在cube

# OLAP系统比较

先大致按照OLAP的设计思路把常用的系统分下类：

**列式数据库加索引**

- Clickhouse
- Apache Pinot
- Druid

**预计算空间换时间**

- Apache Kylin
- Apache Doris
- Mondrian

从所有的系统中选出相对符合的再进行深入一点的比较如下：

|            | Clickhouse | Druid              | Apache Kylin |
| ---------- | ---------- | ------------------ | ------------ |
| 语言       | C++        | Java               | Java         |
| Star       | 7743       | 8405               | 2275         |
| 活跃度     | 活跃       | 活跃               | 活跃         |
| 亚秒级响应 | √          | √                  | √            |
| 列式数据库 | √          | √                  | ×            |
| 预计算     | ×          | √                  | √            |
| 云存储     | ×          | √                  | √            |
| 机器       | 省         | 省                 | 费           |
| join       | √          | ×                  | √            |
| 商业公司   | Altinity   | Imply和Hortonworks | Kyligence    |

至于clickhouse/druid/pinot三者的比较可以参见这篇文章：[Comparison of the Open Source OLAP Systems for Big Data: ClickHouse, Druid, and Pinot](https://medium.com/@leventov/comparison-of-the-open-source-olap-systems-for-big-data-clickhouse-druid-and-pinot-8e042a5ed1c7)，整体写的非常好而且有深度，对比表格翻译如下：

| ClickHouse                                                   | Druid/Pinot                                  |
| :----------------------------------------------------------- | :------------------------------------------- |
| 具备C++经验的组织                                            | 具备Java经验的组织                           |
| 小型集群                                                     | 大型集群                                     |
| 少量表                                                       | 大量表                                       |
| 单一数据集                                                   | 多个不相关的数据集（多租户）                 |
| 表和数据集永久驻留在集群中                                   | 表和数据集定期出现并从群集中退出             |
| 表格大小（以及它们的查询强度）在时间上是稳定的               | 表格随时间热度降低                           |
| 查询的同质性（其类型，大小，按时间分布等）                   | 异质性                                       |
| 存在可以用于分区的维度，且经过该维度分区后，几乎不会触发跨分区的数据查询 | 没有这样的维度，查询经常触及整个集群中的数据 |
| 不使用云，集群部署在特定的物理服务器上                       | 群集部署在云中                               |
| 无需依赖现有的Hadoop或Spark集群                              | Hadoop或Spark的集群已经存在并且可以使用      |

ClickHouse，Druid和Pinot三个系统都还不成熟。在这三个系统中，ClickHouse与Druid和Pinot略有不同，而后两者几乎完全相同，它们几乎是两个独立开发的完全相同系统的实现。

与ClickHouse相比，Druid和Pinot更适合优化大型集群的基础架构成本，并且更适合云环境。

Druid和Pinot之间唯一可持续的区别是，Pinot依赖于Helix框架并将继续依赖ZooKeeper，而德鲁伊可能会远离对ZooKeeper的依赖。另一方面，德鲁伊安装将继续依赖于某些SQL数据库的存在。

# CloudFlare的选择

在CK和Druid中选择了CK，10个节点的规模CK更好

- https://blog.cloudflare.com/how-cloudflare-analyzes-1m-dns-queries-per-second/

目前公司最大的单表数据为10B，也就是100亿：门店3公里内的所有门店以及经纬度数据，占用S3空间110G

# 各个引擎的概况

## Clickhouse

项目地址：https://github.com/yandex/ClickHouse

架构概述：https://clickhouse.yandex/docs/zh/development/architecture/

支持primary key sorting，不支持inverted indexes

https://github.com/yandex/ClickHouse/issues/5125

## Druid

不支持primary key sorting，支持inverted indexes

通过编写 Json 文件，以 HTTP 的方式请求 Druid

支持sql

国内使用Druid比较多，有赞，美团等

https://tech.youzan.com/realtime-olap-on-druid/

https://github.com/apache/incubator-druid

Java star 8405

## Apache Pinot

项目地址：https://github.com/apache/incubator-pinot/

架构概述：https://pinot.readthedocs.io/en/latest/architecture.html

2323 star 活跃

国内使用Pinot的比较少

## Apache Doris

Doris前身是Palo，Palo是百度自研的基于MPP的交互式SQL数据仓库

架构概述：https://doris.incubator.apache.org/Docs/cn/internal/metadata-design.html#id3

项目地址：https://github.com/apache/incubator-doris

1294 star 活跃

C++

来源百度

国内使用Pinot的比较少

## Apache Kylin

https://github.com/apache/kylin

来源：eBay

语言：Java

2275 star 活跃

在国内广泛使用

链家使用Kylin：https://www.infoq.cn/article/lianjia-data-analysis-apache-kylin

典型的空间换时间：维度优化，预计算的结果需要存储到 Hbase

优势：

- 都已经预先计算好了，性能啥的都不会有啥问题
- 主要针对hive的离线数据做分析，属于hadoop生态圈，可以和目前的hive这一套完美结合起来
- Apache Kylin v1.6.0之后支持了近实时的流计算，后续构建成为离线和实时的一站式解决方案
- Apache Kylin v2.0.0 引入了一个全新的基于 Apache Spark 的构建引擎，替换MR，Cube构件时间缩短一半

代价：

1. 需要维护一套hbase集群，空间换时间的操作会极度废机器，但是hbase数据可以存在cloud上
2. 需要在kylin web维护针对查询提前定义维度构建cube
3. 运维Kylin对Admin有较高的要求，首先必须了解HBase，Hive，MapReduce，Spark，HDFS，Yarn的原理；其次对MapReduce Job和Spark Job的问题排查和调优经验要丰富；然后必须掌握对Cube复杂调优的方法；最后出现问题时排查的链路较长，复杂度较高。

Apache kylin中cube的构建过程及原理分析：https://www.cnblogs.com/shibit/p/7039794.html

## Mondrian

https://github.com/pentaho/mondrian

https://blog.csdn.net/ZYC88888/article/details/80311014

792 star，不活跃，性能一般(曹总说)

Java

Mondrian不是一个真正的OLAP数据库，是一个基于关系数据库的分析服务器

查询瓶颈仍然在底层的存储层的查询效率，只是对于动态多维度分析做了优化

通过xml而不是sql查询

# OLAP in zaihui

现状：

- Clickhouse的单表查询速度确实非常非常快，在会员数据这部分的表现非常好

- 通过删表重建的方式处理数据重复不太优雅，对于百亿数据不太现实
- 数据从hive同步到Clickhouse的时间较长，目前是单线程后续可以改成spark等形式

后续可以做的工作：

1. 开发一套clickhouse集群的管理包括扩容等等的自动化系统
2. 开发一套从hive/spark等同步数据到clickhouse的高效服务

---

参考：

- [Comparison of the Open Source OLAP Systems for Big Data: ClickHouse, Druid, and Pinot](https://medium.com/@leventov/comparison-of-the-open-source-olap-systems-for-big-data-clickhouse-druid-and-pinot-8e042a5ed1c7)
- https://www.sspaeti.com/blog/olap-whats-coming-next/
- [BigQuery under the hood](https://cloud.google.com/blog/products/gcp/bigquery-under-the-hood)
- [主流OLAP系统对比总结](https://zhuanlan.zhihu.com/p/38767561)
- https://zhuanlan.zhihu.com/p/51555789
- https://github.com/yandex/ClickHouse/issues/1178
