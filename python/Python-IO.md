---
categories:
  - python
date: '2017-02-13T23:36:16'
description: ''
tags:
  - IO
  - 文件
  - 目录
  - 上下文
  - 序列化
title: Python IO
---




# Python IO

## 文件打开和关闭

文件打开和关闭就是两个函数，一个open函数一个close函数

**open函数的原型**

`open(file, mode='r', buffering=-1, encoding=None, errors=None, newline=None, closefd=True, opener=None)`

前面说open函数返回的是一个file-like对象，但是这个file-like对象并不是固定的，这个对象的类型会随着打开mode的变化而变化。

1. 以文本模式打开文件（'w', 'r'，'wt'，'rt'等），返回一个TextIOWrapper。 
2. 当用二进制模式打开文件时，返回的对象也会变化。
3. 在二进制读取模式，返回一个BufferedReader。
4. 在二进制写模式和二进制追加模式，返回一个BufferedWriter。
5. 在二进制读/写模式下，返回一个BufferedRandom。

```python
In [1]: f = open('./hello.py')	# 直接open函数打开，文件不存在会FileNotFoundError
---------------------------------------------------------------------------
FileNotFoundError                         Traceback (most recent call last)
<ipython-input-1-b6df97277b77> in <module>()
----> 1 f = open('./hello.py')

FileNotFoundError: [Errno 2] No such file or directory: './hello.py'

In [2]: f = open('./hello.py')	# 创建文件之后就可以打开，返回一个file-like对象

In [3]: f.read()	# 读出文件全部内容
Out[3]: "#!/usr/bin/env python\n# coding=utf-8\nprint('hello world')\n"

In [4]: f.close()	# 关闭文件
```

<!--more-->

## 文件读写

文件读写主要是read和write及其变种，文件的读写依赖于open函数的mode参数。

### open函数的mode参数

Mode具体含义如下

- 'r' open for reading (default)
- 'w' open for writing, **truncating the file first**
- 'x' create a new file and open it for writing
- 'a' open for writing, appending to the end of the file if it exists
- 'b' binary mode
- 't' text mode (default)
- '+' open a disk file for updating (reading and writing)
- 'U' universal newline mode (deprecated)

说明：

1. 当mode='x'时，如果文件不存在，则会抛出异常 FileExistsError。
2. 当mode='w'时，只要打开了文件，即使不写入内容，也会先清空文件。
3. 当mode包含+时， 会增加额外的读写操作， 也就说原来是只读的，会增加可写的操作， 原来是只写的，会增加可读的操作，但是+不改变其他行为。

### mode=t&mode=b

- mode=t 按字符操作
- mode=b 按字节操作

```python
In [1]: f = open('./hello.py', mode='rt')	# mode=t 读入的内容是字符串

In [2]: s = f.read()

In [3]: s
Out[3]: "#!/usr/bin/env python\n# coding=utf-8\nprint('hello world')\n"

In [4]: type(s)	# s是str类型的
Out[4]: str

In [5]: f.close()

In [6]: f = open('./hello.py', mode='rb')	# mode=b 读入的是bytes

In [7]: s = f.read()

In [8]: s
Out[8]: b"#!/usr/bin/env python\n# coding=utf-8\nprint('hello world')\n"

In [9]: type(s)
Out[9]: bytes
```

### 文件指针

当打开文件的时候， 解释器会持有一个指针， 指向文件的某个位置，当我们读写文件的时候，总是从指针处开始向后操作，并且移动指针。当mode=r时， 指针是指向0(文件开始)，当mode=a时， 指针指向EOF(文件末尾)

和文件指针相关的两个函数是`tell`函数和`seek`函数

**tell函数**

返回当前流的位置，对于文件来说，就是文件流的位置，即文件指针的位置。

**seek函数**

改变文件流的位置，并返回新的绝对位置。

```
seek(cookie, whence=0, /) method of _io.TextIOWrapper instance
```

**关于文件指针的总结**

当seek超出文件末尾， 不会有异常， tell也会超出文件末尾， 但是写数据的时候，还是会从文件末尾开始写

write 操作 从 min(EOF, tell())处开始

- 文件指针按字节操作（无论是字符模式还是字节模式）
- tell方法返回当前文件指针位置
- seek方法移动文件指针
- whence 参数 SEEK_SET(0) 从0开始向后移动offset个字节, SEEK_CUR(1) 从当前位置向后移动offset个字节, SEEK_END(2) 从EOF向后移动offset个字节
- offset是整数
- **当mode为t时， whence为SEEK_CUR或者SEEK_END时， offset只能为0**
- 文件指针不能为负数
- 读文件的时候从文件指针(pos)开始向后读
- 写文件的时候从min(EOF,pos)处开始向后写
- 以append模式打开的时候，无论文件指针在何处，都从EOF开始写

### 文件缓冲区

文件缓冲区由open函数的buffering参数决定，buffering表示缓冲方式，参数默认值为-1，表示文本模式和二进制模式都是采用默认的缓冲区。

**buffering=-1**

- 二进制模式： DEFAULT_BUFFER_SIZE
- 文本模式： DEFAULT_BUFFER_SIZE

**buffering=0**

- 二进制模式：unbuffered
- 文本模式：不允许

**buffering=1**

- 二进制模式： 1
- 文本模式： line buffering

**buffering>1**

- 二进制模式：buffering
- 文本模式： DEFAULT_BUFFER_SIZE

**总结**

- 二进制模式： 判断缓冲区剩余位置是否足够存放当前字节，如果不能，先flush， 在把当前字节写入缓冲区，如果当前字节大于缓冲区大小， 直接flush。
- 文本模式： line buffering，遇到换行就flush， 非line buffering，如果当前字节加缓冲区中的字节，超出缓冲区大小，**直接将缓冲区和当前字节全部flush**。


- flush和close可以强制刷新缓冲区。

## 上下文管理

上下文管理，会在离开时自动关闭文件， 但是不会开启新的作用域。

```python
In [1]: with open('./hello.py') as f:
    ...:     pass
    ...: 

In [2]: f.readable()	# 离开上下文管理后，文件已关闭，不可再进行I/O操作
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-18-97a5eee249a2> in <module>()
----> 1 f.readable()

ValueError: I/O operation on closed file	

In [3]: f
Out[3]: <_io.TextIOWrapper name='./hello.py' mode='r' encoding='UTF-8'>

In [4]: f.closed	# f已经关闭
Out[4]: True
```

上下文管理除了`with open('./hello.py') as f:`这种写法外，还有另外一种写法

```python
In [21]: f = open('./hello.py')

In [22]: with f:
    ...:     pass
    ...:
```

## File-like对象

像`open()`函数返回的这种有个`read()`方法的对象，在Python中统称为file-like Object。除了file外，还可以是内存的字节流，网络流，自定义流等等。常见的有StringIO和BytesIO。

### StringIO

StringIO顾名思义就是在内存中读写str。

要把str写入StringIO，我们需要先创建一个StringIO对象，然后项文件一样写入并读取。file支持的操作StringIO基本都是支持的。

```python
In [1]: from io import StringIO

In [2]: help(StringIO)


In [3]: sio = StringIO()	# 创建StringIO对象，也可以用str来初始化StringIO

In [4]: sio.write('hello world')
Out[4]: 11

In [5]: sio.write(' !')
Out[5]: 2

In [6]: sio.getvalue()	# getvalue()方法用于获得写入后的str。
Out[6]: 'hello world !'

In [7]: sio.closed
Out[7]: False

In [8]: sio.readline()
Out[8]: ''

In [9]: sio.seekable()
Out[9]: True

In [10]: sio.seek(0, 0)	# 支持seek操作
Out[10]: 0

In [11]: sio.readline()
Out[11]: 'hello world !'
```

要读取StringIO，可以用一个str初始化StringIO，然后，像读文件一样读取：

```python
In [1]: from io import StringIO

In [2]: sio = StringIO('I\nlove\npython!')

In [3]: for line in sio.readlines():
   ...:     print(line.strip())
   ...:     
I
love
python!
```

### BytesIO

StringIO操作的只能是str，如果要操作二进制数据，就需要使用BytesIO。

BytesIO实现了在内存中读写bytes，我们创建一个BytesIO，然后写入一些bytes：

```python
In [1]: from io import BytesIO

In [2]: bio = BytesIO()

In [3]: bio.write(b'abcd')
Out[3]: 4

In [4]: bio.seek(0)
Out[4]: 0

In [5]: bio.read()
Out[5]: b'abcd'

In [6]: bio.getvalue()	# getvalue 可以一次性独处全部内容，不管文件指针在哪里
Out[6]: b'abcd'
```

和StringIO类似，可以用一个bytes初始化BytesIO，然后，像读文件一样读取：

```python
In [1]: from io import BytesIO

In [2]: bio = BytesIO(b'abcd')

In [3]: bio.read()
Out[3]: b'abcd'
```

## 路径操作pathlib

路径操作有os.path和pathlib两种方式。

* os.path是已字符串的方式操作路径的：`import os`
* pathlib是面向对象设计的文件系统路径：`import pathlib`

pathlib在python3.2以上开始默认支持，在python2.7中如果要使用pathlib需要安装

```shell
pip install pathlib
```

pathlib模块的源代码见：Lib/pathlib.py

### 目录操作

pathlib目录的基本使用是pathlib模块中的Path这个类。

```python
In [1]: import pathlib	# 引入pathlib这个模块

In [2]: cwd = pathlib.Path('.')	# 使用pathlib模块的Path类初始化当前路径，参数是一个PurePath

In [3]: cwd	# 返回值是一个PosixPath，如果是windows环境会返回一个WindowsPath
Out[3]: PosixPath('.')
```

通过`help(pathlib.Path)`可以查看到Path类的各个Methods。

```python
Help on class Path in module pathlib:

class Path(PurePath)
 |  PurePath represents a filesystem path and offers operations which
 |  don't imply any actual filesystem I/O.  Depending on your system,
 |  instantiating a PurePath will return either a PurePosixPath or a
 |  PureWindowsPath object.  You can also instantiate either of these classes
 |  directly, regardless of your system.
 |  
 |  Method resolution order:
 |      Path
 |      PurePath
 |      builtins.object
 |  
 |  Methods defined here:
 |  
 |  __enter__(self)
 |  
 |  __exit__(self, t, v, tb)
 |  
...
```

目录操作的几个函数：

* `is_dir(self)`：判断路径是否是目录
* `iterdir(self)`：生成当前路径下所有文件(包括文件夹)的生成器，但是不会yield '.' 和'..'这两个路径
* `mkdir(self, mode=511, parents=False, exist_ok=False)`：删除当前目录，可以指定mode
* `rmdir(self)`：删除目录，并且目录必须为空，否则会报错

使用示例如下

```python
In [4]: cwd.is_dir()
Out[4]: True

In [5]: cwd.iterdir()	# iterdir函数返回的是一个生成器
Out[5]: <generator object Path.iterdir at 0x7f6727d926d0>

In [6]: for f in cwd.iterdir():	# 不会生成'.' 和'..'
   ...:     print(type(f))
   ...:     print(f)
   ...:     
<class 'pathlib.PosixPath'>
hello.py
<class 'pathlib.PosixPath'>
aa.py

In [7]: cwd.mkdir('abc')	# pathlib的mkdir是路径对象的方法
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-7-3b48dd61eb0f> in <module>()
----> 1 cwd.mkdir('abc')

/home/clg/.pyenv/versions/3.5.2/lib/python3.5/pathlib.py in mkdir(self, mode, parents, exist_ok)
   1212         if not parents:
   1213             try:
-> 1214                 self._accessor.mkdir(self, mode)
   1215             except FileExistsError:
   1216                 if not exist_ok or not self.is_dir():

/home/clg/.pyenv/versions/3.5.2/lib/python3.5/pathlib.py in wrapped(pathobj, *args)
    369         @functools.wraps(strfunc)
    370         def wrapped(pathobj, *args):
--> 371             return strfunc(str(pathobj), *args)
    372         return staticmethod(wrapped)
    373 

TypeError: an integer is required (got type str)

In [8]: d = pathlib.Path('./abc')

In [9]: d.exists()
Out[9]: False

In [10]: d.mkdir(755)	 # 创建文件夹，但是755不等于0o755(8进制)

In [11]: %ls
aa.py  abc/  hello.py

In [12]: %ls -ld ./abc
d-wxrw---t. 2 clg clg 6 Feb 13 21:01 ./abc/	# mode指定有问题，所以权限不正常

In [13]: d.rmdir()

In [14]: d.exists()
Out[14]: False

In [15]: d.mkdir(0o755)	# 使用8进制指定mode

In [16]: %ls -ld ./abc
drwxr-xr-x. 2 clg clg 6 Feb 13 21:03 ./abc/
```

### 通用操作

主要是一些路径的通用操作

```python
In [17]: f = pathlib.Path('./ab/cd/a.txt')

In [18]: f.exists()
Out[18]: False

In [19]: f.is_file()
Out[19]: False

In [20]: f.is_absolute()
Out[20]: False

In [21]: f = pathlib.Path('./hello.py')

In [22]: f.is_file()
Out[22]: True

In [23]: f.is_absolute()
Out[23]: False

In [24]: f.absolute()	# 获取路径的绝对路径
Out[24]: PosixPath('/home/clg/workspace/subworkspace/hello.py')

In [25]: f.chmod(0o755)	# 改变路径的权限

In [26]: %ls -ld ./hello.py
-rwxr-xr-x. 1 clg clg 58 Feb  8 13:32 ./hello.py*

In [27]: f.cwd()	# 返回一个新路径指向当前工作目录
Out[27]: PosixPath('/home/clg/workspace/subworkspace')

In [28]: f.home()
Out[28]: PosixPath('/home/clg')

In [29]: pathlib.Path('~').expanduser()	# 将~转换成功绝对路径
Out[29]: PosixPath('/home/clg')

In [30]: f.name()	# name是一个属性，不是一个方法
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-30-f0ea48ccc8ff> in <module>()
----> 1 f.name()

TypeError: 'str' object is not callable

In [31]: f.name	# 获取得到的是基本名称basename
Out[31]: 'hello.py'

In [32]: f.home().name
Out[32]: 'clg'

In [33]: f.owner()	# 获取属主
Out[33]: 'clg'

In [34]: f.home().parent
Out[34]: PosixPath('/home')

In [35]: f.parts
Out[35]: ('hello.py',)

In [36]: f.absolute().parts	# 获取路径的拆分
Out[36]: ('/', 'home', 'clg', 'workspace', 'subworkspace', 'hello.py')

In [37]: f.root	# 获取根目录，但是'./hello.py'获取到的则是'.'
Out[37]: ''

In [38]: f.home().root	# 获取根目录
Out[38]: '/'

In [39]: f.suffix	# 获取后缀
Out[39]: '.py'

In [40]: f.stat()	# 类似os.stat()，返回路径的各项信息
Out[40]: os.stat_result(st_mode=33261, st_ino=34951327, st_dev=64768, st_nlink=1, st_uid=1000, st_gid=1000, st_size=58, st_atime=1486531928, st_mtime=1486531926, st_ctime=1486995977)

In [41]: f.stat().st_mode	# 获取stat()返回结果中的各个信息的方法：使用'.'
Out[41]: 33261

In [42]: d = pathlib.Path('..')

In [43]: for x in d.glob(*.py):	# rglob(self, pattern)参数是一个pattern
  File "<ipython-input-43-3fdfb8e408ac>", line 1
    for x in d.glob(*.py):
                     ^
SyntaxError: invalid syntax


In [44]: for x in d.glob('*.py'):	# 返回当前路径下的通配文件
    ...:     print(x)
    ...:     
../judge.py
../progress.py
../zipperMethod.py
../decorator.py

In [45]: for x in d.rglob('*.py'):	# 返回当前路径下及其子路径下的通配文件（递归）
    ...:     print(x)
    ...:     
../judge.py
../progress.py
../zipperMethod.py
../decorator.py
../subworkspace/hello.py
../subworkspace/aa.py
```

### 文件复制移动删除

使用`shutil`模块即可

```python
import shutil
```

- shutil.copyfileobj 	# 操作对象是文件对象
- shutil.copyfile # 仅复制内容
- shutil.copymode # 仅复制权限
- shutil.copystat # 仅复制元数据
- shutil.copy # 复制文件内容和权限 copyfile + copymode
- shutil.copy2 # 复制文件内容和元数据 copyfile + copystat
- shutil.copytree # 递归复制目录
- shutil.rmtree # 用于递归删除目录
- shutil.move # 具体实现依赖操作系统， 如果操作系统实现了 rename系统调用， 直接走rename系统调用，如果没实现，先使用copytree复制， 然后使用rmtree删除源文件

## 序列化和反序列化

- 序列化： 对象转化为数据
- 反序列化： 数据转化为对象

### Python私有协议pickle

pickle 是Python私有的序列化协议

pickle源代码见：lib/python3.5/pickle.py

主要函数

* `dumps` 对象导出为数据，即序列化
* `loads` 数据载入为对象，即反序列化，反序列化一个对象时，必须存在此对象的类

```python
In [1]: import pickle

In [2]: class A:	# 声明一个类A
   ...:     def print(self):
   ...:         print('aaaa')
   ...:         

In [3]: a = A()	# 定义类A的一个对象a

In [4]: pickle.dumps(a)	# 对象导出为数据
Out[4]: b'\x80\x03c__main__\nA\nq\x00)\x81q\x01.'

In [5]: b = pickle.dumps(a)

In [6]: pickle.loads(b)	# 数据导出为对象
Out[6]: <__main__.A at 0x7f5dcdc71dd8>

In [7]: a
Out[7]: <__main__.A at 0x7f5dcdd28be0>	# 两个对象的地址不一样，但是两个对象的内容确实一样的

In [8]: aa = pickle.loads(b)

In [9]: a.print()	# 原始对象的print函数
aaaa

In [10]: aa.print()	# 反序列化对象的print函数
aaaa
```

### 通用的json协议

JSON格式支持的数据类型如下

| 类型         | 描述                                       |
| ---------- | ---------------------------------------- |
| Number     | 在JavaScript中的双精度浮点格式                     |
| String     | 双引号的反斜杠转义的Unicode，**对应python中的str**      |
| Boolean    | true 或 false                             |
| Array      | 值的有序序列，**对应python中的list**                |
| Value      | 它可以是一个字符串，一个数字，真的还是假（true/false），空(null )等 |
| Object     | 无序集合键值对，**对应python中的dict**               |
| Whitespace | 可以使用任何一对中的令牌                             |
| null       | empty                                    |

使用示例如下

```python
In [1]: import json

In [2]: d = {'a': 1, 'b': [1, 2, 3]}

In [3]: json.dumps(d)
Out[3]: '{"a": 1, "b": [1, 2, 3]}'

In [4]: json.loads('{"a": 1, "b": [1, 2, 3]}')
Out[4]: {'a': 1, 'b': [1, 2, 3]}
```

json参考：[JSON 数据格式](https://www.cnblogs.com/SkySoot/archive/2012/04/17/2453010.html)