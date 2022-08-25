---
categories:
  - python
date: '2016-09-05T19:40:25'
description: ''
tags:
  - 函数
  - 参数
title: Python函数定义及参数详解
---



# Python函数定义及参数详解

## 函数定义

首先我们来创建一个函数，输出指定范围内的斐波拉契数列（Fibonacci series）。

```python
#!/usr/bin/env python 
#coding=utf-8
'''
Created on 2016年9月4日下午2:37:31
@author: Flowsnow
@file: D:/Workspaces/eclipse/HelloPython/main/FibonacciSeries.py
@function: 定义函数-输出给定范围内的斐波拉契数列
'''
def Fibonacci(n):
    #print "success"
    a=0
    b=1
    while a<n:
        print a,
        a,b=b,a+b

#call the function Fibonacci
Fibonacci(2000)
print '\n',
print Fibonacci
f=Fibonacci
f(100)
print '\n',
print Fibonacci(0)
```

<!--more-->

输出结果如下：

```
0 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597 
<function Fibonacci at 0x000000000258D9E8>
0 1 1 2 3 5 8 13 21 34 55 89 
None
```

>由第一行可知 Fibonacci函数输出了2000以内的斐波拉契数列。
>
>由第二行可知 Fibonacci函数在内存中的地址
>
>由第三行可知 将Fibonacci函数的地址值赋给另外一个变量f之后，f也就是一个函数了，这类似于重名机制
>
>由第四行可知 虽然Fibonacci函数没有`return`语句，但是如果我们使用`print`输出的时候可以发现还是有返回值的，只是这个返回值是`None`，这是Python的內建名称。

我们也可以写一个函数，不输出斐波拉契数列的值，而是把值作为返回值返回。

```python
#!/usr/bin/env python 
#coding=utf-8
'''
Created on 2016年9月4日下午3:07:06
@author: Flowsnow
@file: D:/Workspaces/eclipse/HelloPython/main/FibonacciSeriesAdv.py
@function: 函数定义-返回斐波拉契数列，而不是直接打印
'''
def Fibonacci(n):
    a=0
    b=1
    result=[]
    while a<n:
        result.append(a)
        a,b=b,a+b
    return result
result=Fibonacci(2000)
for x in result:
    print x, 
```

输出结果：0 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597

## 参数详解

Python 的内建标准类型有一种分类标准是分为可变类型与不可变类型

- 可变类型：列表、字典
- 不可变类型：数字、字符串、元组

上面函数定义中的参数都是属于不可变类型的。

可变参数三种情况：默认参数，位置参数`*args`关键字参数`**kwargs`。

### 默认参数

默认参数的好处就是在调用函数的时候写上去的参数比在函数定义时的参数少。例如：

```python
#!/usr/bin/env python 
#coding=utf-8
'''
Created on 2016年9月5日下午2:50:12
@author: Flowsnow
@file: D:/Workspaces/eclipse/HelloPython/main/askYesOrNo.py
@function: 测试默认参数的使用
'''
def ask_ok(prompt, retries=4, complaint='Yes or no, please!'):
    while True:
        ok = raw_input(prompt)
        if ok in ('y', 'ye', 'yes'):
            return True
        if ok in ('n', 'no', 'nop', 'nope'):
            return False
        retries = retries - 1
        if retries < 0:
            raise IOError('refusenik user')
        print complaint
```

这个函数的调用方法有很多，比如：

- 只给必选参数：`ask_ok('OK to overwrite the file?')`
- 给一个可选参数：`ask_ok('OK to overwrite the file?', 2)`
- 给所有的可选参数：`ask_ok('OK to overwrite the file?', 2, 'Come on, only yes or no!')`

关于默认值，应该注意的是默认值只会在函数定义的时候被python解析一次。因此

```python
i = 5

def f(arg=i):
    print arg

i = 6
f()
```

这段代码输出的应该是5，而不是6，就是因为i是在函数定义的时候解析的，这个时候i=5。

**重要警告：**默认值只会解析一次。当默认参数是可变对象时，影响比较大，比如列表，字典或者类的对象。下面演示的这个函数会把参数积累并传到随后的函数调用里面：

```python
def f(a, L=[]):
    L.append(a)
    return L

print f(1)
print f(2)
print f(3)
```

这段代码会输出

```python
[1]
[1, 2]
[1, 2, 3]
```

如果不想默认参数在后面的函数调用中共享，可以把函数写成这种形式

```python
def f(a, L=None):
    if L is None:
        L = []
    L.append(a)
    return L
```

这段代码会输出

```
[1]
[2]
[3]
```

### 位置参数*args

位置参数需要在参数前面加一个星号。把参数收集到一个元tuple中，作为变量args。至于为什么叫位置参数，这个是因为各个参数是按照顺序接收的。

```python
def argTest(arg1,*args):
    print arg1
    print('~start to print *args~')
    for x in args:
        print x,

argTest(1,'two',3)
```

这段代码会输出

```
1
~start to print *args~
two 3
```

args被解释为包含多个变量的元组tuple。因此也可用如下写法：

```python
def argTest(arg1,*args):
    print arg1
    print('~start to print *args~')
    for x in args:
        print x,

#argTest(1,'two',3)
args=['two',3]
argTest(1,*args)
```

### 关键字参数**kwargs

函数也能够按照`kwarg=value`这种形式的关键字参数来调用。关键字参数需要在参数前面加两个星号。其作用是把参数收集成一个字典类型，包含参数名和值。

```python
def argTest(arg1,**kwargs):
    print 'arg1',arg1
    for key in kwargs:
        print key,kwargs[key]
argTest(1,arg2='aa',arg3='bb')
argTest(arg1=1,arg2='aa',arg3='bb',arg4='cc')
arg={'arg2':'bb','arg3':'cc','arg4':'dd'}
argTest(arg1='ss',**arg)
argTest(arg1='ss',**arg)
```

这段代码会输出

```
arg1 1
arg2 aa
arg3 bb
arg1 1
arg2 aa
arg3 bb
arg4 cc
arg1 ss
arg2 bb
arg3 cc
arg4 dd
arg1 ss
arg2 bb
arg3 cc
arg4 dd
```

## 参考资料

[Python官网-defining-functions](https://docs.python.org/2/tutorial/controlflow.html#defining-functions)

[Passing arguments to Python functions1.pdf ](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/file/pdf/Passing%20arguments%20to%20Python%20functions1.pdf)

[Python中*args与**args的区别](http://bbs.chinaunix.net/thread-3572865-1-1.html)