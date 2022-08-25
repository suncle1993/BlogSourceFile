---
categories:
  - 数据库
date: '2016-02-19T22:45:12'
description: ''
tags:
  - plsql
  - Oracle
title: plsql
---


**PL/SQL**

PL/SQL也是一种程序语言，叫做**过程化SQL语言**（Procedural Language/SQL）。PL/SQL是Oracle数据库对SQL语句的扩展。在普通SQL语句的使用上增加了编程语言的特点，所以PL/SQL就是把数据操作和查询语句组织在PL/SQL代码的过程性单元中，通过逻辑判断、循环等操作实现复杂的功能或者计算的程序语言。

- SQL是一种集合性语言


- PL/SQL语句效率比SQL低，尽量用SQL。

**PL/SQL循环**

```
SQL> create table t(id int);

表已创建。

SQL> begin
  2  for i in 1..100 loop
  3  insert into t values(i);
  4  end loop;
  5  end;
  6  /

PL/SQL 过程已成功完成。

SQL> select count(*) from t;

  COUNT(*)
----------
       100

SQL> commit;

提交完成。

```

<!--more-->

**PL/SQL变量**

首先看看sql内置的数据类型

|       **数据类型**        | **长度**                         | **说明**                                   |
| :-------------------: | ------------------------------ | ---------------------------------------- |
|   CHAR(n BYTE/CHAR)   | 默认1字节，n值最大为2000                | 末尾填充空格以达到指定长度，超过最大长度报错。默认指定长度为字节数，字符长度可以从1字节到四字节。 |
|       NCHAR(n)        | 默认1字符，最大存储内容2000字节             | 末尾填充空格以达到指定长度，n为Unicode字符数。默认为1字节。       |
|     NVARCHAR2(n)      | 最大长度必须指定，最大存储内容4000字节          | 变长类型。n为Unicode字符数                        |
| VARCHAR2(n BYTE/CHAR) | 最大长度必须指定，至少为1字节或者1字符，n值最大为4000 | 变长类型。超过最大长度报错。默认存储的是长度为0的字符串。            |
|        VARCHAR        | 同VARCHAR2                      | 不建议使用                                    |
|     NUMBER(p[,s])     | 1-22字节。P取值范围1到38。S取值范围-84到127  | 存储定点数，值的绝对值范围为1.0 x 10 -130至1.0 x 10 126。值大于等于1.0 x 10 126时报错。p为有意义的10进制位数，正值s为小数位数，负值s表示四舍五 |
|     BINARY_FLOAT      | 5字节，其中有一长度字节。                  | 32位单精度浮点数类型。符号位1位，指数位8位，尾数位23            |
|     BINARY_DOUBLE     | 9字节，其中有一长度字节。                  | 64位双精度浮点数类型。                             |

具体变量声明参见一下链接：[https://blog.csdn.net/wang_zhou_jian/article/details/5693219](https://blog.csdn.net/wang_zhou_jian/article/details/5693219)

例、

```
SQL> declare
  2  a number(10):=10;
  3  begin
  4  for i in 1..a loop
  5  insert into t values(i);
  6
  7  end loop;
  8  end;
  9  /

PL/SQL 过程已成功完成。

SQL> select * from t;

        ID
----------
         1
         2
         3
         4
         5
         6
         7
         8
         9
        10

```