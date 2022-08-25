---
categories:
  - python
date: '2017-03-15T20:55:20'
description: ''
tags:
  - Python
  - 面向对象
title: Python面向对象的魔术方法
---




# 魔术方法

查看类的魔术方法

```python
class A:
    pass
dir(A)  # 可以得到类所有公有成员
```

输出结果如下

```python
['__class__',
 '__delattr__',
 '__dict__',
 '__dir__',
 '__doc__',
 '__eq__',
 '__format__',
 '__ge__',
 '__getattribute__',
 '__gt__',
 '__hash__',
 '__init__',
 '__le__',
 '__lt__',
 '__module__',
 '__ne__',
 '__new__',
 '__reduce__',
 '__reduce_ex__',
 '__repr__',
 '__setattr__',
 '__sizeof__',
 '__str__',
 '__subclasshook__',
 '__weakref__']
```

在Python中，**所有以`__`双下划线包起来的方法，都统称为魔术方法**。比如最常见的 `__init__` 。

<!--more-->

# 创建/销毁

- `__new__`: `object.__new__(cls)` 创建类的方法：构造函数
- `__del__`:删除类：析构函数
- `__init__`：初始化函数

```python
class A:
    def __new__(cls, *args, **kwargs):
        print('new')
        return object.__new__(cls)

    def __init__(self):
        print('init')
        self.x = 3

    def __del__(self):
        print('del')

A() # 返回一个类<__main__.A at 0x7f4a84767978>
# 输出
new
init

a = A()
del a  # 输出del
```

每当实例空间被收回时(在垃圾收集时)，`__del__`就会自动执行。

# 运算符重载

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        return Point(self.x + other.x, self.y + other.y)

    def __sub__(self, other):
        return Point(self.x - other.x, self.y - other.y)

a = Point(0, 0)
b = Point(3, 5)
c = a + b
c += Point(4, 6)
print(c.x, c.y)  # 7, 11  
p = Point(3, 5) - Point(2, 1)
print(p.x, p.y)  # 1, 4
```

类的对象之间可以进行加减运算，只要类实现了加减运算对应的魔术方法即可。加法的具体实现是`__add__`，减法的具体实现是`__sub__`。

- 具体运算符对应的重载函数可以参考int类中运算符重载的实现：help(int)

**不要过度使用运算符重载**

```python
Point.__add__ = lambda self, value: self - value
p = Point(3, 5) + Point(4, 6)
print(p.x, p.y)  # 输出-1, -1
```

`__add__`的具体实现如果写成了减法，这种类型的错误非常不容易发现，因此如果不是在写库给第三方使用的时候，基本用不上运算符重载。

# hash

- 使用内置函数`hash`对某个对象求hash值时， 会调用对象的`__hash__`方法，示例代码如下

```python
In [1]: class Point:
   ...:     def __hash__(self):
   ...:         return 1
   ...:     

In [2]: hash(Point())
Out[2]: 1
```

- `__hash__`方法必须返回int，否则会抛出TypeError

```python
In [1]: class Point:
   ...:     def __hash__(self):
   ...:         return 'aaa'
   ...:     

In [2]: hash(Point())
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-5-a919dcea3eae> in <module>()
----> 1 hash(Point())

TypeError: __hash__ method should return an integer
```

- 可hash对象，就是具有`__hash__`方法的对象

```python
In [6]: class Point:
   ...:     def __hash__(self):
   ...:         return 1
   ...:         

In [7]: set([Point(), 12]) # 可hash
Out[7]: {<__main__.Point at 0x7f19d4073320>, 12}

In [8]: Point.__hash__ = None

In [9]: set([Point(), 12])  # 不能放在集合里面，因为不能hash
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-10-25999920b521> in <module>()
----> 1 set([Point(), 12])

TypeError: unhashable type: 'Point'

```

- 一个类如果没有重写`__hash__`方法的话，这个类的每个对象，通常具有不同的hash

```python
In [1]: class Point:
   ...:     pass
   ...: 

In [2]: p1 = Point()

In [3]: p2 = Point()

In [4]: hash(p1)
Out[4]: 8757059543567

In [5]: hash(p2)
Out[5]: 8757059543756
```

- 通常 `__hash__` 会和 `__eq__`一起使用， 因为解释器通常同时判断hash是否相等以及实例是否相等

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __hash__(self):
        return hash('{}:{}'.format(self.x, self.y))

    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

p1 = Point(3, 5)
p2 = Point(3, 5)
set([p1, p2])  # 返回 {<__main__.Point at 0x7f286092d588>}
hash(p1) == hash(p2)  # 返回True
p1 == p2  # 返回True
```

# 大小

当对象实现了`__len__`方法时，可以使用内置方法`len`求对象的长度, `__len__`方法必须返回非负整数

```python
lst = [1, 2, 3]
len(lst)  # 返回3
lst.__len__()  # 返回3
```

因此内置函数和`__len__`方法的效果相同。

```python
class Sized:
    def __len__(self):
        return 10

len(Sized())  # 返回10
```

# bool

- 当对象o实现了`__bool__` 方法时， `bool(o)`返回值为`o.__bool__()`

```python
class F:
    def __bool__(self):
        return False

bool(F())  # 返回False

class T:
    def __bool__(self):
        return True

bool(T())  # 返回True
```

- 当对象o没有实现`__bool__`方法时，如果o实现了`__len__`方法， `bool(o)`返回值为 `len(o) != 0`

```python
class L:
    def __len__(self):
        return 3

bool(L())  # 返回True

class Q:
    def __len__(self):
        return 0

bool(Q())  # 返回False
```

- 当对象o既没有实现`__bool__`方法，也没有实现 `__len__`方法的时候， `bool(o)`返回值为`True`

```python
class Boolean:
    pass

bool(Boolean())  # 返回True
```

- `__bool__`优先级比`__len__`更高

```python
class Sized:
    def __init__(self, size):
        self.size = size

    def __len__(self):
        return self.size

    def __bool__(self):
        return self.size == 0

bool(Sized(0))  # 返回True
bool(Sized(10))  # 返回False
```

- `__bool__`方法必须返回bool类型

```python
class B:
    def __bool__(self):
        return None  # 返回非bool类型的值时会出错，即使返回int型的也会报错

bool(B())
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-80-4efbb03885fe> in <module>()
----> 1 bool(B())

TypeError: __bool__ should return bool, returned NoneType
```

# 可视化

- `__str__`方法，print函数本质是调用对象的`__str__`方法，用于给人读
- `__repr__`方法，repr函数本质是调用对象的`__repr__`方法，用于给机器读

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __str__(self):  # 给人来读
        return 'Point<{}, {}>'.format(self.x, self.y)

    def __repr__(self): # 给机器读的
        return 'Point({}, {})'.format(self.x, self.y)

print(Point(3, 5))  # Point<3, 5>
print(repr(Point(3, 5)))  # Point(3, 5)
```

> repr:返回对象的规范化的字符串表示

# 可调用对象

```python
class Fn:
    def __call__(self):
        print('{} called'.format(self))

f = Fn()
f()

# 输出
<__main__.Fn object at 0x7fd254367470> called
```

一个对象，只要实现了`__call__`方法， 就可以通过小括号来来调用， 这一类对象，称之为可调用对象

给对象加上函数也就是对`__call__`方法加上参数：

```python
class Add:
    def __call__(self, x, y):
        return x + y

Add()(3, 5)  # 返回8，等价于 add =Add() add(3, 5)
```

**可调用对象的应用实例：实现可过期可换出的cache装饰器**

```python
import inspect
import datetime
from functools import wraps

class Cache:
    def __init__(self, size=128, expire=0):
        self.size = size
        self.expire = 0
        self.data = {}

    @staticmethod
    def make_key(fn, args, kwargs):
        ret = []
        names = set()
        params = inspect.signature(fn).parameters
        keys = list(params.keys())
        for i, arg in enumerate(args):
            ret.append((keys[i], arg))
            names.add(keys[i])
        ret.extend(kwargs.items())
        names.update(kwargs.keys())
        for k, v in params.items():
            if k not in names:
                ret.append((k, v.default))
        ret.sort(key=lambda x: x[0])
        return '&'.join(['{}={}'.format(name, arg) for name, arg in ret])

    def __call__(self, fn):
        @wraps(fn)
        def wrap(*args, **kwargs):
            key = self.make_key(fn, args, kwargs)
            now = datetime.datetime.now().timestamp()
            if key in self.data.keys():
                value, timestamp, _ = self.data[key]
                if expire == 0 or now - timestamp < expire:
                    self.data[key] = (value, timestamp, now)
                    return value
                else:
                    self.data.pop(key)
            value = fn(*args, **kwargs)
            if len(self.data) >= self.size: 
                # 过期清理
                if self.expire != 0:
                    expires = set()
                    for k, (_, timestamp, _) in self.data.items():
                        if now - timestamp >= self.expire:
                            expires.add(k)
                    for k in expires:
                        self.data.pop(k)
            if len(self.data) >= self.size:
                # 换出
                k = sorted(self.data.items(), key=lambda x: x[1][2])[0][0]
                self.data.pop(k)
            self.data[key] = (value, now, now)
            return value
        return wrap

@Cache()
def add(x, y):
    return x + y

add(1, 2)  # 返回3
```

用`__call__`来实现可调用对象，和闭包是殊途同归的，通常是为了封装一些内部状态

# 上下文管理

## 支持上下文管理的对象

```python
class Context:
    def __enter__(self):
        print('enter context')

    def __exit__(self, *args, **kwargs):
        print('exit context')
```

当一个对象同时实现了`__enter__`和`__exit__`方法，那么这个对象就是支持上下文管理的对象。

支持上下文管理的对象可以使用以下语句块进行处理：

```python
with obj:
    pass
```

比如

```python
with Context():
    print('do somethings')
print('out of context')

# 输出
enter context
do somethings
exit context
out of context
```

所以，`with`开启一个语句块， 执行这个语句块之前，会执行 `__enter__`方法， 执行这个语句块之后，会执行`__exit__` 方法，也就是说在这个语句块的前后会执行一些操作，因此也叫上下文。

- **即使with块抛出异常，`__enter__`和`__exit__`也会被执行，所以上下文管理是安全的。**

```python
with Context():
    raise Exception()

enter context
exit context
---------------------------------------------------------------------------
Exception                                 Traceback (most recent call last)
<ipython-input-126-c1afee4bfdab> in <module>()
      1 with Context():
----> 2     raise Exception()

Exception: 
```

- **即使`with`块中主动退出解释器， `__enter__` 和`__exit__`也能保证执行**

```python
import sys

with Context():
   sys.exit()

enter context
exit context
An exception has occurred, use %tb to see the full traceback.

SystemExit

/home/clg/.pyenv/versions/3.5.2/envs/normal/lib/python3.5/site-packages/IPython/core/interactiveshell.py:2889: UserWarning: To exit: use 'exit', 'quit', or Ctrl-D.
  warn("To exit: use 'exit', 'quit', or Ctrl-D.", stacklevel=1)
```

## with块的as字句

- **`as`子句可以获取`__enter__`方法的返回值**

```python
class Context:
    def __enter__(self):
        print('enter context')
        return self  # __enter__函数的返回值

    def __exit__(self, *args, **kwargs):
        print('exit context')

ctx = Context()
with ctx as c:
    print(id(ctx))
    print(id(c))
    print(c)

# 输出结果
enter context
140541332713712
140541332713712
<__main__.Context object at 0x7fd2543670f0>
exit context
```

## `__enter__`方法

- **`__enter__`方法的返回值可以被as字句捕获到**


- **`__enter__` 除self之外，不带任何参数**

```python
class Context:
    def __enter__(self, *args, **kwargs):
        print('enter context')
        print(args)
        print(kwargs)


    def __exit__(self, *args, **kwargs):
        print('exit context')

# 输出
enter context
()
{}
exit context
```

args和kwargs都是空的，因此上下文管理的时候`__enter__`函数除self外，不带任何参数。

## `__exit__`方法

- **`__exit__`的返回值，没有办法获取到，如果`with`块中抛出异常 `__exit__`返回False的时候，会向上抛出异常，返回True， 会屏蔽异常**

```python
class Context:
    def __enter__(self):
        print('enter context')

    def __exit__(self, *args, **kwargs):
        print('exit context')
        return 'haha'

with Context() as c:
    print(c)

# 输出
enter context
None
exit context
```

- **`__exit__`的三个参数 异常类型， 异常， traceback**

```python
class Context:
    def __enter__(self):
        print('enter context')

    def __exit__(self, *args, **kwargs):
        print('exit context')
        print(args)
        print(kwargs)

with Context():
    pass

# 输出
enter context
exit context
(None, None, None)
{}
```

args输出三个None，表示三个位置参数，kwargs为空，表示没有关键字参数。

```python
with Context():
    raise Exception()

enter context
exit context
(<class 'Exception'>, Exception(), <traceback object at 0x7f28608fdc88>)
{}
---------------------------------------------------------------------------
Exception                                 Traceback (most recent call last)
<ipython-input-145-c1afee4bfdab> in <module>()
      1 with Context():
----> 2     raise Exception()

Exception: 
```

- **使用变量接受`__exit__`的三个参数：exc_type,exc_value,traceback**

```python
class Context:
    def __enter__(self):
        print('enter context')

    def __exit__(self, exc_type, exc_value, traceback):
        print('exit context')
        print('exception type: {}'.format(exc_type))
        print('exception value: {}'.format(exc_value))
        print('exception traceback: {}'.format(traceback))
        return True

with Context():
    raise TypeError('hahaha')

# 输出
enter context
exit context
exception type: <class 'TypeError'>
exception value: hahaha
exception traceback: <traceback object at 0x7fd257c18608>
```

## 上下文管理的应用场景

with 语句适用于对资源进行访问的场合，确保不管使用过程中是否发生异常都会执行必要的“清理”操作，释放资源，比如文件使用后自动关闭、线程中锁的自动获取和释放等。即**凡是在代码块前后插入代码的场景统统适用**

1. 资源管理
2. 权限验证

以下以计时器为例

```python
from functools import wraps
class Timeit:
    def __init__(self, fn=None):
        wraps(fn)(self)

    def __call__(self, *args, **kwargs):
        start = datetime.datetime.now()
        ret = self.__wrapped__(*args, **kwargs)
        cost = datetime.datetime.now() - start
        print(cost)
        return ret

    def __enter__(self):
        self.start = datetime.datetime.now()

    def __exit__(self, *args):
        cost = datetime.datetime.now() - self.start
        print(cost)

with Timeit():
    z = 3 + 8  # 输出0:00:00.000037

@Timeit
def add(x, y):
    return x + y

add(3, 8)  # 输出0:00:00.000044  返回11
```

总共实现了两种计时方式，既可以对语句块计时，也可以对函数计时。

## contextmanager的使用

contextlib是个比with优美的东西，也是提供上下文管理机制的模块，它是通过Generator装饰器实现的，不再是采用`__enter__`和`__exit__`。contextlib中的contextmanager作为装饰器来提供一种针对**函数级别**的上下文管理机制。

```python
import contextlib


@contextlib.contextmanager
def context():
    print('enter context') # 初始化部分 相当于 __enter__ 方法
    try:
        yield 'haha' # 相当于__enter__的返回值
    finally:
        print('exit context') # 清理部分， 相当于 __exit__ 方法


with context() as c:
    print(c)
    raise Exception()

# 输出
enter context
haha
exit context
---------------------------------------------------------------------------
Exception                                 Traceback (most recent call last)
<ipython-input-189-4c1dae6b647a> in <module>()
      1 with context() as c:
      2     print(c)
----> 3     raise Exception()

Exception: 
```

yield后面必须配合finally使用，否则如果抛出异常，程序不会执行yield后面的部门，也就是不会执行`__exit__`部分。

# 反射

**python的反射，核心本质其实就是利用字符串的形式去对象（模块）中操作（查找/获取/删除/添加）成员，就是一种基于字符串的事件驱动！**

关于模块的python反射以及反射机制分析参见：[python反射机制深入分析](https://www.cnblogs.com/feixuelove1009/p/5576206.html) 

以下主要分析类对象的反射机制

## getattr setattr hasattr

三个函数的原型：

1. getattr：getattr(object, name[, default]) -> value。getattr(x, 'y')等效于x.y
2. setattr：setattr(obj, name, value, /)。setattr(x, 'y', v)等效于x.y = v
3. hasattr：hasattr(obj, name, /)

主要作用是通过对象的成员名称获取对象的成员

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def print(self, x, y):
        print(x, y)

p = Point(3, 5)
p.__dict__['x'] # 返回3， 对于属性来说，可以通过 __dict__ 获取
getattr(p, 'print')(3, 5) # 成员方法无法通过__dict__获取，但是可以通过getattr函数获取 # p.print(3, 5)
getattr(p, 'x') # getattrr 也可以获取到属性
setattr(p, 'haha', 'abcd') # p.haha = 'abcd'，给对象p增加属性haha
p.haha  # 返回abcd
hasattr(p, 'print')  # 返回True

```

setattr的对象是实例，如果要给实例动态增加方法，需要先把函数转化为方法，转化的方法如下：

```python
import types

def mm(self):
    print(self.x)

setattr(p, 'mm', types.MethodType(mm, p))  # 将mm函数转化为对象p的方法之后，再给p增加
p.mm()  # 输出3
```

使用getattr setattr hasattr 实现一个命令路由器：

```python
class Command:
    def cmd1(self):
        print('cmd1')
    def cmd2(self):
        print('cmd2')
    def run(self):
        while True:
            cmd = input('>>>').strip()
            if cmd == 'quit':
                return
            getattr(self, cmd, lambda :print('not found cmd {}'.format(cmd)))()

command = Command()
command.run()

# 输出 
>>>cmd1
cmd1
>>>cmd2
cmd2
>>>cmd3
not found cmd cmd3
>>>quit
```

## `__getattr__` `__setattr__` `__delattr__`

- 当一个类定义了`__getattr__`方法时，如果访问不存在的成员，会调用`__getattr__`方法

```python
class A:
    def __init__(self):
        self.x = 3

a = A()
a.x  # 返回3
a.y  # 如果没有实现__getattr__方法，当访问不存在的成员时会报错
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-228-cc7049c6eeec> in <module>()
----> 1 a.y

AttributeError: 'A' object has no attribute 'y'
```

增加`__getattr__`方法

```python
class A:
    def __init__(self):
        self.x = 3

    def __getattr__(self, name):
        return 'missing property {}'.format(name)

a = A()
a.x  # 返回3
a.y  # 返回'missing property y'。即访问不存在的成员，会调用__getattr__方法
```

- 当一个类实现了`__setattr__`时， 任何地方对这个类的对象增加属性，或者对现有属性赋值，都会调用`__setattr__`

```python
class A:
    def __init__(self):
        self.x = 3

    def __setattr__(self, name, value):
        print('set {} to {}'.format(name, value))
        setattr(self, name, value)

a = A()
a.x  # 返回3
a.y = 5  # 输出set y to 5
```

- 当一个类实现了`__delattr__` 方法时，删除其实例的属性，会调用此方法

```python
class A:
    def __init__(self):
        self.x = 3

    def __delattr__(self, name):
        print('you cannot delete property: {}'.format(name))

a = A()
a.x  # 返回3
del a.x  # 输出you cannot delete property: x
```