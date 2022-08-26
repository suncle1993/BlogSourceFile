---
abbrlink: 2607945880
alias: 2017/01/11/Python解析式/index.html
categories:
- python
date: '2017-01-11T22:44:30'
description: ''
tags:
- Python
- 解析式
title: Python解析式
---









# Python解析式

在python中经常能够看到形如`ret = [x ** 2 for x in lst]`这样的赋值语句，对于从C++转到python的人不太容易理解这种for循环的使用，这就是python为了简洁而发明的新语法。python解析式有以下优点：

- 代码简洁，可读性强
- 效率比普通迭代稍高

python的解析式具体分为以下四种：

- 列表解析式
- 生成器解析式
- 集合解析式
- 字典解析式

下面分别介绍这四种解析式的使用。

<!--more-->

## 列表解析式

**列表解析式的形式**

- `[expr for e in iterator]`

```python
In [1]: lst = range(10)

In [2]: %%timeit
   ...: ret = [x ** 2 for x in lst]
   ...: 
100000 loops, best of 3: 5.28 µs per loop

In [3]: %%timeit
   ...: ret = []
   ...: for x in lst:
   ...:     ret.append(x ** 2)
   ...: 
100000 loops, best of 3: 6.09 µs per loop	# 耗时稍高
```

可以发现效率是要稍高一点，最主要的还是代码简洁。

**列表解析式可以和`if`语句一起使用**

例如筛选出列表`lst`中的偶数：

```python
In [4]: ret = []

In [5]: for x in lst:
   ...:     if x % 2 == 0:
   ...:         ret.append(x)	# 使用for循环
   ...:         

In [6]: ret
Out[6]: [0, 2, 4, 6, 8]

In [7]: ret = [x for x in lst if x % 2 == 0]	# 使用列表解析式

In [8]: ret
Out[8]: [0, 2, 4, 6, 8]
```

列表解析式可以像`for`循环一样使用`if`语句。

- 带多个if语句的，都可以转化为条件的逻辑运算， 所以一般来说，不会带多个if语句

列表解析式的`for`语句可以嵌套。

```python
In [9]: (x, y) for x in range(0, 5) for y in range(5, 10)
  File "<ipython-input-9-825e2443da8b>", line 1
    (x, y) for x in range(0, 5) for y in range(5, 10)
             ^
SyntaxError: invalid syntax
# 说明列表解析式一定要使用中括号括起来

In [10]: [(x, y) for x in range(5) for y in range(5, 10)]
Out[10]: 
[(0, 5),
 (0, 6),
 (0, 7),
 (0, 8),
 (0, 9),
 (1, 5),
 (1, 6),
 (1, 7),
 (1, 8),
 (1, 9),
 (2, 5),
 (2, 6),
 (2, 7),
 (2, 8),
 (2, 9),
 (3, 5),
 (3, 6),
 (3, 7),
 (3, 8),
 (3, 9),
 (4, 5),
 (4, 6),
 (4, 7),
 (4, 8),
 (4, 9)]

In [11]: ret = []

In [12]: for x in range(5):
    ...:     for y in range(5, 10):
    ...:         ret.append((x, y))
    ...:         

In [13]: ret
Out[13]: 
[(0, 5),
 (0, 6),
 (0, 7),
 (0, 8),
 (0, 9),
 (1, 5),
 (1, 6),
 (1, 7),
 (1, 8),
 (1, 9),
 (2, 5),
 (2, 6),
 (2, 7),
 (2, 8),
 (2, 9),
 (3, 5),
 (3, 6),
 (3, 7),
 (3, 8),
 (3, 9),
 (4, 5),
 (4, 6),
 (4, 7),
 (4, 8),
 (4, 9)]
```

**`if`语句的特殊用法**

单行if语句的写法和列表解析式很像。

表达式形式：`x if cond else y`

`if`和`else`必须同时存在。

下面以偶数求平方，奇数求立方为例进行演示

```python
In [14]: ret = []

In [15]: for x in lst:
    ...:     if x % 2 == 0:
    ...:         ret.append(x ** 2)
    ...:     else:
    ...:         ret.append(x ** 3)
    ...:         

In [16]: ret
Out[16]: [0, 1, 4, 27, 16, 125, 36, 343, 64, 729]

In [17]: x = 3
# if特殊用法
In [18]: x ** 2 if x % 2 == 0 else x ** 3
Out[18]: 27

In [19]: 3 if True else 4
Out[19]: 3
# 如果采用if特殊用法配合列表解析式 x if cond else y for ...
In [20]: [x ** 2 if x % 2 == 0 else x ** 3 for x in lst]
Out[20]: [0, 1, 4, 27, 16, 125, 36, 343, 64, 729]
```

## 生成器解析式

列表解析式返回的是一个列表，而生成器解析式返回的是一个解析式。列表解析式的中括号变成小括号就是生成器解析式了

```python
In [1]: range(10000)
Out[1]: range(0, 10000)

In [2]: g = (x ** 2 for x in range(100000000000))

In [3]: g
Out[3]: <generator object <genexpr> at 0x7f9f08a5f0a0>

In [4]: next(g)
Out[4]: 0

In [5]: next(g)
Out[5]: 1

In [6]: next(g)
Out[6]: 4
```

列表解析式和生成器解析式的选择

- 需要用下标访问的时候，用列表解析式
- 只需要对结果迭代的时候，优先使用生成器解析式

## 集合解析式

将列表解析式的中括号换成大括号就是集合解析式了。

```python
In [1]: lst = [2, 4, 5, 6, 3, 4, 2]

In [2]: s = {x for x in lst}

In [3]: s
Out[3]: {2, 3, 4, 5, 6}	# 可见列表解析式生成的时候会去掉重复，符合集合要求

In [4]: type(s)
Out[4]: set
```

## 字典解析式

字典解析式使用的也是大括号，但是和集合解析式不同的是在`expr`处使用的不是单个元素而是`k,v`对。

```python
In [1]: {str(x): x for x in range(5)}
Out[1]: {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4}
```

这四种解析式中使用最广泛的还是列表解析式，会经常有一些很巧妙的用法。