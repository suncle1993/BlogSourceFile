---
categories:
  - 数据库
date: '2016-03-16T10:23:40'
description: ''
tags:
  - 数据字典
  - Oracle
title: Oracle数据字典
---




**数据字典的组成——两类视图**

静态数据字典：描述数据库的信息

- 这些数据经常是静止的。

动态数据字典：描述实例的信息

- 反映数据局运行的状态，反映数据库实例运行的信息，这些信息经常是变化的。

**users**

ALL_USERS--lists all users of the database visible to the current user. This view does not describe the users 

- 描述不是用户自己，而是当前用户可见（也就是有权访问）的数据库的所有的用户。

DBA_USERS--describes all users of the database

USER_USERS--describes the current user



**数据字典视图**

查询所有的数据字典视图

```
select * from dict;
```

数据字典的基表

- 是保存数据的真正的表
- 数据字典视图的数据来自于基表
- Oracle不对基表做支持和解释

**DBA常用的一些数据字典视图——静态视图**

**user_tables**

内容和ALL_TABLES类似。

[https://docs.oracle.com/cd/B19306_01/server.102/b14237/statviews_2105.htm#REFRN20286](https://docs.oracle.com/cd/B19306_01/server.102/b14237/statviews_2105.htm#REFRN20286)

**user_tab_partitions**

内容和ALL_TAB_PARTITIONS类似

[https://docs.oracle.com/cd/B19306_01/server.102/b14237/statviews_2098.htm#i1591118](https://docs.oracle.com/cd/B19306_01/server.102/b14237/statviews_2098.htm#i1591118)



<!--more-->

附：

[Oracle数据字典详解](http://www.360doc.com/content/14/1114/11/17440478_425032377.shtml)