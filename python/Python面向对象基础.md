---
abbrlink: 3226889536
alias: 2017/03/08/Python面向对象基础/index.html
categories:
- python
date: '2017-03-08T12:22:42'
description: ''
tags:
- Python
- 面向对象
title: Python面向对象基础
---









# 类的定义

类的定义的语法

```python
class ClassName:
    <statement-1>
    .
    .
    .
    <statement-N>
```

示例代码

```python
class Door:
    def __init__(self, number, status):
        self.number = number
        self.status = status
```

<!--more-->

# 类的实例化

```python
class Door:
    def __init__(self, number, status):
        self.number = number
        self.status = status

door = Door(1001, 'open')
door.number
door.status
```

- 创建对象使用`类名(__init__ 函数除第一个参数外的参数列表)`
- 创建对象的时候实际执行了 `__init__`函数
- `__init__` 函数并不会创建对象

函数创建及初始化的过程

1. 首先创建对象
2. 对象作为self参数传递给`__init__`函数
3. 返回self

# 作用域

## 类变量

示例代码

```python
In [1]: class A:
   ...:         NAME = 'A'  # 类的直接下级作用域 叫做类变量
   ...:         def __init__(self, name):
   ...:             self.name = name  # 关联到实例的变量 叫做实例变量
   ...:          

In [2]: a = A('a')

In [3]: a.NAME
Out[3]: 'A'

In [4]: a.name
Out[4]: 'a'

In [5]: A.NAME
Out[5]: 'A'

In [6]: A.name
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-6-61c1cc534250> in <module>()
----> 1 A.name

AttributeError: type object 'A' has no attribute 'name'

In [7]: a2 = A('a2')

In [8]: a2.NAME
Out[8]: 'A'

In [9]: a2.NAME = 'A2'  # 给示例a2的类变量NAME赋值

In [10]: a2.NAME
Out[10]: 'A2'

In [11]: a.NAME
Out[11]: 'A'

In [12]: A.NAME  # 类变量没有变化
Out[12]: 'A'

In [13]: a2.xxx = 3

In [14]: a2.xxx  # 赋值之后a2多了xxx属性
Out[14]: 3

In [15]: A.NAME = 'AA'  # 直接修改类的类变量

In [16]: A.NAME
Out[16]: 'AA'

In [17]: a.NAME  # 对应的实例的类变量也发生了改变
Out[17]: 'AA'

In [18]: a2.NAME  # a2的类变量在之前的赋值被覆盖了，因此改变类变量的并不会影响a2
Out[18]: 'A2'
```

所以

- 类变量对类和实例都可见
- 所有实例共享类变量
- 当给实例的类变量赋值时，相当于动态的给这个实例增加了一个属性，覆盖了类变量

## 属性查找顺序

- `__dict__`： 实例变量的字典
- `__class__`： 得到实例对应的类
- 先查找`__dict__`在查找`__class__`

代码

```python
In [1]: class A:
   ...:     NAME = 'A'
   ...:     def __init__(self, name):
   ...:         self.name = name
   ...:         

In [2]: a = A('a')

In [3]: a.NAME
Out[3]: 'A'

In [4]: a.__class__.NAME
Out[4]: 'A'

In [5]: a.__dict__
Out[5]: {'name': 'a'}

In [6]: a.__class__  # a.__class__表示实例对应的类
Out[6]: __main__.A

In [7]: a.NAME = 'AA'

In [8]: a.__dict__  # 覆盖类变量之后__dict__增加了一个键值对
Out[8]: {'NAME': 'AA', 'name': 'a'}

In [9]: a.__dict__['NAME'] = 'AAA'  # 可以直接修改__dict__

In [10]: a.__dict__
Out[10]: {'NAME': 'AAA', 'name': 'a'}

In [11]: a.__class__.NAME
Out[11]: 'A'

In [12]: a.__class__.__dict__
Out[12]: 
mappingproxy({'NAME': 'A',
              '__dict__': <attribute '__dict__' of 'A' objects>,
              '__doc__': None,
              '__init__': <function __main__.A.__init__>,
              '__module__': '__main__',
              '__weakref__': <attribute '__weakref__' of 'A' objects>})

```

## 类装饰器

参数是一个类，并且返回一个类的函数就可以是一个类装饰器。

类装饰器通常用于给类增加属性，如果增加方法，则都是类级的方法。

**代码1：给类增加属性**

函数方法增加：定义set_name函数给类F增加一个NAME属性

```python
In [1]: class F:
   ...:     pass
   ...: 

In [2]: def set_name(cls, name):  # 给cls增加属性NAME=name
   ...:     cls.NAME = name
   ...:     return cls
   ...: 

In [3]: F1 = set_name(F, 'F')  # 返回F本身，并且F1指向F

In [4]: F1.NAME
Out[4]: 'F'

In [5]: f1 = F1()

In [6]: f1.NAME
Out[6]: 'F'

In [7]: F1.__dict__
Out[7]: 
mappingproxy({'NAME': 'F',
              '__dict__': <attribute '__dict__' of 'F' objects>,
              '__doc__': None,
              '__module__': '__main__',
              '__weakref__': <attribute '__weakref__' of 'F' objects>})

In [8]: f1.__dict__
Out[8]: {}

In [9]: f1.__class__
Out[9]: __main__.F

In [10]: F.__dict__  # 本质上增加的还是类F
Out[10]: 
mappingproxy({'NAME': 'F',
              '__dict__': <attribute '__dict__' of 'F' objects>,
              '__doc__': None,
              '__module__': '__main__',
              '__weakref__': <attribute '__weakref__' of 'F' objects>})
```

对set_name函数进行柯里化，实现带参数的类装饰器

```python
In [2]: def set_name(name):   # 传入参数name
   ...:     def wrap(cls):   # 装饰器是wrap
   ...:         cls.NAME = name
   ...:         return cls
   ...:     return wrap
   ...: 

In [3]: @set_name('G')
   ...: class G:
   ...:     pass
   ...: 

In [4]: G.NAME
Out[4]: 'G'

In [5]: class G:
   ...:     pass
   ...: 

In [6]: G = set_name('G')(G)  # 装饰器的函数调用方法

In [7]: G.NAME
Out[7]: 'G'
```

**代码2：给类增加方法**

类装饰器`get_name`给类H增加一个方法`__get_name__`

```python
In [1]: def get_name(cls):
   ...:     def _get_name(self):
   ...:         return cls.__name__
   ...:     cls.__get_name__ = _get_name  # 给cls增加__get_name__指向_get_name
   ...:     return cls
   ...: 

In [2]: @get_name
   ...: class H:
   ...:     pass
   ...: 

In [3]: h = H()

In [4]: h.__get_name__()
Out[4]: 'H'

In [5]: H.__dict__
Out[5]: 
mappingproxy({'__dict__': <attribute '__dict__' of 'H' objects>,
              '__doc__': None,
              '__get_name__': <function __main__.get_name.<locals>._get_name>,
              '__module__': '__main__',
              '__weakref__': <attribute '__weakref__' of 'H' objects>})
```

## 类方法/静态方法

方法的定义都是类级的，但是有的方法使用实例调用，有的方法使用类来调用

- 类方法：当一个方法，被classmethod装饰时， 第一个参数会变成类本身， 这样的方法叫类方法
- 当一个方法， 被staticmethod装饰的时候，不会自动传递第一个参数， 这样的方法叫静态方法

**代码**

```python
class I:
    def print(self):  # 实例方法
        print('instance method')

    @classmethod
    def class_print(cls):  # 类方法
        print(id(cls))
        print('class method')

    @staticmethod 
    def static_print():  # 静态方法
        print('static method')

    def xxx_print():  # 一个普通方法
        print('this is a function')
```

- 实例方法只能由实例调用
- 类方法可以被类和实例使用，并且被实例使用时，传入的第一个参数还是类
- 静态方法可以被类和实例使用，都不会传入第一个参数
- 类中的普通方法，因为没有传入self，因此只能被类使用，实例无法使用
- 各种方法根据首参来决定。

# 访问控制

## 双下划线

- 所有**双下划线开始，非双下划线结尾**的成员，都是私有成员
- 严格的说， Python里没有真正私有成员
- Python的私有成员是通过**改名**实现的：_类名 + 带双下划綫的属性
- **除非真的有必要，并且清除明白的知道会有什么后果，否则不要通过改名规则修改私有成员**

```python
In [1]: class Door:
   ...:     def __init__(self, number, status):
   ...:         self.number = number
   ...:         self.__status = status  # 双下划线开始， 非双下划綫结尾的都是私有的， 在类外部无法访问
   ...:     def open(self):
   ...:         self.__status = 'opening'
   ...:     def close(self):
   ...:         self.__status = 'closed'
   ...:     def status(self):
   ...:         return self.__status
   ...:     def __set_number(self, number):  # # 双下滑先开始， 非双下划线结尾的方法也是私有方法
   ...:         self.number = number
   ...:         

In [2]: door = Door(1001, 'closed')

In [3]: door.__status  # 无法访问私有属性
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-3-d55234f04e7f> in <module>()
----> 1 door.__status

AttributeError: 'Door' object has no attribute '__status'

In [4]: door.__dict__  # door对象含有的属性_Door__status
Out[4]: {'_Door__status': 'closed', 'number': 1001}

In [5]: door.__status = 'hahaha'  # 给对象创建了新的属性，并没有修改到__status

In [6]: door.__status
Out[6]: 'hahaha'

In [7]: door.__dict__
Out[7]: {'_Door__status': 'closed', '__status': 'hahaha', 'number': 1001}

In [8]: door.status()
Out[8]: 'closed'

In [9]: door.open()

In [10]: door.status()
Out[10]: 'opening'

In [11]: door.__set_number(1002)
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-11-888a73f63746> in <module>()
----> 1 door.__set_number(1002)

AttributeError: 'Door' object has no attribute '__set_number'

In [12]: door._Door__status
Out[12]: 'opening'

In [13]: door._Door__status = 'hehehe'  # _类名 + 带双下划綫的属性的方式直接修改私有成员

In [14]: door.status()
Out[14]: 'hehehe'
```

## 单下划线

- 单下划线是一种惯用法， 人为标记此成员为私有， 但是解释器不不做任何处理

```python
In [1]: class A:
   ...:     def __init__(self):
   ...:         self._a = 3
   ...:         

In [2]: a = A()

In [3]: a._a
Out[3]: 3

In [4]: a._a = 4

In [5]: a._a
Out[5]: 4

In [6]: a.__dict__
Out[6]: {'_a': 4}

```

## property装饰器

**引入property装饰器**

```python
class Door:
    def __init__(self, number):
        self.__number = number

    def get_number(self):
        return self.__number

    def set_number(self, number):
        self.__number = number
```

当把`number`属性变成私有属性`__number`之后，无法直接访问得到，只能通过`get_number`和`set_number`两个函数访问`__number`属性。

如果既能限制参数访问，又可以用类似属性这样简单的方式来访问类的变量，这个时候就可以使用property装饰器了。

- **Python内置的`@property`装饰器就是负责把一个方法变成属性调用的**

**property装饰器使用**

```python
class Door:
    def __init__(self, number):
        self.__number = number

    # property 装饰器会把一个仅有self参数的函数，变成一个属性， 属性的值，为方法的返回值
    @property
    def number(self):
        return self.__number

    # property setter 装饰器， 可以把一个方法转化为对此赋值，但此方法有一定要求
    # 1.同名 2.必须接收两个参数 self 和 value， value为所赋的值
    @number.setter
    def number(self, number):
        self.__number = number

    @number.deleter
    def number(self):
        print('cannot remove number property')

door = Door(1001)
door.number  # 返回1001
door.number = 1002
door.number  # 返回1002
del door.number  # 输出cannot remove number property
```

# 继承

## 单继承

- 在类名后加括号 括号中是继承列表， 称之为父类或者基类或者超类
- 继承一个明显的好处就是可以获取父类的属性和方法

```python
class Base:
    PUBLIC_CLASS_VAR = 'PUBLIC_CLASS_VAR'
    __PRIVATE_CLASS_VAR = 'PRIVATE_CLASS_VAR'

    def __init__(self):
        self.public_instance_var = 'public_instance_var'
        self.__private_instance_var = 'private__instance_var'

    @classmethod
    def public_class_method(cls):
        return 'public_class_method'

    @classmethod
    def __private_class_method(cls):
        return 'private_class_method'

    @staticmethod
    def public_static_method():
        return 'public static method'

    @staticmethod
    def __private_static_method():
        return 'private static method'

    def public_instance_method(self):
        return 'public_instance_method'

    def __private_instance_method(self):
        return 'private_instance_method'

class Sub(Base):
    pass

sub = Sub()
sub.__dict__
# 输出
{'_Base__private_instance_var': 'private__instance_var',
 'public_instance_var': 'public_instance_var'}
```

- 凡是公有的都能继承
- 凡是私有的都不能继承
- 原来是什么，继承过来还是什么

## 方法重写

- 当子类和父类有同名成员的时候， 子类的成员会覆盖父类的同名成员
- 当父类含有一个带参数的初始化方法的时候，子类一定需要一个初始化方法，并且在初始化方法中调用父类的初始化方法
- super方法：super(type, obj) =》type:类名，obj:传递给后续方法的第一个参数

```python
class Base:
    def __init__(self):
        self.__a = 4

    def print(self):
        print('Base.print')

    @classmethod
    def cls_print(cls):
        print('Base.cls_print')

class Sub(Base):
    def print(self):  ## 当子类和父类有同名成员的时候， 子类的成员会覆盖父类的同名成员
        print('Sub.print')

    @classmethod
    def cls_print(cls):
        print('Sub.cls_print')

    def foo(self):
        # 调用父类的print
        super().print()
        # super(Sub, self).print()

    @classmethod
    def cls_foo(cls):
        #cls.cls_print()
        #Base.cls_print()
        super().cls_print()

class SubSub(Sub):
    def print(self):
        print('SubSub.print')

    @classmethod
    def cls_print(cls):
        print('SubSub.cls_print')

    def foo(self):
        # 调用Base的print
        super(SubSub, self).print()
        # 代理 TYPE 的父类的方法， 并且使用 obj 绑定  第一个参数 指定调用谁的直接父类， 第二个参数指定当调用时，传递什么作为方法的第一个参数
        super(Sub, self).print()
        super(SubSub, SubSub).cls_print()  # 类方法的时候可以传递类，也可以传递实例self

    @classmethod
    def cls_foo(cls):
        # Base.cls_print()
        super(Sub, cls).cls_print()

```

## 多继承与MRO

### 等效的类定义

```python
class A:
    pass

class A(object):
    pass

class A():
    passs
```

### 多继承

- 在继承列表里存在多个类的时候表示多继承
- 多继承会把继承列表里的所有公有成员都继承过来

```python
class A:
    def method(self):
        print('method of A')

class B:
    def method(self):
        print('method of B')

class C(A, B):
    pass

c = C()
c.method()  # 输出method of A
```

### MRO

定义一个多继承，如下

```python
class A:
    def method(self):
        print('method of A')

class B:
    def method(self):
        print('method of B')

class C(A, B):
    pass

class E(A):
    def method(self):
        print('method of E')

class F(E, A):
    pass

F().method()  # 输出method of E
```

如果定义类G继承自(A, E)，如下

```python
class G(A, E):  # 在定义的时候会直接报错
    pass

---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-51-dcac33a3d00c> in <module>()
----> 1 class G(A, E):
      2     pass

TypeError: Cannot create a consistent method resolution
order (MRO) for bases E, A
```

报错显示：Cannot create a consistent method resolution order (MRO) for bases E, A

> 方法解析顺序(MRO)不满足报错

**分析基类E,A的MRO**

```python
>>> A.__mro__
(__main__.A, object)
>>> E.__mro__
(__main__.E, __main__.A, object)
>>> F.__mro__
(__main__.F, __main__.E, __main__.A, object)
```

所以，mro序列就是继承的先后顺序

那么G类的mro序列应该就是(G, A, E, object)，Python通过C3算法来确定多继承的时候是否满足**mro的两个原则**：

1. 本地优先： 自己定义或重写的方法优先，按照继承列表，从左到右查找
2. 单调性：所有子类，也要满足查找顺序

C3算法的主要作用是：在多继承时判断属性来自于哪个类，无法判断时抛出TypeError

**C3算法**

```
class B(O) ：则B的mro序列为： [B, O]
class B(A1, A2, ..., An) ：则B的mro序列为： [B] + merge(mro(A1), mro(A2), ..., mro(An), [A1, A2, ..., An, O])
```

merge操作就是C3算法的核心，merge步骤如下：

```markdown
* 遍历列表
* 看第一个列表的首元素
    * 它在其他列表中也是首元素
    * 或者它在其他列表不存在
* 满足以上条件，则移除该首元素，合并到mro中
* 不满足，则抛出异常
```

**C3算法分析F类的mro**

```
mro(F) -> [F] + merge(mro(E), mro(A), [E, A, O])
	-> [F] + merge([E, A, O], [A, O], [E, A, O])
    -> [F, E] + merge([A, O], [A, O], [A, O])
    -> [F, E, A] + merge([O], [O], [O])
    -> [F, E, A, O]
```

merge操作成功，mro解析正确，最终mro为[F, E, A, O]

**C3算法分析G类的mro**

```
mro(G) -> [G] + merge(mro(A), mro(E), [A, E, O])
	-> [G] + merge([A, O], [E, A, O], [A, E, O])
    -> raise TypeError:
```

第一个列表的首元素为A，在第二个列表中存在但不是首元素，不满足merge的条件，直接抛出异常。

**结论**：

1. **应该尽量避免多继承**
2. 多继承会对程序的心智负担造成非常大的压力

## Mixin类

参考

1. [廖雪峰-多重继承与MixIn](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014318680104044a55f4a9dbf8452caf71e8dc68b75a18000)
2. [知乎-Mixin是什么概念?](https://www.zhihu.com/question/20778853)
3. [Python Cookbook-利用Mixins扩展类功能](https://python3-cookbook.readthedocs.io/zh_CN/latest/c08/p18_extending_classes_with_mixins.html)

在编程中，mixin是指为继承自它的class提供额外的功能, 但它自身却是不单独使用的类.。在具有多继承能力的编程语言中， mixin可以为类增加额外功能或方法。

因此，MixIn模式的目的就是给一个类增加多个功能，这样，在设计类的时候，我们优先考虑通过多重继承来组合多个MixIn的功能，而不是设计多层次的复杂的继承关系。

在Python 3.5.2 源代码 socketserver.py 中的639到643行可以看到以下四个类的定义

```python
class ForkingUDPServer(ForkingMixIn, UDPServer): pass
class ForkingTCPServer(ForkingMixIn, TCPServer): pass

class ThreadingUDPServer(ThreadingMixIn, UDPServer): pass
class ThreadingTCPServer(ThreadingMixIn, TCPServer): pass
```

- BaseServer：server类的基类
- UDPServer：UDP server class，继承自BaseServer
- TCPServer：TCP server class，继承自BaseServer
- ForkingMixIn：Mix-in class to handle each request in a new process.
- ThreadingMixIn：Mix-in class to handle each request in a new thread.

Python自带了`TCPServer`和`UDPServer`这两类网络服务，而要同时服务多个用户就必须使用多进程或多线程模型，这两种模型由`ForkingMixIn`和`ThreadingMixIn`提供。通过组合，就可以得到以上四个类。

这几个类之间的关系如下图：

![socketserver继承关系](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/MixIn.jpg)

可以看到，从BaseServer开始逐层继承的过程中，混入(MixIn)了ForkingMixIn类和ThreadingMixIn类。

这样的多重继承的技巧称为MixIn。

如果不采用MixIn技术，而是采用层次复杂的单继承实现，则类的数量会呈指数增长。

具体不采用MixIn技术设计的继承层次关系参见：[廖雪峰-多重继承与MixIn](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014318680104044a55f4a9dbf8452caf71e8dc68b75a18000)中的Animal类的设计思路。

**MixIn总结**

MixIn其实也是一种组合的方式。通常来说，组合优于继承

Mixin 类的限制

- Mixin类不应该有初始化方法
- Mixin类通常不能独立工作
- Mixin类的祖先也应该是Mixin类

通常情况下，Mixin类总在继承列表的第一位