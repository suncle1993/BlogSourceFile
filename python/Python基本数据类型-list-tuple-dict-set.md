---
abbrlink: 2639636733
alias: 2016/09/02/Python基本数据类型-list-tuple-dict-set/index.html
categories:
- python
date: '2016-09-02T21:20:51'
description: ''
tags:
- Python
- 列表
- 元组
- 字典
- 集合
title: Python基本数据类型-list-tuple-dict-set
---








# Python基本数据类型-list-tuple-dict-set

| 数据类型  | 表示方法        | 特性                                       |
| ----- | ----------- | ---------------------------------------- |
| list  | 列表用方括号表示：[] | list是一种有序的集合，可以随时添加和删除其中的元素。和C++数组的区别就是类型可不同。 |
| tuple | 元组用圆括号表示：() | 和list相比唯一的差异在于元组是只读的，不能修改。               |
| dict  | 字典用花括号表示：{} | 列表是有序的对象结合，字典是无序的对象集合。两者之间的区别在于：字典当中的元素是通过键来存取的，而不是通过偏移存取。 |
| set   | set()       | 集合是一个无序不重复元素集，基本功能包括关系测试和消除重复元素          |

<!--more-->

## 列表list

### 初始化列表

**指定元素初始化列表**

```python
>>> num=['aa','bb','cc',1,2,3]
>>> print num
['aa', 'bb', 'cc', 1, 2, 3]
```

**从字符串初始化列表**

```python
>>> a='oiawoidhoawd97192048f'
>>> num=list(a)
>>> print num
['o', 'i', 'a', 'w', 'o', 'i', 'd', 'h', 'o', 'a', 'w', 'd', '9', '7', '1', '9', '2', '0', '4', '8', 'f']
```

**从元组初始化列表**

```python
>>> a=(1,2,3,4,5,6,7,8)
>>> num=list(a)
>>> print num
```

**创建一个空列表**

```python
>>> num=[]
>>> print num
[]
```

**用某个固定值初始化列表**

```python
>>> initial_value=0
>>> list_length=5
>>> sample_list=[initial_value]*list_length
>>> print sample_list
[0, 0, 0, 0, 0]
>>> sample_list=[initial_value for i in range(10)]
>>> print sample_list
[0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
```

### 访问列表

**访问单个元素**

```python
>>> num=[0,1,2,3,4,5,6,7]
>>> num[3]
3
>>> num=[0,1,2,3,4,5,6,7]
>>> num[0]
0
>>> num[-1]
7
>>> num[-3]
5
```

**遍历整个列表**

```python
num=[0,1,2,3,4,5,6,7]
for a in num:
    print a,

for i in range(len(num)):
    print num[i],
```

输出结果：0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7

### 列表操作

**更新列表**

```python
>>> num=[0,1,2,3,4,5,6,7]
>>> num[1]='abc'
>>> print num
[0, 'abc', 2, 3, 4, 5, 6, 7]
```

**删除列表元素**

```python
num=[0,1,2,3,4,5,6,7]
for i in range(len(num)):
    print num[i],
del num[2]
print num
```

输出结果：0 1 2 3 4 5 6 7 [0, 1, 3, 4, 5, 6, 7]

**列表操作符+***

列表对+和*的操作符与字符串相似。+号用于组合列表，*号用于重复列表。

以下为+操作符

```python
>>> a=['a','b','c']
>>> b=['d','e','f']
>>> c=a+b
>>> print c
['a', 'b', 'c', 'd', 'e', 'f']
```

以下为*操作符

```python
>>> a=['a','b','c']
>>> c=a*4
>>> print c
['a', 'b', 'c', 'a', 'b', 'c', 'a', 'b', 'c', 'a', 'b', 'c']
```

### 列表函数

以下是列表相关函数的分类

![列表分类](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/python/xmind/%E5%88%97%E8%A1%A8%E6%93%8D%E4%BD%9C.png)

xmind文件可点[这里](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/file/xmind/%E5%88%97%E8%A1%A8%E6%93%8D%E4%BD%9C.xmind)下载

以下是`help(list)`的结果中关于重点函数的介绍部分

```python
Help on list object:

class list(object)
 |  list() -> new empty list
 |  list(iterable) -> new list initialized from iterable's items
 |  
 |  Methods defined here:
 |  
 |  append(...)
 |      L.append(object) -> None -- append object to end
 |  
 |  clear(...)
 |      L.clear() -> None -- remove all items from L
 |  
 |  copy(...)
 |      L.copy() -> list -- a shallow copy of L
 |  
 |  count(...)
 |      L.count(value) -> integer -- return number of occurrences of value
 |  
 |  extend(...)
 |      L.extend(iterable) -> None -- extend list by appending elements from the iterable
 |  
 |  index(...)
 |      L.index(value, [start, [stop]]) -> integer -- return first index of value.
 |      Raises ValueError if the value is not present.
 |  
 |  insert(...)
 |      L.insert(index, object) -- insert object before index
 |  
 |  pop(...)
 |      L.pop([index]) -> item -- remove and return item at index (default last).
 |      Raises IndexError if list is empty or index is out of range.
 |  
 |  remove(...)
 |      L.remove(value) -> None -- remove first occurrence of value.
 |      Raises ValueError if the value is not present.
 |  
 |  reverse(...)
 |      L.reverse() -- reverse *IN PLACE*
 |  
 |  sort(...)
 |      L.sort(key=None, reverse=False) -> None -- stable sort *IN PLACE*
 |  
 |  ----------------------------------------------------------------------
 |  Data and other attributes defined here:
 |  
 |  __hash__ = None
```

## 元组tuple

Python的元组与列表类似，不同之处在于元组的元素不能修改；元组使用小括号()，列表使用方括号[]；元组创建很简单，只需要在括号中添加元素，并使用逗号(,)隔开即可。

### 元组初始化

```python
>>> t = (1, 2, 3)
>>> print(t)
(0, 1, 2)
>>> t = tuple(range(3))
>>> print(t)
(0, 1, 2)
```

### 元组函数

关于tuple相关的函数可以使用help命令获得。

```python
help(tuple)
```

```python
Help on class tuple in module builtins:

class tuple(object)
 |  tuple() -> empty tuple
 |  tuple(iterable) -> tuple initialized from iterable's items
 |  
 |  If the argument is a tuple, the return value is the same object.
 |  
 |  Methods defined here:
 |  
 |  count(...)
 |      T.count(value) -> integer -- return number of occurrences of value
 |  
 |  index(...)
 |      T.index(value, [start, [stop]]) -> integer -- return first index of value.
 |      Raises ValueError if the value is not present.
```

list和index方法的使用和list一模一样。

### 命名元组

Python有一个类似tuple的容器namedtuples（命名元组），位于collection模块中。namedtuple是继承自tuple的子类，可创建一个和tuple类似的对象，而且对象拥有可访问的属性。

在c/c++中，对应的数据类型是结构体struct。

```c++
struct Point//声明一个结构体类型Point,代表一个点
{
    int x;  //包括一个整型变量x
    int y;  //包括一个整型变量y
};  //最后有一个分号
```

这样就声明了一个新的结构体类型Point，有了类型就可以定义结构体的变量了。

```C++
Point p1,p2;
```

在c/c++中结构体的最大作用在于组织数据，也就是对数据的封装（可以把结构体理解为特殊的类）。在python中起相同作用的就是命名元组了。命名元祖的具体使用如下

```python
>>> from collections import namedtuple	#依赖collections包的namedtuple模块
>>> Point = namedtuple('Point', 'x,y')
>>> p1 = Point(11, y=22)
>>> p1
Point(x=11, y=22)
>>> type(p1)
__main__.Point
>>> p1.x
11
>>> p1.y
22
>>> p1[0] + p1[1]
33
>>> a, b = p1
>>> a, b
(11, 22)
```

命名元祖的具体使用可以参见：[`namedtuple()`](https://docs.python.org/3/library/collections.html#collections.namedtuple)以及[python 命名元组](https://blog.csdn.net/article/details?id=52183211)

## 字典dict

字典相关的所有内容如下

![字典小结](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/python/xmind/%E5%AD%97%E5%85%B8%E5%8F%8A%E5%85%B6%E6%93%8D%E4%BD%9C.png)

xmind文件可点[这里](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/file/xmind/%E5%AD%97%E5%85%B8%E5%8F%8A%E5%85%B6%E6%93%8D%E4%BD%9C.xmind)下载

```python
help(dict)
```

可以发现，dict是python內建的类，是一种key-value结构

```python
Help on class dict in module __builtin__:

class dict(object)
 |  dict() -> new empty dictionary
 |  dict(mapping) -> new dictionary initialized from a mapping object's
 |      (key, value) pairs
 |  dict(iterable) -> new dictionary initialized as if via:
 |      d = {}
 |      for k, v in iterable:
 |          d[k] = v
 |  dict(**kwargs) -> new dictionary initialized with the name=value pairs
 |      in the keyword argument list.  For example:  dict(one=1, two=2)
 |  
 |  Methods defined here:
```

字典(dictionary)是除列表之外python中最灵活的内置数据结构类型。列表是有序的对象结合，**字典是无序的对象集合**。两者之间的区别在于：字典当中的元素是通过键来存取的，而不是通过偏移存取。

```python
dict = {'Alice': '2341', 'Beth': '9102', 'Cecil': '3258'};
```

每个键与值必须用冒号隔开(:)，每对用逗号分割，整体放在花括号中({})。键必须独一无二，但值则不必；值可以取任何数据类型，但必须是不可变的，如字符串，数或元组。

### 字典初始化

```python
In [1]: d = {}		#{}被字典占用了，所以set不能按照这个初始化

In [2]: type(d)
Out[2]: dict

In [3]: d = dict()

In [4]: d = {'a':1, 'b':2}

In [5]: d = dict([('a', 1), ('b', 2)])	#可接受以元组为元素的列表

In [6]: d
Out[6]: {'a': 1, 'b': 2}

In [7]: d = dict.fromkeys(range(5))		# 传入的可迭代元素为key， 值为None

In [8]: d
Out[8]: {0: None, 1: None, 2: None, 3: None, 4: None}

In [9]: d = dict.fromkeys(range(5), 'abc')	# 传入的可迭代元素为key， 值为'abc'

In [10]: d
Out[10]: {0: 'abc', 1: 'abc', 2: 'abc', 3: 'abc', 4: 'abc'}
```

### 增加

`d[key] = value`,`update`

```python
In [10]: d
Out[10]: {0: 'abc', 1: 'abc', 2: 'abc', 3: 'abc', 4: 'abc'}

In [11]: d['a'] = 1		# 可以直接使用key作为下标， 对某个不存在的下标赋值，会增加kv对

In [12]: d
Out[12]: {0: 'abc', 1: 'abc', 2: 'abc', 3: 'abc', 4: 'abc', 'a': 1}

In [13]: d.update([('c',3),('p',0)])	# update 传入的参数需要和dict保持一致

In [14]: d
Out[14]: {0: 'abc', 1: 'abc', 2: 'abc', 3: 'abc', 4: 'abc', 'p': 0, 'c': 3, 'a': 1}

In [15]: d.update([('c', 4), ('p', 4)])	#对已经存在的update时会进行修改，通常用于合并字典

In [16]: d
Out[16]: {0: 'abc', 1: 'abc', 2: 'abc', 3: 'abc', 4: 'abc', 'p': 4, 'c': 4, 'a': 1}
```

### 删除

- pop 用于从字典删除一个key， 并返回其value，当删除不存在的key的时候， 会抛出KeyError。当删除不存在的key， 并且指定了默认值时， 不会抛出KeyError， 会返回默认值
- popitem **随机** 返回并删除一个kv对的二元组
- clear 清空一个字典
- del语句

```python
In [16]: d
Out[16]: {0: 'abc', 1: 'abc', 2: 'abc', 3: 'abc', 4: 'abc', 'p': 4, 'c': 4, 'a': 1}

In [17]: help(d.pop)
Help on built-in function pop:

pop(...) method of builtins.dict instance
    D.pop(k[,d]) -> v, remove specified key and return the corresponding value.
    If key is not found, d is returned if given, otherwise KeyError is raised

In [18]: d.pop(0)	# 删除一个key，并且返回对应的value
Out[18]: 'abc'

In [19]: d
Out[19]: {1: 'abc', 2: 'abc', 3: 'abc', 4: 'abc', 'p': 4, 'c': 4, 'a': 1}

In [20]: d.pop(0)	# 如果要删除的key不存在，则抛出KeyError
---------------------------------------------------------------------------
KeyError                                  Traceback (most recent call last)
<ipython-input-20-e1702b259b84> in <module>()
----> 1 d.pop(0)

KeyError: 0

In [21]: d.pop(0, 'default')	# 如果给定default，则删除不存在的key时会返回default
Out[21]: 'default'

In [22]: d.pop(1, 'default')	# 给定的default对存在的key不会产生影响
Out[22]: 'abc'

In [23]: d
Out[23]: {2: 'abc', 3: 'abc', 4: 'abc', 'p': 4, 'c': 4, 'a': 1}
```

### 访问

#### 单个元素的访问

- 通过key直接访问
- 通过get函数访问

```python
In [1]: d = {'r':2, 'd':2, 'c':3, 'p':0}

In [2]: d
Out[2]: {'c': 3, 'd': 2, 'p': 0, 'r': 2}

In [3]: d['p']
Out[3]: 0

In [4]: d['c']
Out[4]: 3

In [5]: d['a']
---------------------------------------------------------------------------
KeyError                                  Traceback (most recent call last)
<ipython-input-5-169a40407b7f> in <module>()
----> 1 d['a']

KeyError: 'a'

In [6]: d.get('d')
Out[6]: 2

In [7]: d.get('a')

In [8]: d.get('a','default')
Out[8]: 'default'

In [9]: help(d.setdefault)


In [10]: d.setdefault('c','default')
Out[10]: 3

In [11]: d.setdefault('a','default')
Out[11]: 'default'

In [12]: d
Out[12]: {'a': 'default', 'c': 3, 'd': 2, 'p': 0, 'r': 2}
```

#### 字典的遍历

**直接`for in`遍历**

```python
for x in d:
    print(x)
```

直接用for in 遍历字典， 遍历的是字典的key

**`keys`函数遍历**

- d.keys() # keys 方法返回一个可迭代对象， 元素是字典所有的key
- d.keys() -> dict_keys(['d', 'a', 'c', 'r', 'p'])

```python
for x in d.keys():
	print(x)
```

**`values`函数遍历**

- d.values() # values 方法返回一个可迭代对象，元素是字典所有的value
- d.values() -> dict_values([2, 'default', 3, 2, 0])

```python
for x in d.values():
	print(x)
```

**`items`函数遍历**

- d.items() # items 方法返回一个可迭代对象， 元素是字典的所有(k, v)对
- d.items()  -> dict_items([('d', 2), ('a', 'default'), ('c', 3), ('r', 2), ('p', 0)])

```python
for x in d.items():
	print(x)
```

输出如下

```
('d', 2)
('a', 'default')
('c', 3)
('r', 2)
('p', 0)
```

另外一种方式：解析(k,v)对

```python
for k, v in d.items():
	print(k, v)
```

输出如下

```
d 2
a default
c 3
r 2
p 0
```

> **keys， values， items 返回的都类似生成器的对象, 它并不会复制一份内存**
>
> **Python2对应的函数返回的是列表， 会复制一份内存**

### 字典的限制

- 字典的key不能重复
- **字典的key需要可hash**

### 默认字典

默认字典是`defaultdict`

```python
In [25]: from collections import defaultdict

In [26]: d1 = {}

In [27]: d1
Out[27]: {}

In [28]: d2 = defaultdict(list)	# list在此是list的初始化函数

In [29]: d2
Out[29]: defaultdict(list, {})

In [30]: d1['a']
---------------------------------------------------------------------------
KeyError                                  Traceback (most recent call last)
<ipython-input-30-a9ea8faf9ae0> in <module>()
----> 1 d1['a']

KeyError: 'a'

In [31]: d2['a']
Out[31]: []
```

default初始化的时候， 需要传入一个工厂函数， 具体的介绍可以使用help(defaultdict)来查看，当我们使用下标访问一个key的时候， 如果这个key不存在， defaultdict会自动调用初始化时传入的函数， 生成一个对象作为这个key的value。因此上面的list函数初始化的时候就生成了一个空列表。

以下是使用dict和defaultdict的对比

```python
d = {}

for k in range(10):
    for v in range(10):
        if k not in d.keys():
            d[k] = []
        d[k].append(v)
```

如果这段代码使用defaultdict来写

```python
d = defaultdict(list)

for k in range(10):
    for v in range(10):
        d[k].append(v)
```

### 有序字典

有序字典是OrderedDict（第一个字母大写）

```python
In [33]: from collections import OrderedDict

In [34]: d = OrderedDict()

In [35]: d[0] = 3

In [36]: d[3] = 4

In [37]: d[1] = 5

In [38]: d
Out[38]: OrderedDict([(0, 3), (3, 4), (1, 5)])

In [39]: for k, v in d.items():
    ...:     print(k, v)
    ...:     
0 3
3 4
1 5

```

有序字典会保持插入的顺序

## 集合set

集合相关的所有内容如下

![集合小结](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/python/xmind/%E9%9B%86%E5%90%88%E5%92%8C%E9%9B%86%E5%90%88%E6%93%8D%E4%BD%9C.png)

xmind文件可点[这里](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/file/xmind/%E9%9B%86%E5%90%88%E5%92%8C%E9%9B%86%E5%90%88%E6%93%8D%E4%BD%9C.xmind)下载

```python
>>> help(set)
Help on class set in module __builtin__:

class set(object)
 |  set() -> new empty set object
 |  set(iterable) -> new set object
 |  
 |  Build an unordered collection of unique elements.
 |  
 |  Methods defined here:
```

详细使用可以参考：[https://blog.csdn.net/business122/article/details/7541486](https://blog.csdn.net/business122/article/details/7541486)

下面是一个小例子：

```
>>> a = [11,22,33,44,11,22]  
>>> b = set(a)  
>>> b  
set([33, 11, 44, 22])  
>>> c = [i for i in b]  
>>> c  
[33, 11, 44, 22]  
```

### 定义与初始化

```python
In [2]: s = set()
In [3]: s
Out[3]: set()
In [4]: s = {1, 2, 3}
In [5]: s
Out[5]: {1, 2, 3}
In [6]: s = set(range(3))
In [7]: s
Out[7]: {0, 1, 2}
```

### 增加

增加函数有两个`add`和`update`

`add`是增加单个元素，和列表的append操作类似，是原地修改

`update`是增加一个可迭代对象，和列表的extend操作类似，是原地修改

两个函数对于已经存在的元素会什么也不做

```python
In [7]: s
Out[7]: {0, 1, 2}

In [8]: s.add(3)

In [9]: s
Out[9]: {0, 1, 2, 3}

In [10]: s.add(3)

In [11]: s
Out[11]: {0, 1, 2, 3}

In [12]: help(s.update)
Help on built-in function update:

update(...) method of builtins.set instance
    Update a set with the union of itself and others.

In [13]: s.update(range(4, 7))

In [14]: s
Out[14]: {0, 1, 2, 3, 4, 5, 6}

In [15]: s.update(range(4, 9))

In [16]: s
Out[16]: {0, 1, 2, 3, 4, 5, 6, 7, 8}
```

### 删除

- remove 删除给定的元素， 元素不存在抛出KeyError（需要抛出异常时使用此函数）
- discard 删除给定的元素， 元素不存在，什么也不做（和remove的唯一区别）
- pop **随机arbitrary**删除一个元素并返回， 集合为空，抛出KeyError
- clear 清空集合

```python
In [16]: s
Out[16]: {0, 1, 2, 3, 4, 5, 6, 7, 8}

In [17]: s.remove(0)

In [18]: s
Out[18]: {1, 2, 3, 4, 5, 6, 7, 8}

In [19]: s.remove(10)
---------------------------------------------------------------------------
KeyError                                  Traceback (most recent call last)
<ipython-input-19-99f2b84d3df8> in <module>()
----> 1 s.remove(10)

KeyError: 10


In [21]: s
Out[21]: {1, 3, 4, 5, 6, 7, 8}

In [22]: s.discard(1)

In [23]: s
Out[23]: {3, 4, 5, 6, 7, 8}

In [24]: s.discard(10)

In [25]: s
Out[25]: {3, 4, 5, 6, 7, 8}

In [26]: help(s.pop)
Help on built-in function pop:

pop(...) method of builtins.set instance
    Remove and return an arbitrary（随机的） set element.
    Raises KeyError if the set is empty.

In [27]: s.pop()
Out[27]: 3

In [28]: s.pop()
Out[28]: 4

In [29]: s.clear()

In [30]: s
Out[30]: set()

In [31]: s.pop()
---------------------------------------------------------------------------
KeyError                                  Traceback (most recent call last)
<ipython-input-31-e76f41daca5e> in <module>()
----> 1 s.pop()

KeyError: 'pop from an empty set'
```

### 修改

集合不能修改单个元素

###查找

- 集合不能通过索引


- 集合没有访问单个元素的方法


- **集合不是线性结构， 集合元素没有顺序**

集合的pop操作的随机性可以证明集合不是线性结构的。

```python
In [32]: s = {1, 2, 3, 4, 5, 65, 66, 67, 88}

In [33]: s.pop()
Out[33]: 65
```

### 成员运算符

- in
- not in

用于判断一个元素是否在容器中

```python
In [34]: 1 in [1, 2, 3, 4]
Out[34]: True

In [35]: 5 in [1, 2, 3, 4]
Out[35]: False

In [36]: 5 not in [1, 2, 3, 4]
Out[36]: True

In [37]: 'love' in 'I love python'
Out[37]: True

In [38]: [1, 2] in [1, 2, 3, 4]
Out[38]: False

In [39]: 1 in (1, 2, ,3 ,4)
  File "<ipython-input-39-ed83805ebe55>", line 1
    1 in (1, 2, ,3 ,4)
                ^
SyntaxError: invalid syntax


In [40]: 1 in (1, 2, 3, 4)
Out[40]: True

In [41]: 1 in {1, 2, 3, 4}
Out[41]: True
```

**集合的成员运算和其他线性结构的时间复杂度不同**

```python
In [42]: lst = list(range(100000))

In [43]: s = set(range(100000))

In [44]: %%timeit
    ...: -1 in lst
    ...: 
1000 loops, best of 3: 1.61 ms per loop

In [45]: %%timeit
    ...: -1 in s
    ...: 
The slowest run took 29.72 times longer than the fastest. This could mean that an intermediate result is being cached.
10000000 loops, best of 3: 49.3 ns per loop
```

由以上可见，做成员运算的时候 集合的效率远高于列表。

```python
In [46]: lst2 = list(range(100))

In [47]: s = set(range(100))

In [48]: %%timeit
    ...: -1 in lst2
    ...: 
1000000 loops, best of 3: 1.6 µs per loop

In [49]: %%timeit
    ...: -1 in s
    ...: 
The slowest run took 20.77 times longer than the fastest. This could mean that an intermediate result is being cached.
10000000 loops, best of 3: 56.9 ns per loop
```

做成员运算时 列表的效率和列表的规模有关，而集合的效率和集合的规模无关。

成员运算：

- 集合 O(1)
- 列表(线性结构) O(n)

### 集合运算

集合运算主要有：交集，差集，对称差集，并集

python中的集合运算都对应两个版本，一个默认版本（返回新的集合），一个update版本（会更新集合本身）。

#### 交集

`intersection`

交集的特性：满足交换律，重载了`&`运算符

```python
In [50]: s1 = {1, 2, 3}

In [51]: s2 = {2, 3, 4}

In [52]: s1.intersection(s2)
Out[52]: {2, 3}

In [53]: s2.intersection(s1)		#交集满足交换律
Out[53]: {2, 3}

In [54]: s1
Out[54]: {1, 2, 3}

In [55]: s2
Out[55]: {2, 3, 4}

In [56]: s1.intersection_update(s2)		
    #交集的update版本，做原地修改，返回none,相当于  s1 = s1.intersection(s2)

In [57]: s1
Out[57]: {2, 3}

In [58]: s2
Out[58]: {2, 3, 4}

In [59]: s1 = {1, 2, 3}

In [60]: s1 & s2	#交集重载了&运算符，相当于 s1.intersection(s2)
Out[60]: {2, 3}
```

#### 差集

`difference`

差集特性：不满足交换律，重载了`-`运算符

```python
In [61]: s1 = {1, 2, 3}

In [62]: s2 = {2, 3, 4}

In [63]: s1.difference(s2)
Out[63]: {1}

In [64]: s2.difference(s1)	#差集不满足交换律
Out[64]: {4}

In [65]: s1.difference_update(s2)	#差集的update版本,相当于  s1 = s1.difference(s2)

In [66]: s1
Out[66]: {1}

In [67]: s1 = {1, 2, 3}

In [68]: s1 - s2	#差集重载了-运算符，相当于  s1.difference(s2)
Out[68]: {1}

In [69]: s2 - s1
Out[69]: {4}

```

####对称差集

`symmetric_difference`

对称差集特性：满足交换律，重载了`^`运算符

```python
In [70]: s1 = {1, 2, 3}

In [71]: s2 = {2, 3, 4}

In [72]: s1.symmetric_difference(s2)
Out[72]: {1, 4}

In [73]: s2.symmetric_difference(s1)		#对称差集满足交换律
Out[73]: {1, 4}

In [74]: s1.symmetric_difference_update(s2)		
    #对称差集的update版本，相当于  s1 = s1.symmetric_difference(s2)

In [75]: s1
Out[75]: {1, 4}

In [76]: s1 = {1, 2, 3}

In [77]: s1 ^ s2	#对称差集重载了^运算符，相当于 s1.symmetric_difference(s2)
Out[77]: {1, 4}
```

#### 并集

`union`,`update`

并集特性：满足交换律，重载了`|`运算符

```python
In [78]: s1 = {1, 2, 3}

In [79]: s2 = {2, 3, 4}

In [80]: s1.union(s2)
Out[80]: {1, 2, 3, 4}

In [81]: s2.union(s1)	#并集满足交换律
Out[81]: {1, 2, 3, 4}

In [82]: s1.update(s2)	#update函数就是并集的update版本，相当于 s1 = s1.update(s2)

In [83]: s1
Out[83]: {1, 2, 3, 4}

In [84]: s1 = {1, 2, 3}

In [85]: s1 + s2	#并集重载的运算符不是+
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-85-1659087814e1> in <module>()
----> 1 s1 + s2

TypeError: unsupported operand type(s) for +: 'set' and 'set'

In [86]: s1 | s2	#并集重载的运算符是|
Out[86]: {1, 2, 3, 4}
```

### 集合相关的判断

`issuperset`,`issubset`

`isdisjoint`：判断是否两个集合是否不相交（disjoint），有交集则返回False，没有交集则返回True

```python
In [87]: s1 = {1, 2, 3, 4}

In [88]: s2 = {1, 2}

In [89]: s1.issuperset(s2)		#判断是否是超集
Out[89]: True

In [90]: s2.issubset(s1)		#判断是否是子集
Out[90]: True

In [91]: s1.isdisjoint(s2)		#判断是否不相交
Out[91]: False

In [92]: s1 = {1, 2}

In [93]: s2 = {3, 4}

In [94]: s1.isdisjoint(s2)
Out[94]: True
```

### 集合的限制

集合的元素不能是可变的，集合的元素必须可hash

```python
In [95]: {'a', 'b', 'c'}
Out[95]: {'a', 'b', 'c'}

In [96]: {[1, 2, 3]}
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-96-e7ef34388120> in <module>()
----> 1 {[1, 2, 3]}

TypeError: unhashable type: 'list'

In [97]: {bytearray(b'abc')}
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-97-62b530a8195b> in <module>()
----> 1 {bytearray(b'abc')}

TypeError: unhashable type: 'bytearray'

In [98]: {{3, 4}}
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-98-47b1c90a198f> in <module>()
----> 1 {{3, 4}}

TypeError: unhashable type: 'set'

In [99]: {(1, 2)}
Out[99]: {(1, 2)}

In [100]: {b'abc'}
Out[100]: {b'abc'}

#hash函数可以直接使用
In [101]: hash(b'abc')
Out[101]: 1955665834644107130

In [102]: hash([1, 2, 3])
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-102-0b995650570c> in <module>()
----> 1 hash([1, 2, 3])

TypeError: unhashable type: 'list'
```

## 参考资料

[Python集合（set）类型的操作](https://blog.csdn.net/business122/article/details/7541486)

[python数据类型详解](https://www.cnblogs.com/linjiqin/p/3608541.html)

[Python中的List，Tuple和Dictionary](https://blog.csdn.net/jubincn/article/details/8570327)