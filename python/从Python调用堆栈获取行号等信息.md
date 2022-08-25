---
categories:
  - python
date: '2017-10-31T09:43:51'
description: ''
tags:
  - Python
  - 堆栈
  - inspect
title: 从Python调用堆栈获取行号等信息
---




程序中的日志打印，或者消息上传，比如kafka消息等等。经常上传的消息中需要上传堆栈信息中的文件名、行号、上层调用者等具体用于定位的消息。Python提供了以下两种方法：

- `sys._getframe`， 基础方法
- `inspect.currentframe`， 推荐方法，提供除了`sys._getframe`方法之外更多的frame相关的方法

<!--more-->

具体使用如下

# 使用`sys._getframe`私有方法

具体使用方法如下：

```python
import os
import sys


def get_cur_info():
	"""
        获取调用时的文件名，行号，上层调用者的名称
        :return: 文件名，行号，上层调用者名称
    """
	try:
		current_frame = sys._getframe(2)
		return os.path.basename(current_frame.f_code.co_filename), current_frame.f_lineno, current_frame.f_code.co_name
	except ValueError:
		return 'unknown', 0, 'unknown'

```

具体的函数输出结果演示可以参见下面的inspect模块结果

# 使用`inspect`模块（推荐）

相比于sys的内置私有方法，更推荐inspect模块。inspect模块的具体使用方法如下

```python
import os
import inspect

def get_cur_info():
	try:
		current_frame = inspect.currentframe(2)
		return os.path.basename(current_frame.f_code.co_filename), current_frame.f_lineno, current_frame.f_code.co_name
	except ValueError:
		return 'unknown', 0, 'unknown'


def produce():
	return get_cur_info()


def business():
	return produce()


if __name__ == '__main__':
	print(get_cur_info())  # 输出 ('unknown', 0, 'unknown')

	print(produce())  # 输出 ('a.py', 22, '<module>')

	print(business())  # 输出 ('a.py', 16, 'business')

```

主要依赖inspect.currentframe方法，关于inspect.currentframe方法的使用见帮助文档

```python
>>> help(inspect.currentframe)
Help on built-in function _getframe in module sys:

_getframe(...)
    _getframe([depth]) -> frameobject
    
    Return a frame object from the call stack.  If optional integer depth is
    given, return the frame object that many calls below the top of the stack.
    If that is deeper than the call stack, ValueError is raised.  The default
    for depth is zero, returning the frame at the top of the call stack.
    
    This function should be used for internal and specialized
    purposes only.
```

从调用堆栈返回一个帧对象。深度为整数，默认为0，返回调用堆栈顶部的帧。如果指定深度比调用堆栈深，会抛出ValueError异常。该功能应该只用于内部和专业目的。

inspect.currentframe方法的实现见内置库inspect.py

```python
if hasattr(sys, '_getframe'):
    currentframe = sys._getframe
else:
    currentframe = lambda _=None: None
```

所以本质上inspect.currentframe方法等同于sys._getframe方法

> currentframe = lambda _=None: None 等同于 currentframe = lambda _: None ，即lambda函数接收一个参数，返回None

---

参考：

1. [Python frame hack](http://farmdev.com/src/secrets/framehack/index.html)

2. [StackOverFlow-In Python, how do I obtain the current frame?](https://stackoverflow.com/questions/1140194/in-python-how-do-i-obtain-the-current-frame)