---
categories:
  - python
date: '2017-01-07T22:08:51'
description: ''
tags:
  - Python
  - 函数
title: Python函数
---



# Python函数

函数是Python里组织代码的最小单元，Python函数包含以下几个部分：

- 定义函数
- 调用函数
- 参数
- 函数的返回值
- 函数的嵌套
- 作用域
- 函数执行流程
- 递归函数
- 匿名函数
- 生成器
- 高阶函数

<!--more-->

## 定义函数

```python
def add(x, y):     # 函数定义 def 表示定义一个函数， 紧接着是函数名 函数名后面用一对小括号列出参数列表，参数列表后面使用一个冒号开始函数体
    print(x + y)   # 函数体是正常的Python语句，可以包含任意结构
    return  x + y  # return 语句表示函数的返回值
```

函数是有输入(参数)和输出(返回值)的代码单元， 把输入转化为输出

## 调用函数

定义函数的时候,并不会执行函数体，　当调用函数的时候，才会执行其中的语句块

```python
In [1]: def add(x, y):     # 函数定义 def 表示定义一个函数， 紧接着是函数名 函数名后面用一对小括号
   ...:         print(x + y)   # 函数体是正常的Python语句，可以包含任意结构
   ...:         return  x + y  # return 语句表示函数的返回值
   ...: 

In [2]: add(3, 5) # 函数使用函数名来调用，函数名后紧跟一对小括号，小括号里传入函数定义时的参数
8
Out[2]: 8

In [3]: add(3, 4, 5) # 传入参数必须和函数定义时的参数相匹配，如果不匹配，会抛出TypeError
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-3-a11d83d1db7e> in <module>()
----> 1 add(3, 4, 5)

TypeError: add() takes 2 positional arguments but 3 were given
```

## 参数

### 传参方式

```python
In [5]: def add(x, y):
   ...:     ret = x + y
   ...:     print('{} + {} = {}'.format(x, y, x+y))
   ...:     return ret
   ...: 

In [6]: add(3, 5) #参数按照定义的顺序传入，这样的传参方法叫做位置参数
3 + 5 = 8
Out[6]: 8

In [7]: add(y=3, x=5) #参数按照定义时的变量名传递，这样的传参方法叫做关键字参数，关键字参数和顺序无关
5 + 3 = 8
Out[7]: 8

In [8]: add(5, y=3) # 位置参数和关键字参数可以混用
5 + 3 = 8
Out[8]: 8

In [9]: add(x=3, 5)	# 位置参数不能放在关键字参数的后面
  File "<ipython-input-9-165b39de39ac>", line 1
    add(x=3, 5)
            ^
SyntaxError: positional argument follows keyword argument


In [10]: add('3', '5')	# python是动态语言，传入的参数类型可以不固定
3 + 5 = 35
Out[10]: '35'

In [11]: add(3, '5') # python是强类型语言，传入的参数需要满足强类型要求，否则会抛出TypeError
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-11-335767c130e1> in <module>()
----> 1 add(3, '5')

<ipython-input-5-e720706d1634> in add(x, y)
      1 def add(x, y):
----> 2     ret = x + y
      3     print('{} + {} = {}'.format(x, y, x+y))
      4     return ret

TypeError: unsupported operand type(s) for +: 'int' and 'str'

```

### 参数默认值

参数可以有默认值，当一个参数有默认值时， 调用时如果不传递此参数，会使用默认值

```python
In [12]: def inc(x, y=1):	# 参数y默认为1
    ...:     x += y
    ...:     return x
    ...: 

In [13]: inc(3)	# 传参时只需要传入x即可
Out[13]: 4

In [14]: inc(3, 2)
Out[14]: 5

In [15]: def inc(x=1, y):	# 默认参数不能再非默认参数之前
    ...:     return x + y
  File "<ipython-input-15-993be842d592>", line 1
    def inc(x=1, y):
           ^
SyntaxError: non-default argument follows default argument


In [16]: def connect(host='127.0.0.1', port=3306, user='root', password='', dbname='test'):
    ...:     pass
    ...: 

In [17]: connect('192.168.110.13',password='123456')
```

参数默认值和关键字参数一起使用，会让代码非常简洁

### 可变参数

可变参数两种形式：

- 位置可变参数 ： 参数名前加**一个**星号， 构成**元组**， 传参只能以**位置参数**的形式
- 关键字可变参数： 参数名前加**两个**信号， 构成**字典**， 传参只能以**关键字参数**的形式

**位置可变参数**

```python
In [18]: def sum(*lst):
    ...:     print(type(lst))
    ...:     ret = 0
    ...:     for x in lst:
    ...:         ret += x
    ...:     return ret
    ...: 
# 参数前加一个星号， 表示这个参数是可变的， 也就是可以接受任意多个参数, 这些参数将构成一个元组， 此时只能通过位置参数传参
In [19]: sum(1, 2, 3)
<class 'tuple'>
Out[19]: 6

```

**关键字可变参数**

```python
In [20]: def connect(**kwargs):
    ...:     print(type(kwargs))
    ...:     for k, v in kwargs.items():
    ...:         print('{} => {}'.format(k, v))
    ...:         
# 参数前加两个星号， 表示这个参数是可变的，可以接受任意多个参数， 这些参数构成一个字典，此时只能通过关键字参数传参
In [21]: connect(host='127.0.0.1',port=3306)
<class 'dict'>
host => 127.0.0.1
port => 3306
```

**位置可变参数和关键字可变参数混合使用**

```python
In [22]: def fn(*args, **kwargs):
    ...:         print(args)
    ...:         print(kwargs)
    ...:     

In [23]: fn(1, 2, 3, a=4, b=5)
(1, 2, 3)
{'a': 4, 'b': 5}
# 以上说明位置可变参数和关键字可变参数可以混合使用

In [24]: def fn(**kwargs, *args): 
  File "<ipython-input-24-e42478d184b2>", line 1
    def fn(**kwargs, *args):
                   ^
SyntaxError: invalid syntax
# 以上说明当位置可变参数和关键字可变参数一起使用时， 位置可变参数必须在前面
```

**可变参数和普通参数混合使用**

普通参数可以和可变参数一起使用，但是传参的时候必须匹配，演示如下

```python
In [25]: def fn(x, y, *args, **kwargs):
    ...:         print(x)
    ...:         print(y)
    ...:         print(args)
    ...:         print(kwargs)
    ...:     

In [26]: fn(2, 3, 4, 5, 7, a=1, b=2)
2
3
(4, 5, 7)
{'a': 1, 'b': 2}

In [27]: fn(2, 3)
2
3
()
{}

In [28]: fn(2, 3, 4, 5, x=1)	# x有两个值，一个2，一个1，所以抛出TypeError
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-28-0f8d856dee50> in <module>()
----> 1 fn(2, 3, 4, 5, x=1)

TypeError: fn() got multiple values for argument 'x'

In [29]: fn(2, y=3)
2
3
()
{}
```

位置可变参数可以在普通参数之前， 但是在位置可变参数之后的普通参数变成了**keyword-only**参数:

```python
In [30]: def fn(*args, x):
    ...:     print(args)
    ...:     print(x)
    ...:     

In [31]: fn(2, 3, 4)
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-31-fab2f7df0315> in <module>()
----> 1 fn(2, 3, 4)

TypeError: fn() missing 1 required keyword-only argument: 'x'

In [32]: fn(2, 3, x=4)	# 必须将位置可变参数之后的普通参数变成keyword-only，否则TypeError
(2, 3)
4
```

关键字可变参数不允许在普通参数之前,演示如下：

```python
In [33]: def fn(**kwargs, x=5):
  File "<ipython-input-33-889f99c1c889>", line 1
    def fn(**kwargs, x=5):
                   ^
SyntaxError: invalid syntax
```

关于默认参数和可变参数的总结：

**通常**来说：

- 默认参数靠后
- 可变参数靠后
- 默认参数和可变参数一般不同时出现
- 当默认参数和可变参数一起出现的时候， 默认参数相当于普通参数

### 参数解构

参数解构有两种形式

- **一个星号** 解构的对象：可迭代对象 ，解构的结果：位置参数
- **两个星号** 解构的对象：字典 ，解构的结果：关键字参数

**一个星号的情况**

```python
In [34]: def add(x, y):
    ...:         ret = x + y
    ...:         print('{} + {} = {}'.format(x, y, ret))
    ...:         return ret
    ...: 

In [35]: add(1, 2)
1 + 2 = 3
Out[35]: 3

In [36]: add(x=1, y=2)
1 + 2 = 3
Out[36]: 3

In [37]: t = [1, 2]

In [38]: add(t[0], t[1])	# 如果列表中的元素很多的时候，一个一个解开很不方便简洁
1 + 2 = 3
Out[38]: 3

In [39]: add(*t)	# 位置参数解构  加一个星号， 可以把可迭代对象解构成位置参数
1 + 2 = 3
Out[39]: 3

In [40]: add(*range(2))
0 + 1 = 1
Out[40]: 1
```

**二个星号**

```python
In [42]: d = {'x': 1, 'y':2}

In [43]: add(**d)
1 + 2 = 3
Out[43]: 3
```

**参数解构发生在函数调用时， 可变参数发生函数定义时，所以两者并不冲突**

```python
In [46]: def sum(*args):	# 可变参数发生在函数定义时
    ...:     ret = 0
    ...:     for x in args:
    ...:         ret += x
    ...:     return ret
    ...: 

In [47]: sum(*range(10))	# 参数解构发生在函数调用时
Out[47]: 45

In [48]: def fn(**kwargs):
    ...:     print(kwargs)
    ...:     

In [49]: fn(**{'a-b':1})
{'a-b': 1}

In [50]: fn(**{123:1})	# 关键字参数解构， key必须是str
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-50-3c8b8b3fdf0b> in <module>()
----> 1 fn(**{123:1})

TypeError: fn() keywords must be strings
```

### keyword-only 参数

使用方法参见：[Python: 函数参数列表中单个星号的意思，Keyword-Only Arguments](https://www.polarxiong.com/archives/Python-%E5%87%BD%E6%95%B0%E5%8F%82%E6%95%B0%E5%88%97%E8%A1%A8%E4%B8%AD%E5%8D%95%E4%B8%AA%E6%98%9F%E5%8F%B7%E7%9A%84%E6%84%8F%E6%80%9D-Keyword-Only-Arguments.html)

星号可以以一个参数的形式出现在函数声明中的参数列表中，但星号之后的所有参数都必须有关键字（keyword），这样在函数调用时，星号*之后的所有参数都必须以`keyword=value`的形式调用，而不能以位置顺序调用。

使用示例如下：也可参考上面链接中的示例

```python
In [54]: def fn(*, x):
    ...:     print(x)
    ...:     

In [55]: fn(3)
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-55-f005f2a6106f> in <module>()
----> 1 fn(3)

TypeError: fn() takes 0 positional arguments but 1 was given

In [56]: fn(x=3)
3

In [57]: def fn(x, *, y):
    ...:     print(x)
    ...:     print(y)
    ...:     

In [58]: fn(1, y=2)
1
2

In [59]: fn(1, 2)
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-59-c159019d3516> in <module>()
----> 1 fn(1, 2)

TypeError: fn() takes 1 positional argument but 2 were given
```

## 函数的返回值

- return 语句除了返回值之外，还会结束函数， return之后的语句将不会被执行
- 一个函数可以有多个return语句， 执行到哪个return由哪个return返回结果并结束函数
- 函数中 return可以提前结束循环
- 当函数没有return语句的时候，返回None
- 当函数需要返回多个值时， 可以用封装把返回值封装成一个元组
- 可以通过解构获取到多返回值
- return None 可以简写为 return， 通常用于结束函数

```python
In [63]: def fn(x):
    ...:     for i in range(x):
    ...:         if i > 3:
    ...:             return i	# return可以提前退出循环
    ...:     else:
    ...:         print('not bigger than 3')
    ...:         

In [64]: fn(2)
not bigger than 3

In [65]: fn(10)	
Out[65]: 4

In [66]: def fn():
    ...:     pass	# 没有return时返回的是None
    ...: 

In [67]: ret = fn()

In [68]: ret

In [69]: type(ret)
Out[69]: NoneType

In [70]: def fn():
    ...:     return 3, 5	# 当函数需要返回多个值时， 会把返回值封装成一个元组
    ...: 

In [71]: ret = fn()

In [72]: type(ret)
Out[72]: tuple

In [73]: x, y = fn()	# 可以通过解构获取多个返回值
```

## 函数的嵌套

函数可以嵌套使用

```python
In [75]: def outter():
    ...:     def inner():
    ...:         print('inner')
    ...:     print('outter')
    ...:     inner()
    ...:     

In [76]: outter()
outter
inner
```

## 作用域

### 变量的作用域为定义此变量的作用域

```python
In [6]: def fn(): # 变量的作用域为定义此变量的作用域
   ...:         xx = 1
   ...:         print(xx)
   ...:     

In [7]: fn()
1

In [8]: xx
---------------------------------------------------------------------------
NameError                                 Traceback (most recent call last)
<ipython-input-8-102f5037fe64> in <module>()
----> 1 xx

NameError: name 'xx' is not defined
```

表明变量的作用域就在fn函数之中

### 上级作用域对下级作用域只读可见

不同作用域变量不可见， 但是下级作用域可以对上级作用域的变量只读可见

```python
In [9]: def fn():	# 上级作用域对下级作用域可见
   ...:     xx = 1
   ...:     print(xx)
   ...:     def inner():
   ...:         print(xx)
   ...:     inner()
   ...:     

In [10]: fn()
1
1

In [11]: def fn():	# 上级作用域对下级作用域只读可见
    ...:     xx = 1
    ...:     print(xx)
    ...:     def inner():
    ...:         xx = 2
    ...:     inner()
    ...:     print(xx)
    ...:     

In [12]: fn()
1
1	# 可以发现xx并没有被下级作用域修改
```

### 不要使用全局变量global

**除非你清楚的知道global会带来什么，并且明确的知道，非global不行， 否则不要使用global**

```python
In [13]: xx = 1

In [14]: def fn():
    ...:     global xx	# global 可以提升变量作用域为全局变量
    ...:     xx += 1
    ...:     

In [15]: fn()

In [16]: xx
Out[16]: 2
```

### 闭包函数

**闭包定义（Wikipedia）**：在一些语言中，在函数中可以（嵌套）定义另一个函数时，如果内部的函数引用了外部的函数的变量，则可能产生闭包。闭包可以用来在一个函数与一组“私有”变量之间创建关联关系。在给定函数被多次调用的过程中，这些私有变量能够保持其持久性

**通俗理解**：当某个函数被当成对象返回时，**夹带了外部变量**，就形成了一个闭包。

如果我们想实现一个无限增长的计数器，可以写一个counter函数，函数内部进行自增就行。假定我们按照以下写法：就会报错

```python
In [17]: def counter(): 
    ...:     c = 0
    ...:     def inc():
    ...:         c += 1 # c[0] = c[0] + 1
    ...:         return c
    ...: return inc
    ...: 

In [18]: f = counter()

In [19]: f()
---------------------------------------------------------------------------
UnboundLocalError                         Traceback (most recent call last)
<ipython-input-19-0ec059b9bfe1> in <module>()
----> 1 f()

<ipython-input-17-9dd4cd4942f6> in inc()
      2     c = 0
      3     def inc():
----> 4         c += 1 # c[0] = c[0] + 1
      5         return c
      6     return inc

UnboundLocalError: local variable 'c' referenced before assignment
```

在 python 的函数内，可以直接引用外部变量，但不能改写外部变量，因此如果在闭包中直接改写父函数的变量，就会发生错误。比如上述程序直接改写父函数中的变量`c`

python的闭包中如果想改写父函数的变量可以用可变容器实现，这也是python2实现的唯一方式

```python
In [1]: def counter():
   ...:     c=[0]
   ...:     def inc():
   ...:         c[0] += 1
   ...:         return c[0]
   ...:     return inc
   ...: 

In [2]: f = counter()

In [3]: f
Out[3]: <function __main__.counter.<locals>.inc>

In [4]: f()
Out[4]: 1

In [5]: f()
Out[5]: 2

In [6]: f()
Out[6]: 3
```

### nonlocal关键字

在python3中改写父变量还有一种方就是使用**`nonlocal`关键字**

nonlocal 关键字用于标记一个变量由他的上级作用域定义， 通过nonlocal标记的变量， 可读可写

```python
In [7]: def counter():
   ...:     c = 0
   ...:     def inc():
   ...:         nonlocal c
   ...:         c += 1
   ...:         return c
   ...:     return inc
   ...: 

In [8]: f = counter()

In [9]: f
Out[9]: <function __main__.counter.<locals>.inc>

In [10]: f()
Out[10]: 1

In [11]: f()
Out[11]: 2
```

如果上级没有定义nonlocal的变量，使用nonlocal时会抛出语法错误

```python
In [12]: def fn():
    ...:     nonlocal xxx
  File "<ipython-input-12-2d2b8104e945>", line 2
    nonlocal xxx
SyntaxError: no binding for nonlocal 'xxx' found
```

### 函数的`__defaults__`属性

可变参数和不可变参数的`__defaults__`属性不一样

**参数可变时**

当使用可变类型作为默认值参数默认值时，需要特别注意，会改变函数的`__default__`属性

```python
In [1]: def fn(xxyy=[]):
   ...:     xxyy.append(1)
   ...:     print(xxyy)
   ...:     

In [2]: fn()
[1]

In [3]: fn()
[1, 1]

In [4]: fn.__defaults__		# 参数是函数对象的属性
Out[4]: ([1, 1],)

In [5]: fn()
[1, 1, 1]

In [6]: fn.__defaults__ 	# 所有的函数参数封装成一个元组，第一个函数参数时列表在动态变化
Out[6]: ([1, 1, 1],)
```

**参数不可变时**

使用不可变类型作为默认值，函数体内不改变默认值

```python
In [8]: def fn(x=0, y=0):
   ...:         x = 3   # 赋值即定义
   ...:         y = 3   # 赋值即定义
   ...:     

In [9]: fn.__defaults__
Out[9]: (0, 0)

In [10]: fn()

In [11]: fn.__defaults__
Out[11]: (0, 0)
```

### 可变参数时None的使用

通常如果使用一个可变类型作为默认参数时， 会使用None来代替

```python
In [1]: def fn(lst=None):	# 向一个列表中插入元素3，列表默认为None
   ...:     if lst is None:
   ...:         lst = []
   ...:     lst.append(3)
   ...:     print(lst)
   ...:     

In [2]: fn.__defaults__		# 函数的__defaults__属性就是可变参数对应的None
Out[2]: (None,)

In [3]: fn()
[3]

In [4]: fn()				# 如果不传入值，函数执行的时候会先创建一个空列表，然后append
[3]

In [5]: fn.__defaults__
Out[5]: (None,)

In [6]: fn([1,2])
[1, 2, 3]

In [7]: fn.__defaults__		# 传入值之后，也不会改变函数的__default__属性
Out[7]: (None,)
```

### Python作用域、闭包、装饰器资料

- [Python 的闭包和装饰器](https://segmentfault.com/a/1190000004461404)


- [说说Python中的闭包 - Closure](https://segmentfault.com/a/1190000007321972)


- [Python Enclosing作用域、闭包、装饰器话聊上篇](https://segmentfault.com/a/1190000006236947)


- [Python Enclosing作用域、闭包、装饰器话聊下篇](https://segmentfault.com/a/1190000006659077)

## 函数执行流程

函数的执行过程就是压栈和出栈的过程。具体如下

当调用函数的时候， 解释器会把当前现场压栈，然后开始执行被调函数， 被调函数执行完成，解释器弹出当前栈顶，恢复现场

## 递归函数

递归函数的定义就是函数调用函数自身。

- 递归函数必须要有退出条件
- 为了保护解释器， Python对最大递归深度有限制
- 绝大多数递归都可以转化为循环使用
- 尽量避免使用递归
- sys模块中的getrecursionlimit和setrecursionlimit可以获取和设置最大递归深度

## 匿名函数

```python
In [1]: lambda x: x + 1
Out[1]: <function __main__.<lambda>>
```

匿名函数有以下特点

- lambda来定义
- 参数列表不需要用小括号
- 冒号不是用来开启新语句块
- 没有return，最后一个表达式的值即返回值
- 匿名函数（lambda表达式）只能写在一行上，所以也叫单行函数

匿名函数的好处是

- 函数没有名字，不必担心函数名冲突
- 匿名函数也是一个函数对象，可以把匿名函数返回给一个变量，再利用变量调用函数

```python
In [1]: lambda x: x + 1
Out[1]: <function __main__.<lambda>>

In [2]: f = lambda x: x + 1		# 直接把lambda函数返回给变量f

In [3]: f(3)					# 由变量f调用函数
Out[3]: 4

In [4]: f(5)
Out[4]: 6

In [5]: (lambda x: x * 2)(3)	# 第一对括号用来改变优先级 第二对括号表示函数调用
Out[5]: 6

In [6]: (lambda : 1)()			# lambda表示式参数可以为空
Out[6]: 1

In [7]: (lambda x, y: x + y)(3, 5)	# lambda表达式的位置参数
Out[7]: 8

In [8]: (lambda *args: args)(*range(3))	# lambda表达式的位置可变参数
Out[8]: (0, 1, 2)

In [9]: (lambda *args, **kwargs: print(args, kwargs))(*range(3), **{str(x):x for x in range(3)})	# lambda表达式的位置可变参数和关键字可变参数
(0, 1, 2) {'0': 0, '1': 1, '2': 2}

In [10]: (lambda *, x: x)(x=3)	# *号后面的位置参数必须使用关键字参数
Out[10]: 3
```

> 普通函数所支持的参数的变化，匿名函数都支持

匿名函数的常见用法：通常用于高阶函数的参数， 当此函数非常短小的时候，就适合使用匿名函数

比如匿名函数可以作为`sorted`函数的自定义键函数（custom key function）

```python
In [11]: help(sorted)
    Help on built-in function sorted in module builtins:

	sorted(iterable, key=None, reverse=False)
    	Return a new list containing all items from the iterable in ascending order.

    	A custom key function can be supplied to customise the sort order, and the
    	reverse flag can be set to request the result in descending order.

In [12]: from collections import namedtuple

In [13]: point = namedtuple('point',['x','y'])	# 定义命名元组point

In [14]: points = [point(1, 2), point(4, 3), point(8, 9)]

In [15]: def getY(point):
    ...:     return point.y
    ...: 

In [16]: sorted(points, key=getY)	# 简短的函数可以作为自定义键函数
Out[16]: [point(x=1, y=2), point(x=4, y=3), point(x=8, y=9)]

In [17]: sorted(points, key=lambda x: x.y)	# lambda表示也可以作为自定义键函数
Out[17]: [point(x=1, y=2), point(x=4, y=3), point(x=8, y=9)]
```

## 高阶函数

### 高阶函数的定义

高阶函数英文叫Higher-order function。

在数学和计算机科学中，**高阶函数**是至少满足下列一个条件的函数：

- 接受一个或多个函数作为输入：通常用于大多数逻辑固定，少部分逻辑不固定的场景
- 输出一个函数：函数作为返回值： 通常是用于闭包的场景， 需要封装一些变量

常见的高阶函数有map,reduce,filter

### 高阶函数:插入排序

插入排序时，排序顺序分为升序和降序，我们可以使用一个函数作为插入排序函数的参数来控制是升序还是降序。

首先看一下按照升序插入排序，然后再改进成升序降序可控的插入排序

```python
def insertSort(iter):
    ret = []
    for x in iter:
        for i, y in enumerate(ret):
            if x < y:				# 修改处
                ret.insert(i, x)
                break
        else:
            ret.append(x)
    return ret
```

如果想让这个函数降序排序，则只需要修改代码中的注释处，改成`x > y`即可

如果传入一个函数来控制if后面的bool值，则就实现了通过参数控制升降了

```python
def insertSort(iter, cmp = lambda x, y: x < y):
    ret = []
    for x in iter:
        for i, y in enumerate(ret):
            if cmp(x, y):
                ret.insert(i, x)
                break
        else:
            ret.append(x)
    return ret
```

这个函数就默认为升序排序了，但是可以传入一个比较函数变成降序，如下

```python
lst = insertSort([1, 3, 2, 4, 6, 8, 5],lambda x, y: x > y)
```

### map

`map()`函数原型：**`map(func, *iterables) --> map object`**

`map()`函数接收两个参数，一个是函数`func`，一个是可迭代对象`Iterable`，`map`将传入的函数依次作用到可迭代对象的每个元素，并把结果放入**`map对象`**这个迭代器中。所以`map`函数是高阶函数。

`map`类中存在`__iter__`和`__next__`函数

**map使用示例**

把list中的所有数字的平方

```python
In [1]: def f(x):								# 定义平方函数f
   ...:         return x ** 2
   ...: 

In [2]: ret = map(f, [1, 2, 3, 4, 5, 6, 7])		# 函数f和列表作为map的参数

In [3]: ret										# map的返回值只是一个返回值
Out[3]: <map at 0x7f2d539a7470>

In [4]: next(ret)								# 可以用next方法输出map的结果
Out[4]: 1

In [5]: next(ret)
Out[5]: 4

In [6]: lst = list(ret)							# 也可以用list函数计算出所有的值

In [7]: lst
Out[7]: [9, 16, 25, 36, 49]
```

### reduce

`map`函数是`map`类的函数，但是reduce函数属于`functools`包的`reduce`模块中

```python
from functools import reduce
```

然后可以使用`help`方法查看`reduce`函数的使用

```python
help(reduce)
```

输出结果如下

```
Help on built-in function reduce in module _functools:

reduce(...)
    reduce(function, sequence[, initial]) -> value

    Apply a function of two arguments cumulatively to the items of a sequence,
    from left to right, so as to reduce the sequence to a single value.
    For example, reduce(lambda x, y: x+y, [1, 2, 3, 4, 5]) calculates
    ((((1+2)+3)+4)+5).  If initial is present, it is placed before the items
    of the sequence in the calculation, and serves as a default when the
    sequence is empty.
```

**reduce使用示例**

- 输出1到10的和

```python
def add(x,y): 
    return x + y
print(reduce(add, range(1, 11)))
```

输出结果为55

- 把字符串转化为`int`，不适用`int()`函数

```python
def str2int(s):
    def char2num(c):
        return {'0': 0, '1': 1, '2': 2 ,'3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}[c]
    def f(x, y):
        return 10 * x + y 
    return reduce(f, map(char2num, s))
```

> str2int('1234321')  => 1234321

### filter

`help(filter)`之后可以发现`filter`是一个类，其中有一个`filter`函数，原型如下

```python
filter(function or None, iterable) --> filter object
```

和`map()`类似，`filter()`也接收一个函数和一个序列。和`map()`不同的是，`filter()`把传入的函数依次作用于每个元素，然后根据返回值是`True`还是`False`决定保留还是丢弃该元素。返回值也是一个迭代器。

**filter使用示例**

使用filter筛选出list中的回文数

```python
def is_palindrome(n):
    m = str(n)
    for i in range(len(m)//2):
        if m[i] != m[len(m) - i -1]:
            return False
    else:
        return True

lst = list(filter(is_palindrome, [12321, 194, 13431]))
print(lst)
# 结果: [12321, 13431]
```

所以`filter()`函数用于过滤序列，重点在于选择一个正确的筛选函数。

## 生成器

带yield语句的函数称之为生成器函数， 生成器函数的返回值是生成器

- 生成器函数执行的时候，不会执行函数体
- 当next生成器的时候， 当前代码执行到之后的第一个yield，会弹出值，并且暂停函数
- 当再次next生成器的时候，从上次暂停处开始往下执行
- 当没有多余的yield的时候，会抛出StopIteration异常，异常的value是函数的返回值

### 生成器的基本形式

```python
In [1]: def g():
    ...:     for x in range(5):
    ...:         yield x	# 弹出x
    ...:         

In [2]: r = g()		# 函数调用完成之后函数现场并没有被销毁

In [3]: r
Out[3]: <generator object g at 0x7f0e18543990>

In [4]: next(r)
Out[4]: 0

In [5]: next(r)
Out[5]: 1

In [6]: for x in r:
    ...:     print(x)
    ...:     
2
3
4
```

### 生成器的执行顺序

```python
In [1]: def g():
   ...:     print('a')
   ...:     yield 1
   ...:     print('b')
   ...:     yield 2
   ...:     return 3
   ...: 

In [2]: r = g()	# 执行生成器函数的时候函数并没有被执行 

In [3]: next(r)	# 执行到第一个yield就停止执行
a
Out[3]: 1

In [4]: next(r)	# 执行到第二个yield就停止执行
b
Out[4]: 2

In [5]: next(r)	# 从第二个yield开始，当没有更多yield的时候，抛出StopIteration异常，异常的值正好是return的返回值
---------------------------------------------------------------------------
StopIteration                             Traceback (most recent call last)
<ipython-input-5-0b5056469c9c> in <module>()
----> 1 next(r)

StopIteration: 3
```

### 生成器的应用

**计数器第一种形式**

```python
In [1]: def counter():
   ...:     x = 0
   ...:     while True:
   ...:         x += 1
   ...:         yield x	# 每次将+1之后的x弹出
   ...:         

In [2]: def inc(c):
   ...:     return next(c)
   ...: 

In [3]: c = counter()	# counter函数执行的结果就是一个生成器，所以c就是生成器

In [4]: inc(c)
Out[4]: 1

In [5]: inc(c)
Out[5]: 2
```

**计数器第二种形式**

```python
In [6]: def make_inc():
   ...:     def counter():
   ...:         x = 0
   ...:         while True:
   ...:             x += 1
   ...:             yield x
   ...:     c = counter()
   ...:     return lambda : next(c)	# 使用lambda表达式将next(c)作为函数返回，而不是只返回一个next(c)
   ...: 

In [7]: make_inc()
Out[7]: <function __main__.make_inc.<locals>.<lambda>>	# make_inc本质是一个匿名函数

In [8]: inc = make_inc()

In [9]: inc()
Out[9]: 1

In [10]: inc()
Out[10]: 2
```

**斐波拉契数列**

```python
In [11]: def fib():
    ...:     a = 1
    ...:     b = 1
    ...:     while True:
    ...:         yield a
    ...:         a, b = b, a + b
    ...:         

In [12]: fib()
Out[12]: <generator object fib at 0x7f9ff2746830>

In [13]: f = fib()	# 生成器f

In [15]: next(f)
Out[15]: 1

In [16]: next(f)
Out[16]: 1

In [17]: next(f)
Out[17]: 2

In [18]: next(f)
Out[18]: 3

In [19]: g = fib()

In [20]: ret = []	# 将yield的值都保存在ret中

In [21]: for _ in range(1000):	# 遍历生成器
    ...:     ret.append(next(g))
    ...:     

In [22]: ret[-1]	# 取ret列表的最后一个元素值，速度很快
Out[22]: 43466557686937456435688527675040625802564660517371780402481729089536555417949051890403879840079255169295922593080322634775209689623239873322471161642996440906533187938298969649928516003704476137795166849228875
```

**生成器的高级用法**

生成器的高级用法是**协程**

- 协程：协程运行在一个线程之内， 在用户态调度

### 生成器参考资料

- [python生成器到底有什么优点？](https://www.zhihu.com/question/24807364)

- [Understanding Generators in Python](http://stackoverflow.com/questions/1756096/understanding-generators-in-python)

- [Introduction to Python Generators](http://intermediatepythonista.com/python-generators)


- [生成器资料汇总](http://fullstackpython.atjiang.com/generators.html)