---
categories:
  - python
date: '2017-01-12T17:14:43'
description: ''
tags:
  - Python
  - 拉链法
  - 字典
title: Python拉链法和开地址法实现字典
---




# Python拉链法和开地址法实现字典

Python字典(dictionary)是除列表之外python中最灵活的内置数据结构类型。列表是有序的对象结合，**字典是无序的对象集合**。两者之间的区别在于：字典当中的元素是通过键来存取的，而不是通过偏移存取。

在列表中使用下标索引可以快速的得到对应的值，那么我们需要做的有两件事情：

- **怎样把键计算出一个唯一值**
- **怎样把这个唯一值均匀并且唯一的分布在长度固定的列表中**

**怎样把键计算出一个唯一值**

>  因为字典的键是不可变的，可hash的，因此我们可以用hash函数计算key对应的唯一hash值。

**怎样把这个唯一值均匀并且唯一的分布在长度固定的列表中**

> hash散列是可以把大数据集映射到定长数据集的算法，因此我们可以对上述计算出来的hash值进行散列。很明显散列之后会出现散列冲突。因此我们需要处理这种冲突一遍唯一值能够均匀唯一的分布。这个时候就有两种处理散列冲突的方法：拉链法和开地址法

<!--more-->

## 拉链法

把具有相同散列地址的`k,v`对放在同一个单链表中。下面实现两个函数

- `put`函数：`put(slots, key, value)`，用来向字典中插入数据
- `get`函数：`get(slots, key)`，用来从字典中读取数据。

还可以实现更多的函数，比如`dict.keys()`

```python
#!/usr/bin/env python
# coding=utf-8

slots = []
slotsNum = 32
for _ in range(32):
    slots.append([])

def put(slots, key, value):
    i = hash(key) % slotsNum
    pos = -1
    for pos, (k, v) in enumerate(slots[i]):
        if key == k:
            break
    else:
        slots[i].append((key, value))
    if pos >= 0 and pos < len(slots[i]):
        slots[i][pos] = (key, value)

def get(slots, key):
    i = hash(key) % slotsNum
    for k, v in slots[i]:
        if key == k:
            return v
    else:
        raise KeyError(key)		# 不存在时抛出异常

put(slots, 'a', 1)
print(get(slots, 'a'))
put(slots, 'b' ,2)
print(get(slots, 'b'))
put(slots, 'a', 3)
print(get(slots, 'a'))
```

下面将这两个函数封装成类

```python
class Dict:
    def __init__(self, num):
        self.__solts__ = []
        self.num = num
        for _ in range(num):
            self.__solts__.append([])

    def put(self, key, value):
        i = hash(key) % self.num
        for p, (k, v) in enumerate(self.__solts__[i]):
            if k == key:
                break
        else:
            self.__solts__[i].append((key, value))
            return
        self.__solts__[i][p] = (key, value)

    def get(self, key):
        i = hash(key) % self.num
        for k, v in self.__solts__[i]:
            if k == key:
                return v
        raise KeyError(key)

    # keys函数
    def keys(self):
        ret = []
        for solt in self.__solts__:
            for k, _ in solt:
                ret.append(k)
        return ret
```

封装成类之后，使用方法和Python提供的`dict`就比较像了

## 开地址法

Python字典内部实现时处理散列冲突的方法就是开地址法，开地址法在后续补充

- [《Python源码剖析》的笔记-第五章 Python中的dict对象](https://book.douban.com/annotation/23775810/)


- [【译】Python字典实现](https://harveyqing.gitbooks.io/python-read-and-write/content/python_advance/python_dict_implementation.html)