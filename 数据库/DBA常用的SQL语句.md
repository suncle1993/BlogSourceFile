---
abbrlink: 1301073950
alias: 2016/03/16/DBA常用的SQL语句/index.html
categories:
- 数据库
date: '2016-03-16T11:07:19'
description: ''
tags:
- DBA
- SQL
title: DBA常用的SQL语句
---







----

DBA常用的SQL语句

**数据库的大小**

数据库的大小主要是数据文件（dba_data_files）和临时文件(dba_temp_files;)的大小之和。

```sql
--查询数据文件大小
SQL> select sum(bytes) from dba_data_files;

SUM(BYTES)
----------
1515192320

--查询临时文件大小
SQL> select sum(bytes) from dba_temp_files;

SUM(BYTES)
----------
  30408704

--查询数据库的大小：两项相加
SQL> select (select sum(bytes) from dba_data_files)+(select sum(bytes) from dba_
temp_files) from dual;

(SELECTSUM(BYTES)FROMDBA_DATA_FILES)+(SELECTSUM(BYTES)FROMDBA_TEMP_FILES)
-------------------------------------------------------------------------
                                                               1545601024

--取一个别名total_size
SQL> select (select sum(bytes) from dba_data_files)+(select sum(bytes) from dba_
temp_files) as total_size from dual;

TOTAL_SIZE
----------
1545601024
```

<!--more-->

**查询某个段对象（表，索引）的大小**

**dba_segments:**  **DBA_SEGMENTS**describes the storage allocated for all segments in the database

**查看有哪些表空间**

```
SQL> select tablespace_name from dba_tablespaces;

TABLESPACE_NAME
------------------------------
SYSTEM
SYSAUX
UNDOTBS1
TEMP
USERS
EXAMPLE

已选择6行。
```