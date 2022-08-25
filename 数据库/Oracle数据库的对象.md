---
categories:
  - 数据库
date: '2016-02-19T17:04:15'
description: ''
tags:
  - Oracle
  - 对象
title: Oracle数据库的对象
---



查看oracle数据库中的所有对象

```
select distinct object_type from dba_objects; /*distinct??*/
```

> dba_objects是存放数据库对象的一个视图



**schema**：数据库中一个对象的合集称为一个schema，它的名字和拥有这些对象的用户名相同。---比如scott用户和它下面的表统一称为一个schema

下面分别介绍一下Oracle数据库中的各个对象

## 1、表

### 表-段（segment）

段是表物理化的过程，在Oracle数据库里只要是分配了存储空间的对象，都可以叫做段。

CLOB是内置类型，它将字符大对象 (Character Large Object) 存储为数据库表某一行中的一个列值。

### 表-分区（partition）

便于对表的管理。

对分区的具体操作如下图：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-Oracle%E8%A1%A8%E5%88%86%E5%8C%BA%E7%9A%84%E6%93%8D%E4%BD%9C.jpg)


<!--more-->



## 2、索引

- 目的--用于加快数据的访问
- 缺点：占据额外空间，影响DML操作的效率

> 对数据库增删改查的操作都需要在索引中多执行一次

### 创建索引

创建索引命令如下：

```
create index idx_student on student(id);
```

为student表的id列创建名为idx_student的索引

### 索引的种类

按数据的组织方式分类

- B-tree B树索引 （有利于资源的节约利用）
- Bitmap 位图索引 （对于重复次数很多的数据专门建立的索引）
- Text    全文索引 （上述索引方式不好用时可以采用全文索引）

## 3、视图-view

只是一句SQL代码，并不占用内存空间

**物化视图**

- 将查询的结果集保存下来，用于后续的查询，提高查询效率
- 和普通的视图不同，物化视图是一个段对象，占用物理空间
- 提高查询效率，可以用于数据复制

## 4、sequence

- 为业务提供一个序列号
- 唯一但不保证连续

## 5、同义词-SYNONYM

- 提供对象的一个别名
- 使不同用户下对象的引用变得方便
- [https://www.cnblogs.com/kerrycode/archive/2012/12/19/2824963.html](https://www.cnblogs.com/kerrycode/archive/2012/12/19/2824963.html)

## 6、数据库链-database link 

- 用于数据库之间的数据访问和操作
- 由oracle保证数据访问和操作的事务性
- [https://www.cnblogs.com/sumsen/archive/2013/03/04/2943471.html](https://www.cnblogs.com/sumsen/archive/2013/03/04/2943471.html)

## 7、表空间
逻辑存储对象

## 8、重做日志-Redo
见视频
## 9、undo
见视频