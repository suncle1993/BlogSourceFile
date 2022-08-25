---
categories:
  - 数据库
date: '2016-02-19T22:50:12'
description: ''
tags:
  - SQL
  - Oracle
title: 复杂一点的SQL语句
---


## DDL：对表或者表的属性进行了改变

### create：创建表创建用户创建视图

创建表

create table student(id int,score int) ;

student后面与括号之间可以有空格可以没有

创建用户

create user liuyifei identified by 4852396;



drop：删除整个表、删除指定的用户、删除指定的存储空间

```
drop table table_name;
drop user user_name;

--删除空的表空间，但是不包含物理文件
drop tablespace tablespace_name;
--删除非空表空间，但是不包含物理文件
drop tablespace tablespace_name including contents;
--删除空表空间，包含物理文件
drop tablespace tablespace_name including datafiles;
--删除非空表空间，包含物理文件
drop tablespace tablespace_name including contents and datafiles;
--如果其他表空间中的表有外键等约束关联到了本表空间中的表的字段，就要加上CASCADE CONSTRAINTS
drop tablespace tablespace_name including contents and datafiles CASCADE CONSTRAINTS;
```



### truncate

删除表中的所有数据，但是表还是存在的。和drop的先后参见如下：

```
SQL> create table st1(id int);

表已创建。

SQL> truncate table st1;

表被截断。

SQL> drop table st1;

表已删除。

SQL> create table st1(id int);

表已创建。

SQL> drop table st1;

表已删除。

SQL> truncate table st1;
truncate table st1
               *
第 1 行出现错误:
ORA-00942: 表或视图不存在
```

<!--more-->

### alter：增加删除修改字段

```
SQL> create table s1(id int,a int,score int);

表已创建。

SQL> alter table s1 add name varchar2(10);

表已更改。

SQL>
SQL> desc s1;
 名称                                      是否为空? 类型
 ----------------------------------------- -------- ----------------------------

 ID                                                 NUMBER(38)
 A                                                  NUMBER(38)
 SCORE                                              NUMBER(38)
 NAME                                               VARCHAR2(10)

SQL> alter table s1 drop a;
alter table s1 drop a
                    *
第 1 行出现错误:
ORA-00905: 缺失关键字


SQL> alter table s1 drop column a;

表已更改。

SQL> alter table s1 rename to s2;

表已更改。

SQL> desc s2;
 名称                                      是否为空? 类型
 ----------------------------------------- -------- ----------------------------

 ID                                                 NUMBER(38)
 SCORE                                              NUMBER(38)
 NAME                                               VARCHAR2(10)

SQL> desc s1;
ERROR:
ORA-04043: 对象 s1 不存在


SQL> alter table s2 rename column name to sname;

表已更改。

SQL> desc s2;
 名称                                      是否为空? 类型
 ----------------------------------------- -------- ----------------------------

 ID                                                 NUMBER(38)
 SCORE                                              NUMBER(38)
 SNAME                                              VARCHAR2(10)
```

## DML：只对表的数据改变，没有改变表的属性

DML操作之后要进行commit操作才会更改数据库。

### select：查询

```
SQL> select score,sname from s2 where id='2';

     SCORE SNAME
---------- ----------
        99 ayun
```

### insert：插入记录

```
SQL> insert into s2 values(1,100,'aming');

已创建 1 行。

SQL> insert into s2 values(2,99,'ayun');

已创建 1 行。

SQL> insert into s2 values(3,79,'ahe');

已创建 1 行。
```

### delete：删除记录，不改变表的属性。

```
SQL> delete from s2 where score='100';

已删除 1 行。

SQL> select * from s2;

        ID      SCORE SNAME
---------- ---------- ----------
         2         99 ayun
         3         79 ahe

SQL> delete from s2;

已删除2行。

SQL> select * from s2;

未选定行

SQL> desc s2;
 名称                                      是否为空? 类型
 ----------------------------------------- -------- ----------------------------

 ID                                                 NUMBER(38)
 SCORE                                              NUMBER(38)
 SNAME                                              VARCHAR2(10)
```

### update：更新记录

```
SQL> update s2 set score=100 where sname='ahe';

已更新 1 行。

SQL> select * from s2;

        ID      SCORE SNAME
---------- ---------- ----------
         1        100 aming
         2         99 ayun
         3        100 ahe
```

## DCL：只改变属性

grant：授权

revoke：收回权限

```
grant语法：GRANT privilege[, ...] ON object[, ...] TO { PUBLIC | GROUP group| username}

权限privilege：
    select：查询
    insert：插入
    update：更新
    delete：删除
    rule：
    all：所有

grant select,insert,update on tablename to public;
给所有用户授予查询、插入、更新tablename表的权限
revoke select,insert,update on tablename from public;//收回所有用户查询、插入、更新tablename表的权限

object:
    table：表
    view：视图
    sequence：序列
    index：索引

grant select,insert,update on tablename,viewname,sequencename,indexname to public;

public:对所有用户开放权限
GROUP groupname:对该组所有用户开放权限
username：对指定用户开放权限
```



给用户授权，connect权限和resource权限。

不给新建用户授予*connect*权限，新建用户无法通过SID或SERVICE_NAME连接数据库实例。

不给新建用户授予resource权限，新建用户无法创建表。

```
SQL>
SQL> create user liuyifei identified by a4852396;

用户已创建。

SQL> conn liuyifei/a4852396;
ERROR:
ORA-01045: user LIUYIFEI lacks CREATE SESSION privilege; logon denied


警告: 您不再连接到 ORACLE。
SQL> show user;
USER 为 ""
SQL> conn / as sysdba;
已连接。
SQL> show user;
USER 为 "SYS"

SQL> grant connect to liuyifei;

授权成功。

SQL> conn liuyifei/a4852396;
已连接。
SQL> show user;
USER 为 "LIUYIFEI"

SQL> create table stu(id int);
create table stu(id int)
*
第 1 行出现错误:
ORA-01031: 权限不足


SQL> conn / as sysdba;
已连接。
SQL> show user;
USER 为 "SYS"
SQL> grant resource to liuyifei;

授权成功。

SQL> conn liuyifei/a4852396;
已连接。
SQL> create table stu(id int);

表已创建。

```

查看指定用户有哪些系统权限

这项操作只可以是dba查看，普通用户是不能查看的，即使是查看自己的。下面的代码已经验证了这个问题。

```
SQL> select * from dba_tab_privs where grantee=uper('liuyifei');
select * from dba_tab_privs where grantee=uper('liuyifei')
              *
第 1 行出现错误:
ORA-00942: 表或视图不存在


SQL> select * from dba_roles_privs where grantee=uper('liuyifei');
select * from dba_roles_privs where grantee=uper('liuyifei')
              *
第 1 行出现错误:
ORA-00942: 表或视图不存在


SQL> show user;
USER 为 "LIUYIFEI"
SQL> conn / as sysdba;
已连接。

SQL> select * from dba_tab_privs where grantee=upper('liuyifei');

未选定行

SQL> select * from dba_role_privs where grantee=upper('liuyifei');

GRANTEE                        GRANTED_ROLE                   ADM DEF
------------------------------ ------------------------------ --- ---
LIUYIFEI                       CONNECT                        NO  YES
LIUYIFEI                       RESOURCE                       NO  YES
```



**附录1：**

truncate和delete的区别

truncate会收回表空间，delete不会收回表空间

**附录2：**

sys用户和system用户的区别：

SYS用户，缺省始终创建，且未被锁定，拥有数据字典及其关联的所有对象

SYSTEM用户，缺省始终创建，且未被锁定，可以访问数据库内的所有对象