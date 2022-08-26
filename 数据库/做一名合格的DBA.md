---
abbrlink: 2680367260
alias: 2016/03/16/做一名合格的DBA/index.html
categories:
- 数据库
date: '2016-03-16T10:59:00'
description: ''
tags:
- DBA
- Oracle
title: 做一名合格的DBA
---







------

***Oracle DBA的角色定义***

**开发型DBA**

- 数据库安装
- 数据库架构设计（架构和建模）
- 代码开发（存储过程，SQL）

**运维型DBA**

- 数据库日常监控
- 故障处理
- 性能优化
- 数据备份，容灾
- 数据库安全规划

***DBA的操守***

**在自己的责任范围内**

- 让数据库设计更合理，预防设计导致的性能或安全隐患
- 数据更安全
- 数据库性能更优
- 数据库日常管理更合理
- 故障发现，处理及时

***数据库的架构设计***

数据库架构

- 分布or单库

实例的冗余

- RAC or single

数据库的安全和容灾

- DG or streams or Rman

空间的考虑存储的规划

- ASM （自动存储管理）+ SAN（Storage Area Network，SAN网络存储）

软件的生命周期和业务（数据）增长的预测

***数据库的建模***

实体，关系的设计E-R

- 表
- 索引
- 主、外键的引用

***数据库的开发--SQL和存储过程***

- SQL是否绑定变量
- SQL语句的性能问题
- 表的分析的方式-分析选项，分析频率等...
- 影响SQL执行效率的性能参数



<!--more-->

***数据库的运维***

数据库的监控

- 定制
- 开源软件+脚本
- OEM+grid control
- 第三方软件Quest等

数据的管理及安全

- 备份策略
- 数据的保留，删除策略
- 分区，压缩，只读表空间...


- 最首要的问题是数据的安全问题

故障处理

- 对数据库的深入理解
- Oracle support（metalink，metalinkOracle的官方技术支持站点Oracle公司通过该网站来支持全球的客户）
- asktom.oracle.com
- www.itpub.net

数据库的性能优化

- 对业务流程的深入理解
- 用户感知为导向的优化思路
- 性能基线的建立-cpu,i/o,sessions...
- 定期的AWR（Automatic Workload Repository，Oracle AWR）报告分析

数据库的安全

- 口令管理策略（精细化的授权机制）
- Oracle的安全产品DB vault和Audit vault
- Oracle的细粒度审计（FGA）
- 数据加密
- 操作系统口令管理

***Oracle的学习思路***

入门->深入Oracle->Oracle性能优化艺术->基于海量数据的数据库设计