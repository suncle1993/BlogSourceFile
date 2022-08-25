---
categories:
  - java
date: '2019-01-22T21:52:33'
description: ''
tags:
  - wrapper class
  - primitive type
title: Best practice on when to use the wrapper class and primitive type in Java
---




> **Best practice on when to use the wrapper class and primitive type in Java**

四个概念：

- primitive type：原始类型
- wrapper class：包装类型
- autoboxing：自动包装
- unboxing：解包

对应关系：

| Primitive type | Wrapper class |
| -------------- | ------------- |
| boolean        | Boolean       |
| byte           | Byte          |
| char           | Character     |
| float          | Float         |
| int            | Integer       |
| long           | Long          |
| short          | Short         |
| double         | Double        |

在 Effective Java 的第五项中, Joshua Bloch 有这样的观点：

> The lesson is clear: **prefer primitives to boxed primitives, and watch out for unintentional autoboxing**.

意思就是：相对于 `boxed primitive` 更喜欢 `primitive`，并且需要注意无意识的 **autoboxing** 机制。

类的一个很好的用途是作为泛型类型（包括Collection类，比如list和map），或者当你想要将它们转化为其他类型而不进行隐式转换时（例如 `Intege类`具有方法 `doubleValue()` or `byteValue()`）。

因此，最佳实践是能使用primitive的都用primitive，除非你正在处理泛型（确保你知道 autoboxing 和 unboxing）

<!--more-->

# 使用 primitive

在以下几种情况下使用 primitive

## primitive 性能更好

```java
//隐式的降低程序速度
public static void main(String[] args) {
    Long sum = 0L;
    for (long i = 0; i < Integer.MAX_VALUE; i++) {
         sum += i;
    }
    System.out.println(sum);
}
```

这个程序虽然能得到正确 `sum` ，但是效率比理论中会慢很多。变量 `sum` 被定义成了Long而不是 `long` ，这就意味着程序构建了 2^31 次没必要的 `Long` 实例（每次 `long i` 被加到 `Long sum` 上时算一次）。将`sum` 的类型从 `Long` 改为 `long`，程序时间可以达到数量级的缩减。

## primitive 可读性更高

```java
Integer a = 2;
Integer b = 2;
if (!a.equals(b)) {
    // ...
}
int c = 2;
int d = 2;
if (c != d) {
    // ...
}
```

代码中的两种相等比较，多数人都会认为第二种更易读，对于先上手 C++ / Python 的人更是这样。

## 使用 primitive 可以避免一些错误

如果不了解 `wrapper class` 中的一些机制，会遇到一些莫名其妙的问题 

### 莫名其妙的 NullPointException

```java
Integer getValue();
...
int myValue = getValue(); // getValue返回null时就会抛出NPE
```

这个代码可以编译通过，但是会抛出空指针异常。`int b = a`实际上是`int b = a.intValue()`，由于a的引用值为null，在空对象上调用方法就会抛出NPE。

### wrapper class 的引用相等性

在Java中，`==` 符号判断的内存地址所对应的值的相等性，具体来说，基本类型判断值是否相等，引用类型判断其指向的地址是否相等。

```java
Integer a1 = 1;
Integer a2 = 1;
System.out.println(a1 == a2); // true

Integer b1 = 222;
Integer b2 = 222;
System.out.println(b1 == b2); // false
```

这两段代码的结果是不同的，具体需要看下 java.lang.Integer 的 `valueOf` 方法的源码：

```java
package java.lang;

import java.lang.annotation.Native;

public final class Integer extends Number implements Comparable<Integer> {

    /**
     * Returns an {@code Integer} instance representing the specified
     * {@code int} value.  If a new {@code Integer} instance is not
     * required, this method should generally be used in preference to
     * the constructor {@link #Integer(int)}, as this method is likely
     * to yield significantly better space and time performance by
     * caching frequently requested values.
     *
     * This method will always cache values in the range -128 to 127,
     * inclusive, and may cache other values outside of this range.
     *
     * @param  i an {@code int} value.
     * @return an {@code Integer} instance representing {@code i}.
     * @since  1.5
     */
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }

    /**
     * The value of the {@code Integer}.
     *
     * @serial
     */
    private final int value;
}
```

`valueOf` 方法会缓存 -128 到 127 之间的值，因此第一段代码会取到的是同一个对象，第二段代码会创建两个对象且地址不一样，因此导致的结果不同。

# 使用 wrapper class

使用泛型的时候必须使用 wrapper class，因为Java不支持使用基本类型作为类型参数

```Java
List<int> list; // 编译器会提示：Type argument cannot be of primitive type
List<Integer> list; // 这个就是正确的
```

---

参考：

- [Java-Oracle-Docs: Autoboxing and Unboxing](https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html)

- [wrapper class and primitive type](https://stackoverflow.com/questions/1570416/when-to-use-wrapper-class-and-primitive-type)

- [When to use primitive vs class in Java?](https://softwareengineering.stackexchange.com/questions/203970/when-to-use-primitive-vs-class-in-java)