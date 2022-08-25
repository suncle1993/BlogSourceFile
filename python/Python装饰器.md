---
categories:
  - python
date: '2017-02-15T19:34:59'
description: ''
tags:
  - Python
  - 装饰器
  - 元信息
  - 柯里化
title: Python装饰器
---



# Python装饰器

## 引入装饰器

如果想在一个函数执行前后执行一些别的代码，比如打印一点日志用来输出这个函数的调用情况那应该怎么做呢？

```python
#!/usr/bin/env python
# coding=utf-8

def logger(fn):									# 函数作为参数即fn可以为任何参数
    def wrap(*args, **kwargs):					# 可变参数args和kwargs
        print('call {}'.format(fn.__name__))	
        ret = fn(*args, **kwargs)				# 函数调用时的参数解构
        print('{} called'.format(fn.__name__))
        return ret								# 返回函数的返回值
    return wrap

def add(x, y):
    return x + y

logger_add = logger(add)
print(logger_add.__name__)
print(logger_add)
ret = logger_add(3, 5)
print(ret)

#输出结果：
wrap
<function logger.<locals>.wrap at 0x7fba35f4fe18>
call add
add called
8
```

<!--more-->

也可以用以下方式来实现这种效果

```python
@logger                                                                                  
def add(x, y):                                                                            
	return x + y                                                                         ret = add(3, 5)                                                                      
print(ret) 

# 输出结果：
call add
add called
8
```

这就是Python装饰器的一个简单使用

## 什么是装饰器？

装饰器是用于软件设计模式的名称。 装饰器可以动态地改变函数，方法或类的功能，而不必直接使用子类或改变被装饰的函数的源代码。Python装饰器是对Python语法的一种特殊改变，它允许我们更方便地修改函数，方法以及类。

当我们按照以下方式编写代码时：

```python
@logger
def add(x, y):
	...
```

和单独执行下面的步骤是一样的：

```
def add(x, y):
	...
logger_add = logger(add)
```

装饰器内部的代码一般会创建一个新的函数，利用`*args`和`**kwargs`来接受任意的参数，上述代码中的wrap()函数就是这样的。在这个函数内部，我们需要调用原来的输入函数（即被包装的函数，它是装饰器的输入参数）并返回它的结果。但是也可以添加任何想要添加的代码，比如在上述代码中输出函数的调用情况，也可以添加计时处理等等。这个新创建的wrap函数会作为装饰器的结果返回，取代了原来的函数。

所以**在Python中，装饰器的参数是一个函数， 返回值是一个函数的函数**。

## 装饰器的示例：计时处理

写一个装饰器，用来计算一个函数的执行时间

```python
import time

def timethis(fn):
    def wrap(*args, **kwargs):
        start = time.time()
        ret = fn(*args, **kwargs)
        end = time.time()
        print(fn.__name__, end - start)
        return ret
    return wrap
```

如果要对add函数计时：

```python
@timethis
def add(x, y):
    return x + y

ret = add(3, 5)
print(ret)

# 输出结果
add 1.9073486328125e-06
8
```

如果要对sleep函数计时：

```python
@timethis
def sleep(x):
    time.sleep(x)

sleep(3)

# 输出结果
sleep 3.003262519836426
```

## 保存被装饰函数的元信息

### 什么是函数的元信息

比如装饰器的名称，装饰器的doc等等。我们可以使用dir函数列出函数的所有元信息：`dir(sleep)`，输出结果如下

```
['__annotations__', '__call__', '__class__', '__closure__', '__code__', '__defaults__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__get__', '__getattribute__', '__globals__', '__gt__', '__hash__', '__init__', '__kwdefaults__', '__le__', '__lt__', '__module__', '__name__', '__ne__', '__new__', '__qualname__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__']
```

可以看到有很多的元信息，我们比较常用的是`__name__`和`__doc__`这两个属性\

而且`__doc__`属性也就是函数的文档信息，可以通过help函数查看得到

### 为什么要保存被装饰函数的元信息

改写**装饰器的应用1：计时处理**中的sleep函数如下：

```python
@timeit
def sleep(x):
    '''This function is sleep.'''
    time.sleep(x)

sleep(3)
print(sleep.__name__)
print(sleep.__doc__)
```

以上代码输出结果如下：

```
3.0032713413238525
wrap
None
```

可以发现sleep函数的`__name__`是wrap，而不是sleep，而`__doc__`属性为空，而不是sleep函数的docstring。也就是说**经过装饰器装饰过后的函数的元信息发生了改变**，这时候如果程序需要函数的元信息，那么就有问题了。

### 如何保存被装饰函数的元信息

#### 方案1：手动给被装饰函数的元信息赋值

以`__name__`和`__doc__`这两个属性为例

```python
import time

def timeit(fn):
    def wrap(*args, **kwargs):
        start = time.time()
        ret = fn(*args, **kwargs)
        end = time.time()
        print(end - start)
        return ret
    wrap.__doc__ = fn.__doc__	# 手动赋值__doc__信息
    wrap.__name__ = fn.__name__	# 手动赋值__name__信息
    return wrap

@timeit
def sleep(x):
    '''This function is sleep.'''
    time.sleep(x)

if __name__ == "__main__":
    sleep(3)
    # print(dir(sleep))
    print(sleep.__name__)
    print(sleep.__doc__)
```

输出结果如下

```
3.004547119140625
sleep
This function is sleep.
```

可以发现，`__name__`和`__doc__`这两个属性确实赋值成功了。

我们可以将元信息赋值的过程改写为函数，如下

```python
import time


def copy_properties(src, dst):	# 将元信息赋值的过程改成函数copy_properties
    dst.__name__ = src.__name__
    dst.__doc__ = src.__doc__

def timeit(fn):
    def wrap(*args, **kwargs):
        start = time.time()
        ret = fn(*args, **kwargs)
        end = time.time()
        print(end - start)
        return ret
    copy_properties(fn, wrap)	# 调用copy_properties函数修改元信息
    return wrap

@timeit
def sleep(x):
    '''This function is sleep.'''
    time.sleep(x)

if __name__ == "__main__":
    sleep(3)
    # print(dir(sleep))
    print(sleep.__name__)
    print(sleep.__doc__)
```

这样修改后，同样可以解决问题。

继续修改copy_properties函数，使得copy_properties可以返回一个函数

```python
def copy_properties(src):
    def _copy(dst):	# 内置一个_copy函数便于返回
        dst.__name__ = src.__name__
        dst.__doc__ = src.__doc__
    return _copy

def timeit(fn):
    def wrap(*args, **kwargs):
        start = time.time()
        ret = fn(*args, **kwargs)
        end = time.time()
        print(end - start)
        return ret
    copy_properties(fn)(wrap)	# 调用copy_properties函数
    return wrap
```

同样可以问题。

如果继续修改copy_properties函数，使得_copy函数是一个装饰器，传入dst，返回dst，修改如下：

```python
def copy_properties(src):	# 先固定dst，传入src
    def _copy(dst):	# 传入dst
        dst.__name__ = src.__name__
        dst.__doc__ = src.__doc__
        return dst	# 返回dst
    return _copy	# 返回一个装饰器

def timeit(fn):
    @copy_properties(fn)	# 带参数装饰器的使用方法
    def wrap(*args, **kwargs):
        start = time.time()
        ret = fn(*args, **kwargs)
        end = time.time()
        print(end - start)
        return ret
    return wrap
```

copy_properties在此处返回一个带参数的装饰器，因此可以直接按照装饰器的使用方法来装饰wrap函数，这个修改copy_properties函数的过程称为函数的柯里化。

#### 方案2：使用functools库的@wraps装饰器

functools库的@wraps装饰器本质上就是copy_properties函数的高级版本：包含更多的函数元信息。首先查看wrap装饰器的帮助信息：

```python
import functools
help(functools.wraps)
```

wrap装饰器函数的原型是：

```python
wraps(wrapped, assigned=('module', 'name', 'qualname', 'doc', 'annotations'), updated=('dict',))
```

所以这个装饰器会复制module等元信息，但是也不是所有的元信息，并且会更新dict。

使用示例如下：

```python
import time
import functools

def timeit(fn):
    @functools.wraps(fn)	# wraps装饰器的使用
    def wrap(*args, **kwargs):
        start = time.time()
        ret = fn(*args, **kwargs)
        end = time.time()
        print(end - start)
        return ret
    return wrap

def sleep(x):
    time.sleep(x)

print(sleep.__name__)
print(sleep.__doc__)
```

## 编写一个带参数的装饰器

如果上述的timeit装饰器，我们需要输出执行时间超过若干秒（比如一秒）的函数的名称和执行时间，那么就需要给装饰器传入一个参数s，表示传入的时间间隔，默认为1s。

我们可以给写好的装饰器外面包一个函数timeitS，时间间隔s作为这个函数的参数传入，并且对内层的函数可见，然后这个函数返回写好的装饰器。

```python
import time
import functools


def timeitS(s):
    def timeit(fn):
        @functools.wraps(fn)
        def wrap(*args, **kwargs):
            start = time.time()
            ret = fn(*args, **kwargs)
            end = time.time()
            if end - start > s:
                print('call {} takes {}s'.format(fn.__name__, end - start))
            else:
                print('call {} takes {}s less than {}'.format(fn.__name__, end - start, s))
            return ret
        return wrap
    return timeit

@timeitS(2)
def sleep(x):
    time.sleep(x)

sleep(3)
sleep(1)

```

输出结果如下：

```
call sleep takes 3.001342535018921s
call sleep takes 1.000471830368042s less than 2
```

所以，我们可以将带参数的装饰器理解为：

* 带参数的装饰器就是一个函数， 这个函数返回一个不带参数的装饰器