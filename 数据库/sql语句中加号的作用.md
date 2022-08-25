---
categories:
  - 数据库
date: '2016-07-08T01:20:23'
description: ''
tags:
  - sql
  - 表连接
title: sql语句中(+)的作用
---



## 演示示例

说明：以下示例中，表a是员工表，有a,b,c,d四个员工，性别都是男性m。表b是工资表，有a,b,d四个员工，工资对应的是1000，2000，4000。然后分别演示带(+)符号的和不带(+)符号的，结果如下。

```sql
SQL> select * from a;

NAME                 SEX
-------------------- -----
a                    m
b                    m
c                    m
d                    m

SQL> select * from b;

NAME                      MONEY
-------------------- ----------
a                          1000
b                          2000
d                          4000

SQL> select a.name,b.money from a,b where a.name=b.name(+);

NAME                      MONEY
-------------------- ----------
a                          1000
b                          2000
d                          4000
c

SQL> select a.name,b.money from a,b where a.name=b.name;

NAME                      MONEY
-------------------- ----------
a                          1000
b                          2000
d                          4000
```

可见，带(+)号时，a表中的所有人都在，即使工资为空。不带(+)时，a表中的没有出现工资为空的员工c。

<!--more-->

## 对(+)号的解释

**(+) 表示外连接。**条件关联时，一般只列出表中满足连接条件的数据。如果条件的一边出现（+），则另一边的表就是主表，主表中的所有记录都会出现，即使附表中有的记录为空

## (+)的扩展：SQL表连接

### SQL表连接分类

内连接，外连接，交叉连接，其中外连接包括左连接和右连接。

### SQL表连接示例

**内连接**

```sql
SQL> select a.name,b.money from a,b where a.name=b.name;

SQL> select a.name,b.money from a inner join b on a.name=b.name;
```

**左连接**

```sql
SQL> select a.name,b.money from a,b where a.name=b.name(+);

SQL> select a.name,b.money from a left join b on a.name=b.name;
```

**右连接**

```sql
SQL> select a.name,b.money from a right join b on a.name=b.name;

SQL> select a.name,b.money from a,b where a.name(+)=b.name;
```

**交叉连接**

```
SQL> select a.name,b.money from a full join b on a.name=b.name;

NAME                      MONEY
-------------------- ----------
a                          1000
b                          2000
c
d                          4000

SQL> select a.name,b.money from a,b where a.name(+)=b.name(+);
select a.name,b.money from a,b where a.name(+)=b.name(+)
                                              *
第 1 行出现错误:
ORA-01468: 一个谓词只能引用一个外部联接的表
```

所以(+)只是表示外连接，并不表示交叉连接。



**参考：**

[SQL Server 数据库 (+) 这个是什么意思](http://zhidao.baidu.com/link?url=4_K7GV8c8MfdhAjL6IBQl0hrDbguxmYK1S3B3Xc2GEmSdQHHrwWGu1GWIn7fXPhYRP1ihdgPjdk59c4xFcNilq)

[SQL表连接查询(inner join、full join、left join、right join)](https://www.cnblogs.com/still-windows7/archive/2012/10/22/2734613.html)