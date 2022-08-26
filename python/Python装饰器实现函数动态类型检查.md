---
abbrlink: 106366875
alias: 2017/02/16/Python装饰器实现函数动态类型检查/index.html
categories:
- python
date: '2017-02-16T19:48:03'
description: ''
tags:
- inspect
- 装饰器
- 类型检查
title: Python装饰器实现函数动态类型检查
---








# Python装饰器实现函数动态类型检查

## 函数动态类型检查的装饰器代码

```python
import inspect
import functools

def typeHints(fn):
    @functools.wraps(fn)
    def wrap(*args, **kwargs):
        sig = inspect.signature(fn)
        params = sig.parameters
        # 处理kwargs：字典
        for k, v in kwargs:
            param = params[k]
            if param.annotation != inspect._empty and not isinstance(v, param.annotation):
                raise TypeError('parameter {} requires {}, but got {}'.format(k, param.annotation, type(v)))
        # 处理args：元组
        for i, x in enumerate(args):
            param = list(params.values())[i]
            if param.annotation != inspect._empty and not isinstance(x, param.annotation):
                raise TypeError('parameter {} requires {}, but got {}'.format(param.name, param.annotation, type(x)))
        ret = fn(*args, **kwargs)
        return ret
    return wrap


@typeHints
def add(x: int, y: int) -> int:
    return x + y

@typeHints
def add1(x, y:int) -> int:
    return x + y

print(add(3, 5))	# 输出结果为8
print(add1(1, 2))	# 输出结果为3
```

类型检查主要使用了inspect库。本次代码运行环境是python3.5.2。inspect库的使用方法在下面介绍。

<!--more-->

## inspect模块

检查函数动态类型时，我们主要使用的是inspect库中的signature类，parameter类。可以使用help方法查看inspect的详细信息：

```python
import inspect
help(inspect)
```

inspect库的源代码见：/home/clg/.pyenv/versions/3.5.2/**lib/python3.5/inspect.py**

这个库用来获取Python动态对象的有用信息，比如本次用到的注解。

### Signature类

Signature是inspect模块的一个类，inspect模块的signature函数用来获取一个Signature对象，函数原型如下:

`signature() - get a Signature object for the callable`

Signature类有一个属性是OrderedDict类型的parameters，存储的是参数名称到参数对象（Parameter类的对象）的一个有序映射。

### Parameter类  

Parameter类的对象主要用来组成signature()返回的Signature对象的parameters属性。Parameter类有两个常用的属性：

- name :str 参数的名称 
- annotation 参数的注解，如果没有注解，则annotation为`Parameter.empty`

inspect模块示例

```python
def add(x: int, y: int) -> int:
    return x + y

import inspect
sig = inspect.signature(add)
print(sig.parameters)
print(sig.parameters['x'])
print(sig.parameters.values())

# 输出结果
OrderedDict([('x', <Parameter "x:int">), ('y', <Parameter "y:int">)])
x:int
odict_values([<Parameter "x:int">, <Parameter "y:int">])
```

`odict_values`类似于list，但是不支持下表操作，因此需要用`list()`转化为list之后再做下表操作。