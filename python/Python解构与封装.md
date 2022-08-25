---
categories:
  - python
date: '2016-12-23T23:45:40'
description: ''
tags:
  - Python
  - 解构
  - 封装
title: Python解构与封装
---




# Python解构与封装

## 提出问题

先看以下代码

```python
x = 1
y = 2

tmp = x
x = y
y = tmp

print(x, y)
```

代码的输出结果是：2 1

再看以下代码：

```python
x = 1
y = 2

x, y = y, x
print(x, y)
```

代码的输出结果是：2 1

`x, y = y, x`这段代码背后的含义就是解构和封装

<!--more-->

## Python封装

```python
In [1]: t = 1, 2

In [2]: t
Out[2]: (1, 2)

In [3]: type(t)
Out[3]: tuple			# 定义元组是可以省略小括号的

In [4]: t1 = (1, 2)

In [5]: t2 = 1, 2
						# t1和t2等效
In [6]: t1
Out[6]: (1, 2)

In [7]: t2
Out[7]: (1, 2)

```

所以**封装出来的结果一定是元组**。

`x, y = y, x`这段代码的右侧就会封装成(y, x)

## Python解构

### 基本解构

```python
In [8]: lst = [1, 2]

In [9]: first, second = lst

In [10]: print(first, second)
1 2
```

按照元素顺序，把线性结构lst的元素赋给变量first,second

### 加星号解构

```python
In [11]: lst = list(range(5))

In [12]: head, *tail = lst

In [13]: head
Out[13]: 0

In [14]: tail
Out[14]: [1, 2, 3, 4]

In [15]: *lst2 = lst	# 左边必须有一个加星号的变量
  File "<ipython-input-15-98211a44ccfb>", line 1
    *lst2 = lst
               ^
SyntaxError: starred assignment target must be in a list or tuple


In [16]: *head, tail = lst

In [17]: head
Out[17]: [0, 1, 2, 3]

In [18]: lst
Out[18]: [0, 1, 2, 3, 4]

In [19]: tail
Out[19]: 4

In [20]: head, *m1, *m2, tail = lst		# 星号不能有多个只能有一个
  File "<ipython-input-20-1fc1a52caa8e>", line 1
    head, *m1, *m2, tail = lst
                              ^
SyntaxError: two starred expressions in assignment


In [21]: v1, v2, v3, v4, v5, v6, v7 = lst	# 左边变量数不能超过右边元素数
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-21-9366cfb498a1> in <module>()
----> 1 v1, v2, v3, v4, v5, v6, v7 = lst

ValueError: not enough values to unpack (expected 7, got 5)

In [22]: v1, v2 = lst						#左边变量数不能少于右边元素数
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-22-d7b0a4e7871e> in <module>()
----> 1 v1, v2 = lst

ValueError: too many values to unpack (expected 2)
```

总结为以下规律：

- 元素按照顺序赋值给变量
- 变量和元素必须匹配
- 加星号变量，可以接受任意个数的元素
- 加星号的变量不能单独出现

### 多层次解构

**解构是支持多层次的**

```python
In [23]: lst = [1, (2, 3), 5]

In [24]: _, v, *_ = lst		# v解析成(2, 3)

In [25]: v
Out[25]: (2, 3)

In [26]: _, val = v			# v可以进一步解构

In [27]: val
Out[27]: 3

In [28]: _, (_, val), *_ = lst		# 可以一步一次性解构

In [29]: val
Out[29]: 3

In [30]: _, [*_, val], *_ = lst		# 中间部分解构成列表

In [31]: val
Out[31]: 3

In [32]: _, _, val, *_ = lst		# (2, 3)解析成第二个_

In [33]: val
Out[33]: 5

```

### Python下划线的使用

使用单个下划线 _ 表示丢弃该变量，这是Python的一个惯例。单个下划线也是Python合法的标识符， 但是如果不是要丢弃一个变量，通常不要用单个下划线表示一个有意义的变量。可以理解为约定俗成。

### 解构与封装的使用

非常复杂的数据结构，多层嵌套的线性结构的时候，可以用解构快速提取其中的值，非常的便利

比如以下的使用方法

```python
In [1]: key, _, value = 'I love Python'.partition(' love ')

In [2]: key
Out[2]: 'I'

In [3]: value
Out[3]: 'Python'
```