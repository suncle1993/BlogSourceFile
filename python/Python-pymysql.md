---
abbrlink: 1140005001
alias: 2017/04/14/Python-pymysql/index.html
categories:
- python
date: '2017-04-14T22:29:54'
description: ''
tags:
- pymysql
- mysql
- MariaDB
title: Python-pymysql
---








# 安装

安装mysql数据库的难度和oracle数据库简直没得比，安装步骤如下：

## 安装MariaDB

```shell
yum install mariadb mariadb-server  # 安装，centos7默认的mysql就是mariadb
systemctl start mariadb  # 启动mariadb
systemctl enable mariadb  # 开机自启动
mysql_secure_installation  # 设置root密码
mysql -uroot -p  # 登录
```

## 安装pymysql

```
pip install pymysql
```

# 基本操作

数据库基本操作主要是：

1. 创建连接
2. 获取游标
3. 执行sql
4. 提交事务：**针对非查询性SQL**

**代码**

```python
import pymysql

# connect函数打开数据库连接
conn = pymysql.connect(host='192.168.110.13', user='root', password='123456', database='student')
# cursor方法创建游标对象cur
cur = conn.cursor()
# execute方法执行SQL语句
cur.execute("SELECT VERSION()")
# fetchone方法获取单条数据
data = cur.fetchone()
print ('Database version : {}'.format(data))
# 关闭游标
cur.close()
# 关闭数据库连接
conn.close()
```

<!--more-->

# DDL

DDL：数据定义语言。包括创建表，创建索引等等

```python
import pymysql

# connect函数打开数据库连接
conn = pymysql.connect(host='192.168.110.13', user='root', password='123456', database='student')

# cursor方法创建游标对象cur
cur = conn.cursor()

# 创建表
sql = '''create table user (
         name char(20) not null,
         age int,  
         sex char(1))'''

cur.execute(sql)

# 关闭游标
cur.close()
# 关闭数据库连接
conn.close()
```

# DML

DML：数据操作语言，包含增删改三项操作。

## insert

```python
import pymysql

# connect函数打开数据库连接
conn = pymysql.connect(host='192.168.110.13', user='root', password='123456', database='student')

# cursor方法创建游标对象cur
cur = conn.cursor()

# 创建表
sql = '''insert into user(name, age, sex) values('suncle', 18, 'm')'''

try:
    # 执行sql语句
    cur.execute(sql)
    # 提交到数据库执行
    conn.commit()
except:
    # 如果发生错误则回滚
    conn.rollback()

# 关闭游标
cur.close()
# 关闭数据库连接
conn.close()
```

## update

```python
import pymysql

# connect函数打开数据库连接
conn = pymysql.connect(host='192.168.110.13', user='root', password='123456', database='student')

# cursor方法创建游标对象cur
cur = conn.cursor()

# 创建表
sql = '''update user t set t.age = 20 where t.name='suncle' '''

try:
    # 执行sql语句
    cur.execute(sql)
    # 提交到数据库执行
    conn.commit()
except:
    # 如果发生错误则回滚
    conn.rollback()

# 关闭游标
cur.close()
# 关闭数据库连接
conn.close()
```

## delete

```python
import pymysql

# connect函数打开数据库连接
conn = pymysql.connect(host='192.168.110.13', user='root', password='123456', database='student')

# cursor方法创建游标对象cur
cur = conn.cursor()

# 创建表
sql = '''delete from user where age=20 '''

try:
    # 执行sql语句
    cur.execute(sql)
    # 提交到数据库执行
    conn.commit()
except:
    # 如果发生错误则回滚
    conn.rollback()

# 关闭游标
cur.close()
# 关闭数据库连接
conn.close()
```

# QUERY

## 基础查询

主要有三个函数

- cursor.fetchall 返回行的元组
- cursor.fetchmany 返回行的元组， 可以指定返回前N行 相当于对fetchall切片fetchall[:N]
- cursor.fetchone 返回首行， 相当于fetchall[0]

查询语句如下：

```python
cur.execute('''select * from user t where t.age<=19;''')
```

三种方法得到的结果分别为：

```python
cur.fetchall()  # (('suncle', 18, 'm'), ('suncle1', 19, 'm'))

cur.fetchmany(1)  # (('suncle', 18, 'm'),)

cur.fetchone()  # ('suncle', 18, 'm')
```

可见：每行数据也是一个元组， 元组的内容由sql决定

如果要让返回的数据带上列名，也就是要返回字典，那么就需要用到cursors.DictCursor。

## DictCursor

创建cursor时创建DictCursor类型的就可以fetch回来字典形式的结果了

**代码**

```python
import pymysql
conn = pymysql.connect(host='192.168.110.13', user='root', password='123456', database='student')

# 创建cursor时指定cursor参数cursor=pymysql.cursors.DictCursor表示cursor类型
cur = conn.cursor(cursor=pymysql.cursors.DictCursor)
cur.execute('''select * from user t where t.age<=20;''')
cur.fetchall()
```

fetchall返回结果为：

```python
[{'age': 18, 'name': 'suncle', 'sex': 'm'},
 {'age': 19, 'name': 'suncle1', 'sex': 'm'},
 {'age': 20, 'name': 'suncle2', 'sex': 'm'}]
```

返回每一行记录都是一个字典，整体结果是由字典组成的列表。而默认的cursor是由元组组成的元组。

# 参数化查询

## 基础的SQL注入

```python
import pymysql

conn = pymysql.connect(host='192.168.110.13', user='root', password='123456', database='student')
cur = conn.cursor()

def get_user(age=18):
    sql = '''select * from user t where t.age<={};'''.format(age)
    cur.execute(sql)
    return cur.fetchall()

get_user()  # 返回(('suncle', 18, 'm'),)

get_user('18 or 1=1')  # 返回(('suncle', 18, 'm'), ('suncle1', 19, 'm'))
```

当传入参数的age中带sql条件的时候，就会发生sql注入，使得结果可能并不满足要求。

为了解决sql注入，我们可以使用参数化查询。

## 使用参数化查询

以上代码做以下修改之后就可以避免sql注入

```python
import pymysql


conn = pymysql.connect(host='192.168.110.13', user='root', password='123456', database='student')
cur = conn.cursor()

def get_user(age=18):
    # 不管数据库定义的是什么类型，统一使用%s
    sql = '''select * from user t where t.age<=%s;'''.format(age)
    cur.execute(sql, (age, ))  # 参数化查询
    return cur.fetchall()
```

参数化查询最大的优势在于避免了SQL注入，同时参数化之后避免了sql多次硬解析，能提高查询效率。所以，总是应该使用参数化查询。

# 上下文管理

数据库连接和游标都支持上下文管理。

**游标**

查看cur实例对应Cursor类的方法

```python
cur = conn.cursor()
help(cur)
```

对应的with语句使用如下

```python
with cur:
    cur.execute('''select * from user''')

cur.execute('''select * from user''')  # 抛出错误：ProgrammingError: Cursor closed
```

with语句块结束之后cur就已经关闭了。

**连接**

通过help命令查看Connection类的`__enter__`和`__exit__`两种方法的实现

```python
conn = pymysql.connect(host='192.168.110.13', user='root', password='123456', database='student')
help(conn)  # conn是Connection类
```

查看结果如下：

```python
 |  __enter__(self)
 |      Context manager that returns a Cursor
 |  
 |  __exit__(self, exc, value, traceback)
 |      On successful exit, commit. On exception, rollback
```

- `__enter__`方法会返回一个游标
- `__exit__`方法：如果成功推出就会自动提交commit，如果发生异常就会回滚rollback

对应的with语句使用如下

```python
with conn as cur:
    cur.execute('''update user t set t.age = 20 where t.name='suncle' ''')

cur.execute('''select * from user''')  # 退出with块之后游标仍然没有关闭
```

虽然游标没有关闭， 但是数据库操作已经提交。

**游标和连接共同上下文管理**

```python
with conn as cur:
    with cur:
        cur.execute('''update user t set t.age = 20 where t.name='suncle' ''')
```

退出整个上下文管理块之后，游标会关闭，并且会自动提交。

# 数据库连接池

一般来说，应用程序访问数据库的过程是：

1. 装载数据库驱动程序
2. 建立数据库连接
3. 访问数据库，执行sql语句
4. 断开数据库连接

相对于性能正常的SQL的执行效率来说，建立连接是一个费时的活动，而且系统还要为每一个连接分配内存资源。在现在web请求的大并发量情况下，必然会导致频繁的数据库操作。而频繁的进行数据库连接操作势必占用很多的系统资源，使得系统的响应速度下降，严重的甚至会造成服务器的崩溃。

**引入数据库连接池技术之后，应用程序访问数据库的过程是：**

1. 请求数据库操作时，从连接池中取出创建好的数据库连接
2. 执行sql语句
3. 不断开数据库连接，而是放回连接池中，等待下次使用

连接池还有个优点就是能控制数据库的压力，当大量用户同时涌入时，连接池只会使用池限制数据库连接数目，而不会不停的向数据库请求连接，最后导致服务器崩溃。

**Python实现数据库连接池**

- 使用队列Queue保存数据库连接

代码如下

```python
from queue import Queue
import pymysql

class ConnectionPool():  # args和kwargs用来接收数据库url信息
    def __init__(self, size, *args, **kwargs):
        self.args = args
        self.kwargs = kwargs
        self.size = size
        self.pool = Queue(maxsize=self.size)
        for _ in range(self.size):
            self.pool.put(self._connect())
            
    def _connect(self):
        return pymysql.connect(*self.args, **self.kwargs)
    
    @staticmethod
    def _close(conn):
        conn.close()
    
    def get_connection(self):
        return self.pool.get()
 
    def return_connection(self, conn):
        return self.pool.put(conn)
 
    def close_pool(self):
        while not self.is_empty():
            self._close(self.pool.get())
 
    def is_empty(self):
        return self.pool.empty()
    
    def is_full(self):
        return self.pool.full()
    
    def current_connection_count(self):
        return self.pool.qsize()
    
    
pool = ConnectionPool(20, host='192.168.110.13', user='root', password='123456', database='student')

conn = pool.get_connection()  # 获取数据库连接
print(conn)  # <pymysql.connections.Connection at 0x7f6290300940>
print(pool.current_connection_count())  # 19
cur = conn.cursor()
cur.execute("SELECT VERSION()")
data = cur.fetchone()
print ('Database version : {}'.format(data[0]))
cur.close()
pool.return_connection(conn)  # 关闭游标之后需要回收数据库连接
print(pool.current_connection_count())  # 20
```

