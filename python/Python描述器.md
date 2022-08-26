---
abbrlink: 1516127351
alias: 2017/03/16/Python描述器/index.html
categories:
- python
date: '2017-03-16T23:01:23'
description: ''
tags:
- Python
- 描述器
title: Python描述器
---








# 引入描述器

以stackoverflow上关于描述器（descriptor ）的疑问开篇。

```python
class Celsius:

    def __get__(self, instance, owner):
        return 5 * (instance.fahrenheit - 32) / 9

    def __set__(self, instance, value):
        instance.fahrenheit = 32 + 9 * value / 5


class Temperature:

    celsius = Celsius()

    def __init__(self, initial_f):
        self.fahrenheit = initial_f


t = Temperature(212)
print(t.celsius)  # 输出100.0
t.celsius = 0
print(t.fahrenheit)  # 输出32.0
```

以上代码实现了温度的摄氏温度和华氏温度之间的自动转换。其中Temperature类含有实例变量fahrenheit和类变量celsius，celsius由描述器Celsius进行代理。由这段代码引出的三点疑问：

1. 疑问一：什么是描述器？
2. 疑问二：`__get__`,`__set__`,`__delete__`三种方法的参数
3. 疑问三：描述器有哪些应用场景
4. 疑问四：property和描述器的区别是什么？

<!--more-->

# 疑问一：什么是描述器？

描述器是一个 实现了 `__get__`、 `__set__`和`__delete__`中1个或多个方法的类对象。当一个类变量指向这样的一个装饰器的时候， 访问这个类变量会调用`__get__` 方法， 对这个类变量赋值会调用`__set__ `方法，这种类变量就叫做描述器。

描述器 事实上是一种代理机制：当一个**类变量**被定义为描述器，对这个类变量的操作，将由此描述器来代理。

# 疑问二：描述器三种方法的参数

```python
class descriptor:
    def __get__(self, instance, owner):
        print(instance)
        print(owner)
        return 'desc'

    def __set__(self, instance, value):
        print(instance)
        print(value)

    def __delete__(self, instance):
        print(instance)

class A:
    a = descriptor()

del A().a  # 输出<__main__.A object at 0x7f3fc867cbe0>
A().a  # 返回desc，输出<__main__.A object at 0x7f3fc86741d0>，<class '__main__.A'>
A.a  # 返回desc，输出None，<class '__main__.A'>
A().a = 5  # 输出<__main__.A object at 0x7f3fc86744a8>，5
A.a = 5  # 直接修改类A的类变量，也就是a不再由descriptor描述器进行代理。
```

由以上输出结果可以得出结论：

## 参数解释

- `__get__(self, instance, owner)` instance 表示当前实例 owner 表示类本身, 使用类访问的时候， instance为None
- `__set__(self, instance, value)` instance 表示当前实例, value 右值， 只有实例才会调用 `__set__`
- `__delete__(self, instance)` instance 表示当前实例

## 三种方法的本质

- 访问：`instance.descriptor`实际是调用了`descriptor.__get__(self, instance, owner)`方法，并且需要返回一个value
- 赋值：`instance.descriptor = value`实际是调用了`descriptor.__set__(self, instance, value)`方法，返回值为None。
- 删除：`del instance.descriptor`实际是调用了`descriptor.__delete__(self, obj_instance)`方法，返回值为None

# 疑问三：描述器有哪些应用场景

> 我们想创建一种新形式的实例属性，除了修改、访问之外还有一些额外的功能，例如 类型检查、数值校验等，就需要用到描述器 《Python Cookbook》

即描述器主要用来接管对实例变量的操作。

## 实现classmethod装饰器

```python
from functools import partial
from functools import wraps

class Classmethod():
    def __init__(self, fn):
        self.fn = fn

    def __get__(self, instance, owner):
        return wraps(self.fn)(partial(self.fn, owner))
```

将方法fn的第一个参数固定成实例的类。可参考python官方文档的另一种写法：[descriptor](https://docs.python.org/3.5/howto/descriptor.html#static-methods-and-class-methods)

```python
class ClassMethod(object):
    def __init__(self, fn):
        self.fn = fn

    def __get__(self, instance, owner=None):
        if owner is None:
            owner = type(obj)
        def newfunc(*args):
            return self.f(owner, *args)
        return newfunc
```

## 实现staticmethod装饰器

```python
class Staticmethod:
    def __init__(self, fn):
        self.fn = fn

    def __get__(self, instance, cls):
        return self.fn
```

## 实现property装饰器

```python
class Property:
    def __init__(self, fget, fset=None, fdel=None, doc=''):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel
        self.doc = doc

    def __get__(self, instance ,owner):
        if instance is not None:
            return self.fget(instance)
        return self

    def __set__(self, instance, value):
        if not callable(self.fset):
            raise AttibuteError('cannot set')
        self.fset(instance, value)

    def __delete__(self, instance):
        if not callable(self.fdel):
            raise AttributeError('cannot delete')
        self.fdel(instance)

    def setter(self, fset):
        self.fset = fset
        return self

    def deleter(self, fdel):
        self.fdel = fdel
        return self
```

使用自定义的Property来描述farenheit和celsius类变量：

```python
class Temperature:
    def __init__(self, cTemp):
        self.cTemp = cTemp  # 有一个实例变量cTemp：celsius temperature

    def fget(self):
        return self.celsius * 9 /5 +32

    def fset(self, value):
        self.celsius = (float(value) -32) * 5 /9

    def fdel(self):
        print('Farenhei cannot delete')

    farenheit = Property(fget, fset, fdel, doc='Farenheit temperature')

    def cget(self):
        return self.cTemp

    def cset(self, value):
        self.cTemp = float(value)

    def cdel(self):
        print('Celsius cannot delete')

    celsius = Property(cget, cset, cdel, doc='Celsius temperature')
```

使用结果：

```python
t = Temperature(0)
t.celsius  # 返回0.0
del t.celsius  # 输出Celsius cannot delete
t.celsius = 5
t.farenheit  # 返回41.0
t.farenheit = 212
t.celsius  # 返回100.0
del t.farenheit  # 输出Farenhei cannot delete
```

使用装饰器的方式来装饰Temperature的两个属性farenheit和celsius：

```python
class Temperature:
    def __init__(self, cTemp):
        self.cTemp = cTemp

    @Property  # celsius = Property(celsius)
    def celsius(self):
        return self.cTemp

    @celsius.setter
    def celsius(self, value):
        self.cTemp = value

    @celsius.deleter
    def celsius(self):
        print('Celsius cannot delete')

    @Property  # farenheit = Property(farenheit)
    def farenheit(self):
        return self.celsius * 9 /5 +32

    @farenheit.setter
    def farenheit(self, value):
        self.celsius = (float(value) -32) * 5 /9

    @farenheit.deleter
    def farenheit(self):
        print('Farenheit cannot delete')
```

使用结果同直接用描述器描述类变量

## 实现属性的类型检查

首先实现一个类型检查的描述器Typed

```python
class Typed:
    def __init__(self, name, expected_type):
        # 每个属性都有一个名称和对应的类型
        self.name = name
        self.expected_type = expected_type

    def __get__(self, instance, cls):
        if instance is None:
            return self
        return instance.__dict__[self.name]

    def __set__(self, instance ,value):
        if not isinstance(value, self.expected_type):
            raise TypeError('Attribute {} expected {}'.format(self.name, self.expected_type))
        instance.__dict__[self.name] = value

    def __delete__(self, instance):
        del instance.__dict__[self.name]
```

然后实现一个Person类，Person类的属性name和age都由Typed来描述

```python
class Person:
    name = Typed('name', str)
    age = Typed('age', int)

    def __init__(self, name: str, age: int):
        self.name = name
        self.age = age
```

类型检查过程：

```python
>>> Person.__dict__
mappingproxy({'__dict__': <attribute '__dict__' of 'Person' objects>,
              '__doc__': None,
              '__init__': <function __main__.Person.__init__>,
              '__module__': '__main__',
              '__weakref__': <attribute '__weakref__' of 'Person' objects>,
              'age': <__main__.Typed at 0x7fe2f440bd68>,
              'name': <__main__.Typed at 0x7fe2f440bc88>})
>>> p = Person('suncle', 18)
>>> p.__dict__
{'age': 18, 'name': 'suncle'}
>>> p = Person(18, 'suncle')
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-88-ca4808b23f89> in <module>()
----> 1 p = Person(18, 'suncle')

<ipython-input-84-f876ec954895> in __init__(self, name, age)
      4 
      5     def __init__(self, name: str, age: int):
----> 6         self.name = name
      7         self.age = age

<ipython-input-83-ac59ba73c709> in __set__(self, instance, value)
     11     def __set__(self, instance ,value):
     12         if not isinstance(value, self.expected_type):
---> 13             raise TypeError('Attribute {} expected {}'.format(self.name, self.expected_type))
     14         instance.__dict__[self.name] = value
     15 

TypeError: Attribute name expected <class 'str'>
```

但是上述类型检查的方法存在一些问题，Person类可能有很多属性，那么每一个属性都需要使用Typed描述器描述一次。我们可以写一个带参数的类装饰器来解决这个问题：

```python
def typeassert(**kwargs):
    def wrap(cls):
        for name, expected_type in kwargs.items():
            setattr(cls, name, Typed(name, expected_type))  # 经典写法
        return cls
    return wrap
```

然后使用typeassert类装饰器重新定义Person类：

```python
@typeassert(name=str, age=int)
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
```

可以看到typeassert类装饰器的参数是传入的属性名称和类型的键值对。

如果我们想让typeassert类装饰器自动的识别类的初始化参数类型，并且增加相应的类变量的时候，我们就可以借助inspect库和python的类型注解实现了：

```python
import inspect
def typeassert(cls):
    params = inspect.signature(cls).parameters
    for name, param in params.items():
        if param.annotation != inspect._empty:
            setattr(cls, name, Typed(name, param.annotation))
    return cls

@typeassert
class Person:
    def __init__(self, name: str, age: int):  # 没有类型注解的参数不会被托管
        self.name = name
        self.age = age
```

# 疑问四：property和描述器的区别

我们可以利用Python的内部机制获取和设置属性值。总共有三种方法：

1. Getters和Setter。我们可以使用方法来封装每个实例变量，获取和设置该实例变量的值。为了确保实例变量不被外部访问，可以把这些实例变量定义为私有的。所以，访问对象的属性需要通过显式函数：anObject.setPrice（someValue）; anObject.getValue（）。
2. property。我们可以使用内置的property函数将getter，setter（和deleter）函数与属性名绑定。因此，对属性的引用看起来就像直接访问那么简单，但是本质上是调用对象的相应函数。例如，anObject.price = someValue; anObject.value。
3. 描述器。我们可以将getter，setter（和deleter）函数绑定到一个单独的类中。然后，我们将该类的对象分配给属性名称。这时候对每个属性的引用也像直接访问一样，但是本质上是调用这个描述器对象相应的方法，例如，anObject.price = someValue; anObject.value。

Getter和Setter这种设计模式不够Pythonic，虽然在C++和JAVA中很常见，但是Python追求的是简介，追求的是能够直接访问。

---

**附1、data-descriptor and no-data descriptor**

翻译为中文其实就是资料描述器和非资料描述器

- data-descriptor：同时实现了`__get__`和`__set__`方法的描述器
- no-data descriptor：只实现了`__get__`方法的描述器

两者的区别在于：

- no-data descriptor的优先级低于`instance.__dict__`

```python
class Int:
    def __get__(self, instance, cls):
        return 3

class A:
    val = Int()

    def __init__(self):
        self.__dict__['val'] = 5

A().val  # 返回5
```

- data descriptor的优先级高于`instance.__dict__`

```python
class Int:
    def __get__(self, instance, cls):
        return 3

    def __set__(self, instance, value):
        pass

class A:
    val = Int()

    def __init__(self):
        self.__dict__['val'] = 5

A().val  # 返回3
```

**附2、描述器机制分析资料：**

1. [官方文档-descriptor](https://docs.python.org/3.5/howto/descriptor.html)
2. [understanding-get-and-set-and-python-descriptors](http://stackoverflow.com/questions/3798835/understanding-get-and-set-and-python-descriptors)
3. [anyisalin - Python - 描述器](https://anyisalin.github.io/2017/03/08/python-descriptor/)
4. [Python描述器引导(翻译)](http://pyzh.readthedocs.io/en/latest/Descriptor-HOW-TO-Guide.html#id8)
5. [Properties and Descriptors](http://www.linuxtopia.org/online_books/programming_books/python_programming/python_ch25.html)