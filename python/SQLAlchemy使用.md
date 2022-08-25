---
categories:
  - python
date: '2017-05-10T00:37:13'
description: ''
tags:
  - SQLAlchemy
  - ORM
  - Python
title: SQLAlchemy使用
---




# 简介

SQLAlchemy是Python语言的一款流行的ORM（Object Relational Mapper）框架，该框架建立在数据库API之上，使用关系对象映射进行数据库操作，即将对象转换成SQL，然后使用数据API执行SQL并获取执行结果。

安装SQLAlchemy也很简单，直接使用pip安装即可。

```shell
pip install sqlalchemy
```

下面重点介绍SQLAlchemy的使用。

# 版本检查

```python
import sqlalchemy
sqlalchemy.__version__	# 1.1.9
```

当前sqlalchemy版本为1.1.9

# 连接数据库

```python
from sqlalchemy import create_engine
engine = create_engine('mysql+pymysql://root:123456@192.168.110.13:3306/student', echo=True)
```

- engine 是 Engine类的一个对象
- echo=True表明开启logging模块的日志
- 数据库连接：`engine://user:password@host:port/database`，其中engine为mysql+pymysql，或者是mysql+mysqldb，或者是oracle+cx_oracle等等

<!--more-->

# 创建表

```python
from sqlalchemy import create_engine
from sqlalchemy import Column, String, Integer
from sqlalchemy.ext.declarative import declarative_base


engine = create_engine('mysql+pymysql://root:123456@192.168.110.13:3306/student', echo=True)
Base = declarative_base()  # 生成Model类的基类

class User1(Base):
    __tablename__ = 'user1'
    
    extend_existing = True
    # 定义三个列
    id = Column(Integer, autoincrement=True, primary_key=True)
    name = Column(String(64), unique=True, nullable=False)
    age  = Column(Integer)
    
        
    def __repr__(self):
        return 'User(id={}, name={}, age={})'.format(self.id, self,name, self.age)
    
    def __str__(self):
        return self.__repr__()
        
Base.metadata.create_all(engine)  # 创建所有表
Base.metadata.drop_all(engine)  # 删除所有表

# 定义类的实例方法1
u1 = User()	# User类只接收一个位置参数self，和关键字参数**kwargs
u1.name = 'aa'  # 给User类的各个列赋值
u1.age=19
print(u1)  # User(id=None, name=aa, age=19)

# 定义类的实例方法2
u2 = User(name='bb', age='123')
print(u2)  # User(id=None, name=bb, age=123)
```

- 派生类User会继承基类Base的初始化函数`__init__`，会自动的接受我们所定义的列对应的关键字参数
- 未赋值的列会用None初始化，如上面的id

# Session

SQLAlchemy真正处理数据库的部分是Session。

如果已经创建好了一个Engine对象engine，那么可以用以下语句创建一个Session

```python
from sqlalchemy.orm import sessionmaker
Session = sessionmaker(bind=engine)
```

如果engine为创建好，则可以用以下语句创建

```python
Session = sessionmaker()
```

当engine创建好之后，在配置Session即可

```python
Session.configure(bind=engine)
```

当需要和数据库交互的时候，就需要实例化Session

```python
session = Session()
```

创建完成之后这个session并没有马上获取数据库连接。只有当这个session第一次操作数据库的时候才会从Engine维护的连接池中获取一个连接，并持有这个连接一直到我们提交了所有的改变或者关闭了这个session。

# DML

## insert

```python
user = User(name='haha', age='123')
session.add(user)
session.commit()
```

如果这个commit的过程中发生异常，则后续所有的commit都无法执行，因此DML都需要放在try...except中处理，如下

```python
user = User(name='flowsnow', age=18)
session.add(user)
try:
    session.commit()
except Exception as e:
    session.rollback()
    raise e
```

## update

和insert类似，都是使用session.add方法，但是update操作的时候需要数据库中存在带操作的记录。

```python
user.age = 20
session.add(user)
try:
    session.commit()
except Exception as e:
    session.rollback()
    raise e
```

## delete

删除之前必须确保数据库中存在要删除的记录。

```python
session.delete(user)	# user必须已经存在
try:
    session.commit()
except Exception as e:
    session.rollback()
    raise e
```

# QUERY

```python
for u in session.query(User).filter(User.age < 20).order_by(User.age.desc())[1:3]:
    print(u)
```

此条语句经ORM转换之后的SQL如下：

```sql
SELECT
	USER.id AS user_id,
	USER.NAME AS user_name,
	USER.age AS user_age
FROM USER
WHERE USER.age < % (age_1) s
ORDER BY USER.age DESC
LIMIT % (param_1) s, % (param_2) s
```

query函数的返回结果为一个Query对象，Query对象是可迭代的，支持切片操作。

下面列举常见的filter操作

- 相等

  ```python
  query.filter(User.name == 'suncle')
  ```

- 不相等

  ```python
  query.filter(User.name != 'suncle')
  ```

- 模糊匹配like：大小写敏感

  ```python
  query.filter(User.name.like('%sun%'))
  ```


- 模糊匹配ilike：大小写不敏感

  ```python
  query.filter(User.name.ilike('%sun%'))
  ```


- IN

  ```python
  query.filter(User.name.in_(['suncle', 'abc', 'suncle']))

  # 也支持Query对象
  query.filter(User.name.in_(
      session.query(User.name).filter(User.name.like('%sun%'))
  ))
  ```

- NOT IN

  ```python
  query.filter(~User.name.in_(['ed', 'wendy', 'jack']))
  ```

- IS NULL

  ```python
  query.filter(User.name == None)

  # 上面的写法不符合pep8规范，IDE会给出提示，可以用下面的方法替代，pep8的写法是is None
  query.filter(User.name.is_(None))
  ```

- IS NOT NULL

  ```python
  query.filter(User.name != None)

  # 上面的写法不符合pep8规范，IDE会给出提示，可以用下面的方法替代，pep8的写法是is not None
  query.filter(User.name.isnot(None))
  ```

- AND

  ```python
  # 方法1：使用and_()方法
  from sqlalchemy import and_
  query.filter(and_(User.name == 'flowsnow', User.age == 18))

  # 方法2：filter()支持多个关键字参数
  query.filter(User.name == 'flowsnow', User.age == 18)

  # 方法3：多次调用filter函数
  query.filter(User.name == 'flowsnow').filter(User.age == 18)
  ```


- OR

  ```python
  from sqlalchemy import or_
  query.filter(or_(User.name == 'suncle', User.name == 'flowsnow'))
  ```

下面列举SQL支持的常见的function

```python
from sqlalchemy import func
session.query(func.count(User.id)).first() # count
session.query(func.max(User.age)).first() # max
session.query(func.avg(User.age)).first() # avg
```

# Relationship

表和表之间会有外键关系，数据库的外键关系在ORM中的使用方法如下：

```python
from sqlalchemy import create_engine
from sqlalchemy import Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from sqlalchemy import ForeignKey
from sqlalchemy.orm import relationship

Base = declarative_base()  # 生成Model类的基类

class Author(Base):  # 作者类
    __tablename__ = 'author'
    
    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(64), unique=True, nullable=False)
    
    posts = relationship('Post')
    
    def __repr__(self):
        return 'Author<id={}， name={}>'.format(self.id, self.name)
    
    def __str__(self):
        return self.__repr__()
    
    
class Post(Base):  # 文章类
    __tablename__ = 'post'
    
    id = Column(Integer, primary_key=True, autoincrement=True)
    title = Column(String(128), nullable=False, index=True)
    content = Column(String(8096), nullable=False)
    author_id = Column(Integer, ForeignKey('author.id'), nullable=False)
    
    author = relationship('Author')
    
    def __repr__(self):
        return 'Post<id={}, title={}>'.format(self.id, self.title)
    
    def __str__(self):
        return self.__repr__()
    

engine = create_engine('mysql+pymysql://root:123456@192.168.110.13:3306/student', echo=True)
Base.metadata.create_all(engine)
Session = sessionmaker(bind=engine)
session = Session()

# 新增一个作者
author = Author()
author.name = 'flowsnow'
session.add(author)
session.commit()
print(author)  # Author<id=1， name=flowsnow>

# 新增一篇文章
post = Post()
post.title = 'first post'
post.content = 'oihdoshfohro'
post.author = author
session.add(post)
session.commit()
print(author.posts)  # [Post<id=1, title=first post>]

# 再新增一篇文章
post = Post()
post.title = 'second post'
post.content = 'liabhgekegpaerg'
post.author = author
session.add(post)
session.commit()
print(author.posts)  # [Post<id=1, title=first post>, Post<id=2, title=second post>]
```

数据库维护数据之间的外键关系会消耗数据库资源，影响性能，在大型的应用中一般不使用外键等数据库高级特性，而是由应用框架来维护数据之间的约束。

---

**参考**

1. [官方文档-Object Relational Tutorial](http://docs.sqlalchemy.org/en/rel_1_1/orm/tutorial.html)
2. [A step-by-step SQLAlchemy tutorial](http://www.rmunn.com/sqlalchemy-tutorial/tutorial.html)
3. [廖雪峰-使用SQLAlchemy](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014320114981139589ac5f02944601ae22834e9c521415000)