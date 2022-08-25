---
categories:
  - 数据库
date: '2016-03-23T11:00:22'
description: ''
tags:
  - 事务
  - Transaction
  - Oracle
title: 事务Transaction
---


参考资料：

<a href="https://docs.oracle.com/cd/E11882_01/server.112/e40540/transact.htmhttps://docs.oracle.com/cd/E11882_01/server.112/e40540/transact.htm" rel="nofollow">Transactions</a>

[关于Oracle事务的总结](https://blog.csdn.net/junmail/article/details/5556561)

---

## 什么是事务？

事务(Transaction)是访问并可能更新数据库中各种[数据项](http://baike.baidu.com/view/178581.htm)的一个程序执行单元(unit)。事务由事务开始(**begin transaction**)和事务结束(**end transaction**)之间执行的全体操作组成。



## 事务的属性-ACID

- **原子性（Atomicity）**-事务的原子性强调了一个事物是一个逻辑工作单元，是一个整体，是不可分割的。一个事务所包含的操作要么全部做，要不全部不做。


- **一致性（Consistency）**-一个事务执行一项数据库操作，事务使数据库从一种一致性的状态变换成另一种一致性状态。

- **隔离性（Isolation）**-在事务未提交前，它操作的数据，对其他用户不可见。

- **持久性（Durability）**-一旦事务成功完成，该事务对数据库所施加的所有更新都是永久的。

  - redo日志--提交的事务被永久的记录到redo日志中。

<!--more-->


## 数据库事务的开始和结束

以第一个DML语句的执行作为开始

以下面的其中之一作为结束：

- commit或rollback语句
- DDL或DCL语句（自动提交）
- 用户会话正常结束--commit
- 系统异常终了--rollback

## 并发与数据的读取

当多个会话同时访问（操作）相同的数据时，将会出现一些意想不到的结果。包括：
- 脏读 --dirty reads

  > 一个事务读取了另一个事务未提交的数据,而这个数据是有可能回滚
  > ​
- 不可重复读 --non-repeatable reads

  > 在数据库访问中，一个事务范围内两个相同的查询却返回了不同数据。这是由于查询时系统中其他事务修改的提交而引起的。
  > ​
- 幻读 --Phantom（虚幻的） reads

  > 事务1读取记录时事务2增加了记录并提交，事务1再次读取时可以看到事务2新增的记录。对事物1而言就好像出现了幻觉一样。

## 事务的隔离等级

**ANSI定义的事务的隔离等级：**

| 事务隔离等级                 |  脏读  | 不可重复读 |  幻读  |
| :--------------------- | :--: | :---: | :--: |
| Read uncommited（读未提交的） |  Y   |   Y   |  Y   |
| Read commited（读提交的）    |  N   |   Y   |  Y   |
| Repeatable read        |  N   |   N   |  Y   |
| Serializable           |  N   |   N   |  N   |

**Oracle定义的事务隔离等级：**

| 事务隔离等级        | 影响                                       |
| :------------ | :--------------------------------------- |
| Read commited | Oracle默认的隔离等级，对一条SQL，可以保证数据的一致性，对于一个事务，无法做到repeatable read。 |
| Serializable  | 只能看到事务开始时所有提交的改变以及自身的改变                  |
| Read-only     | 只能看到事务开始时所有提交的改变，自身不允许DML操作              |



## 事务的并发控制-锁

Oracle的锁定机制

- Oracle尽可能的减少锁定的使用

- Oracle的读操作不会对表加锁，一些数据库会使用查询锁定（共享锁，排它锁）

- Oracle通过回滚机制，保证读不会受到阻塞

- Oracle没有锁管理器

- Oracle中锁作为数据块的一种属性存在

Oracle和Sql Server锁的区别

| Sql Server         | Oracle          |
| ------------------ | --------------- |
| 并发和读一致性不可兼得，必须牺牲一方 | 可兼得             |
| 因为锁实现方式，事务代价昂贵     | 没有真正的锁，事务没有资源代价 |
| 提倡尽快提交             | 主张按照业务需求确定事务边界  |



## 事务的控制-savepoint

通过在事务中间设置检查点，可以更加精细的控制事务，防止一部分错误操作导致整个事务重新运行。演示如下：

```sql
SQL> create table t(id int);

表已创建。

SQL> insert into t values(1);

已创建 1 行。

SQL> savepoint s1;

保存点已创建。

SQL> select * from t;

        ID
----------
         1

SQL> update t set id=2;

已更新 1 行。

SQL> savepoint s2;

保存点已创建。

SQL> select * from t;

        ID
----------
         2

SQL> rollback to s1;

回退已完成。

SQL> select * from t;

        ID
----------
         1
```

一旦返回到保存点s1之后s2就失去了效果，因为已经回到s1了，这时候s2还不存在。



## 自治事务

自治事务允许在一个事务中存在独立的事务，它的操作不会对当前事务产生影响。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-%E8%87%AA%E6%B2%BB%E4%BA%8B%E5%8A%A1.jpg)

语法：

```sql
pragma autonomous_transaction
```

关于自治事务的使用可以参考：[**ORACLE中的自治事务**](https://blog.csdn.net/fenglibing/article/details/4059924)

实验演示如下：(演示用例来自参考资料Oracle中的自治事务)

首先是不使用自治事务

```sql
SQL> create table msg (msg varchar2(120));
SQL> set serveroutput on
SQL> declare
  2    cnt number := -1;  --} Global variables
  3    procedure local is
  4    begin
  5       select count(*) into cnt from msg;
  6       dbms_output.put_line('local: # of rows is '||cnt);
  7
  8       insert into msg values ('New Record');
  9       commit;
 10    end;
 11    begin
 12       delete from msg ;
 13       commit;
 14       insert into msg values ('Row 1');
 15       local;
 16       select count(*) into cnt from msg;
 17       dbms_output.put_line('main: # of rows is '||cnt);
 18       rollback;
 19
 20       local;
 21       insert into msg values ('Row 2');
 22       commit;
 23
 24       local;
 25       select count(*) into cnt from msg;
 26       dbms_output.put_line('main: # of rows is '||cnt);
 27    end;
 28  /
local: # of rows is 1  -> 子程序local中可以’看到’主匿名块中的uncommitted记录
main: # of rows is 2   -> 主匿名块可以’看到’2条记录(它们都是被local commit掉的)
local: # of rows is 2  -> 子程序local首先’看到’2条记录,然后又commit了第三条记录
local: # of rows is 4  -> 子程序local又’看到’了新增加的记录(它们都是被local commit掉的),然后又commit了第五条记录
main: # of rows is 5   -> 主匿名块最后’看到’了所有的记录. 

PL/SQL 过程已成功完成。
```

从这个例子中,我们看到COMMIT和ROLLBACK的位置无论是在主匿名块中或者在子程序中,都会影响到整个当前事务. 

现在如果将procedure local改成自治事务,在procedure local后面加上：

`pragma AUTONOMOUS_TRANSACTION;`

效果如下：

```sql
SQL> declare
  2    cnt number := -1;  --} Global variables
  3    procedure local is
  4    pragma AUTONOMOUS_TRANSACTION;
  5    begin
  6       select count(*) into cnt from msg;
  7       dbms_output.put_line('local: # of rows is '||cnt);
  8
  9       insert into msg values ('New Record');
 10       commit;
 11    end;
 12    begin
 13       delete from msg ;
 14       commit;
 15       insert into msg values ('Row 1');
 16       local;
 17       select count(*) into cnt from msg;
 18       dbms_output.put_line('main: # of rows is '||cnt);
 19       rollback;
 20
 21       local;
 22       insert into msg values ('Row 2');
 23       commit;
 24
 25       local;
 26       select count(*) into cnt from msg;
 27       dbms_output.put_line('main: # of rows is '||cnt);
 28    end;
 29  /
local: # of rows is 0  -> 子程序local中无法可以’看到’主匿名块中的uncommitted记录 (因为它是独立的)
main: # of rows is 2   -> 主匿名块可以’看到’2条记录,但只有一条是被commited.
local: # of rows is 1  -> 子程序local中可以’看到’它前一次commit的记录,但是主匿名块中的记录已经被提前rollback了
local: # of rows is 3  -> 子程序local 中可以’看到’3条记录包括主匿名块commit的记录
main: # of rows is 4   ->主匿名块最后’看到’了所有的记录.

PL/SQL 过程已成功完成。
```



## 分布式事务

- 发生在多台数据库之间的事务。
- 通过dblink方式进行事务处理。
- 分布式事务要比单机事务要复杂的多。
- 可能的风险：软件，服务器，网络。

### 分布式事务的组成

| 角色                | 描述                                |
| ----------------- | --------------------------------- |
| client            | 调用其它数据库信息的节点                      |
| database          | 接受来自其它节点请求的节点                     |
| Global coordinate | 发起分布式事务的节点（全局调度者）                 |
| Local coordinate  | 处理本地事务，并和其它节点通信的节点（本地调度者）         |
| Commit point site | 被global coordinate指定第一个提交或回滚事务的节点 |

commit Point Strength

Oracle选取Commit Point Strength（相当于权重）最大的数据库作为Commit point。

### Oracle分布式事务的机制-两阶段提交

2PC-two phase commit

- prepare phase
- commit phase

**准备阶段prepare phase**

为了完成准备阶段，除了commit point机器外，其它的数据库机器按照以下步骤执行：

- 每个节点检查自己是否被其它节点所引用，如果有，就通知这些节点准备提交（进入prepare阶段）

- 每个节点检查自己运行的事务，如果发现本地运行的事务不做修改数据操作，则跳过后面的步骤，直接返回一个read only给全局协调进程。

- 如果事务需要修改数据，为事务分配相应的资源用于保证修改的正常进行。

- 对事物做的修改，记录redo信息。

- 本地redo保证事务失败后的回滚。

- 当上面的工作都成功后，给全局协调进程返回准备就绪的信息，反之，返回失败的信号。

**提交阶段commit phase**

提交阶段按下面的步骤进行：

- 全局协调器通知commit point进行提交
- commit point提交完成。
- commit point服务器通知全局协调器提交完成
- 全局协调器通知其它节点进行提交
- 其它节点提交本地的事务，释放资源（提交先后顺序根据Commit Point Strength）
- 其它节点在redo上记录相应的redo日志，并标注提交完成
- 其它节点通知全局协调器提交完成。

### 分布式事务的结束

分布式事务的结束就是全局协调器和commit point两者之间释放资源的顺序。

- 全局协调器通知commit point数据库所有节点提交完成。
- commit point数据库释放和事务相关的所有资源，然后通知全局协调器。
- 全局协调器释放自己持有的资源
- 分布式事务结束

###分布式事务的安全性

2PC是否真的可以保证分布式事务的一致性？

- 理论上是不可能保证分布式事务的一致性。

关于CAP理论可以参见：[CAP理论](https://blog.csdn.net/chen77716/article/details/30635543)