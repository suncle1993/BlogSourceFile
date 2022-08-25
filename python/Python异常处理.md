---
categories:
  - python
date: '2017-03-05T20:28:17'
description: ''
tags:
  - Python
  - 异常处理
title: Python异常处理
---



## 区分Exception和Syntax Error

在写Python程序的时候经常会报错，报错通常有以下两种情况：

1. 语法错误（Syntax Error）: **部分语法错误属于异常**
2. 异常（Exception）

### 语法错误

语法错误也称为解析错误，是最常遇到的一种错误

```python
In [1]: while True print('Hello!')
  File "<ipython-input-1-5c66e4fd0ae9>", line 1
    while True print('Hello!')
                   ^
SyntaxError: invalid syntax

```

当代码不符合Python语法的时候就会抛出SyntaxError。

### 异常

Python用异常对象来表示异常情况。遇到错误后，会引发异常。如果异常没有处理或捕捉，程序就会用**traceback**终止程序的执行，如果是在多线程程序中，则会终止当前线程的执行。

```python
In [2]: 1/0
---------------------------------------------------------------------------
ZeroDivisionError                         Traceback (most recent call last)
<ipython-input-2-05c9758a9c21> in <module>()
----> 1 1/0

ZeroDivisionError: division by zero

```

除以0时，就会抛出ZeroDivisionError异常（ZeroDivisionError类的一个实例）。

<!--more-->

## 异常层次结构

Python 3.5.2中内置异常的类层次结构如下：参考标准库

```
BaseException  # 所有异常的基类
 +-- SystemExit  # 程序退出/终止
 +-- KeyboardInterrupt  # 由键盘中断（通常为Ctrl+C) 生成
 +-- GeneratorExit  # 由生成器.close()方法引发
 +-- Exception  # 所有非退出异常的基类
      +-- StopIteration  # 停止迭代错误
      +-- StopAsyncIteration  # 停止异步迭代错误
      +-- ArithmeticError  # 算数异常的基类
      |    +-- FloatingPointError  # 浮点操作异常
      |    +-- OverflowError  # 溢出导致的异常
      |    +-- ZeroDivisionError  # 对0进行除或取模操作导致的异常
      +-- AssertionError  # 由assert语句引发
      +-- AttributeError  # 当属性名称无效时引发
      +-- BufferError  # 缓冲区错误引发
      +-- EOFError  # 到达文件结尾时引发
      +-- ImportError  # import语句失败
      +-- LookupError  # 索引和键错误
      |    +-- IndexError  # 超出序列索引的范围
      |    +-- KeyError  # 键不存在
      +-- MemoryError  # 内存不足
      +-- NameError  # 无法找到局部或全局名称
      |    +-- UnboundLocalError  # 未绑定的局部变量
      +-- OSError  # 操作系统错误
      |    +-- BlockingIOError  # IO阻塞
      |    +-- ChildProcessError  # 子进程
      |    +-- ConnectionError  # 连接错误
      |    |    +-- BrokenPipeError  # 管道断开
      |    |    +-- ConnectionAbortedError  # 连接中止
      |    |    +-- ConnectionRefusedError  # 连接拒绝
      |    |    +-- ConnectionResetError  # 连接重置
      |    +-- FileExistsError # 文件已存在
      |    +-- FileNotFoundError  # 文件不存在
      |    +-- InterruptedError  # 中断错误
      |    +-- IsADirectoryError  # 目录错误
      |    +-- NotADirectoryError  # 非目录错误
      |    +-- PermissionError  # 权限错误
      |    +-- ProcessLookupError  # 进程查找错误
      |    +-- TimeoutError  # 超时错误
      +-- ReferenceError  # 销毁被引用对象后仍然使用引用
      +-- RuntimeError  # 运行时错误
      |    +-- NotImplementedError  # 没有实现的特性
      |    +-- RecursionError  # 递归错误
      +-- SyntaxError  # 语法错误
      |    +-- IndentationError  # 缩进错误
      |         +-- TabError  # 使用不一致的制表符
      +-- SystemError  # 解释器中的非致命系统错误
      +-- TypeError  # 给操作传递了错误的类型
      +-- ValueError  # 无效类型
      |    +-- UnicodeError  # Unicode错误
      |         +-- UnicodeDecodeError  # Unicode解码错误
      |         +-- UnicodeEncodeError  # Unicode编码错误
      |         +-- UnicodeTranslateError  # Unicode转换错误
      +-- Warning  # 警告的基类
           +-- DeprecationWarning  # 关于被弃用的特征的警告
           +-- PendingDeprecationWarning  # 关于特性将会被废弃的警告
           +-- RuntimeWarning  # 可疑的运行时行为的警告
           +-- SyntaxWarning  # 可疑的语法的警告
           +-- UserWarning  # 用户代码生成的警告
           +-- FutureWarning  # 关于构造将来语义会有改变的警告
           +-- ImportWarning  # import语句的警告
           +-- UnicodeWarning  # Unicode警告
           +-- BytesWarning  # Bytes警告
           +-- ResourceWarning  # 资源警告
```

- 所有异常的基类都是BaseException
- 除SystemExit，KeyboardInterrupt，GeneratorExit三种异常外都继承自Exception

## 捕获异常

捕获异常可以使用try/except语句。try/except语句用来检测try语句块中的错误，从而让except语句捕获异常信息并处理。

### try/except

**基础语法**

```
try:
    <语句>
except <name>：
    <语句>          #如果在try部分引发了名为'name'的异常，则执行这段代码
```

**示例**

```python
In [3]: try:
   ...:     x = int(input("Please enter a number: "))
   ...: except ValueError:
   ...:     print("No valid number.")
   ...:     
Please enter a number: asd
No valid number.
```

### 多个except

```python
In [4]: import sys

In [5]: try:
   ...:     f = open('file.txt')  # 文件不存在的时候就会抛出FileNotFoundError异常
   ...:     s = f.readline()
   ...:     i = int(s.strip())
   ...: except OSError:  # FileNotFoundError异常的上层异常
   ...:     print('OS error.')
   ...: except ValueError:
   ...:     print('Could not convert data to integer.')
   ...: except Exception:
   ...:     print('Exception.')
   ...: except:  # 不加具体异常类型时，会捕获所有的异常，应该不用或者慎用
   ...:     print('Unexpected error:', sys.exc_info()[0])
   ...:     
OS error.
```

各个except之间的执行顺序：

- except顺序捕获try中抛出的异常
- 越具体的异常应该越靠前，越一般的异常应该越靠后

### 可选的else语句

**语法**

```
try:
    <语句>
except <name>：
    <语句>          #如果在try部分引发了名为'name'的异常，则执行这段代码
else:
    <语句>          #如果没有异常发生，则执行这段代码
```

如果try部分没有抛出异常，但是又必须执行的语句，则放在else语句中。

**代码**

```python
for arg in sys.argv[1:]:
    try:
        f = open(arg, 'r')
    except IOError:
        print('cannot open', arg)
    else:  # 没有抛出异常（即文件正确打开）时打印出文件中的每一行
        print(arg, 'has', len(f.readlines()), 'lines')
        f.close()

```

### finally语句

finally语句用来定义在任何情况下都必须执行的语句。

```python
In [1]: try:
   ...:     raise KeyboardInterrupt
   ...: finally:
   ...:     print('Goodbye')
   ...:     
Goodbye
---------------------------------------------------------------------------
KeyboardInterrupt                         Traceback (most recent call last)
<ipython-input-8-132d568ca0fb> in <module>()
      1 try:
----> 2     raise KeyboardInterrupt
      3 finally:
      4     print('Goodbye')
      5 

KeyboardInterrupt:
```

**带return语句的finally执行顺序**

```python
def p(x):
    print(x)
    return x

def t():
    try:
        return p(2)
        print('haha')
    finally:
        return p(3)

x = t()

# 输出结果为：
2
3
# 返回值x为3
```

可见，在try块中，只要有finally语句，即使函数提前返回，也会在退出try块之前执行finally语句，因此返回值会被finally中的return语句替代。

**综合使用示例**

```python
In [1]: def divide(x, y):
   ...:     try:
   ...:         result = x / y
   ...:     except ZeroDivisionError:
   ...:         print('division by zero!')
   ...:     else:
   ...:         print('result is ', result)
   ...:     finally:
   ...:         print('executing finally clause.')
   ...:         

In [2]: divide(2, 0)
division by zero!
executing finally clause.

In [3]: divide(2, 1)
result is  2.0
executing finally clause.

In [4]: divide('2', '1')
executing finally clause.
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-4-34bb38fa74fd> in <module>()
----> 1 divide('2', '1')

<ipython-input-1-4273ffa41b76> in divide(x, y)
      1 def divide(x, y):
      2     try:
----> 3         result = x / y
      4     except ZeroDivisionError:
      5         print('division by zero!')

TypeError: unsupported operand type(s) for /: 'str' and 'str'
```

结论：

- 任何情况下finally语句都会执行。
- 即使try部分中有return语句，也会在退出try块之前执行finally语句，并且返回值是finally中的return
- 如果有异常没有被处理，则在执行完成finally语句之后会会抛出没有被处理的异常
- 在实际使用中，finally通常用来释放额外的资源，比如文件或者网络连接

## 主动抛出异常

**raise语句**

```python
In [1]: raise NameError('Hello')
---------------------------------------------------------------------------
NameError                                 Traceback (most recent call last)
<ipython-input-1-64f372e60821> in <module>()
----> 1 raise NameError('Hello')

NameError: Hello
```

## 用户自定义异常

用户自定义异常类时，应该直接或者间接的继承自Exception类。

```python
class CustomException(Exception):
    def __init__(self, code, message):
        self.code = code
        self.message = message

try:
    raise CustomException(500, 'error')
except CustomException as e:
    print('{},{}'.format(e.code, e.message))

# 输出结果：500,error
```

## 异常的传递

在函数内引发异常时，如果异常没有被捕获到，那么它就会被传播到函数被调用的地方。

```python
In [1]: def a():
   ...:         raise Exception('Hello')
   ...: 

In [2]: def b():
   ...:         print('enter b')
   ...:         a()  # 函数a中引发的异常，会传递到父函数的调用出
   ...:         print('exit b')  # a中抛出异常之后传递到b，中止b的执行
   ...:     

In [3]: b()
enter b
---------------------------------------------------------------------------
Exception                                 Traceback (most recent call last)
<ipython-input-3-9c619ddbd09b> in <module>()
----> 1 b()

<ipython-input-2-f99a565bd6f8> in b()
      1 def b():
      2         print('enter b')
----> 3         a()
      4         print('exit b')
      5 

<ipython-input-1-6e68e60e93b5> in a()
      1 def a():
----> 2         raise Exception('Hello')

Exception: Hello

```





