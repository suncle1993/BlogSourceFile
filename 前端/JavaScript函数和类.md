---
abbrlink: 3769256843
alias: 2017/04/18/JavaScript函数和类/index.html
categories:
- 前端
date: '2017-04-18T23:00:29'
description: ''
tags:
- 函数
- 类
- JavaScript
title: JavaScript函数和类
---








# JavaScript函数

## 函数定义

一个**函数定义**（也称为**函数声明**，或**函数语句**）由一系列的[`函数`](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Statements/function)关键字组成, 依次为：

- 函数的名称。
- 函数参数列表，包围在括号( )中并由逗号( , )区隔。
- 函数功能，包围在花括号{ }中，用于定义函数功能的一些JavaScript语句。

```javascript
function add(x, y) {
    return x + y;
}
```

函数参数传递时，如果是传值则是传形参。如果是传对象，则是传引用（函数内部对对象的改变对外部是可见的）。

## 函数表达式

虽然上面的函数声明在语法上是一个语句，但函数也可以由**函数表达式**创建。这样的函数可以是**匿名**的；它不必有一个名称。例如，函数`square`也可这样来定义：

```javascript
// 第一种定义方式
const add = function(x, y) {
    return x + y;
}

// 第二种定义方式
const add = function _add(x, y) {
    return x + y;
}
```

函数表达式可以直接调用：

```javascript
let val = function (x, y) {
    return x + y
}(1, 3)

console.log(val);  // 输出4
```

函数表达式就是把一个函数赋值给变量或者常量

**什么时候使用函数表达式？什么时候使用命名方式的函数表达式？**

<!--more-->

- 当存在递归的时候，应该以命名函数表达式的方式定义函数
- 不存在递归时，习惯使用函数表达式

```javascript
// 演示1.命名方式定义函数,赋值给常量时，递归函数正常执行
function fib(x) {
    if (x < 2) {
        return 1
    } else {
        return fib(x - 1) + fib(x - 2)
    }
}
const fib2 = fib
console.log(fib2(5))
delete fib
console.log(fib2(5))


// 演示2.命名方式定义函数,赋值给变量，递归函数正常执行
function fib(x) {
    if (x < 2) {
        return 1
    } else {
        return fib(x - 1) + fib(x - 2)
    }
}
let fib2 = fib
console.log(fib2(5))
delete fib
console.log(fib2(5))


// 演示3.函数表达式方式定义递归函数时，有可能会存在问题
let fib = function(x) {
    if (x < 2) {
        return 1
    } else {
        return fib(x - 1) + fib(x - 2)
    }
}
const fib2 = fib
console.log(fib2(5))
//delete fib
fib = function() {}
console.log(fib2(5))  // 输出NaN		(NaN表示not a number)
```

## 高阶函数

JavaScript的高阶函数的定义和Python是一样的，只是JavaScript函数的参数可以直接写上函数的实现部分，而Python最多可以写上一个lambda函数。

```javascript
// 高阶函数：函数作为参数
map = function(arr, fn) {
    const result = []
    for (let a of arr) {
        result.push(fn(a))
    }
    return result
}
let square = function(x) {
    return x * x
}
const val = map([1, 2, 3, 4], square)
console.log(val)


// 高阶函数：函数实现直接作为参数
map = function(arr, fn) {
    const result = []
    for (let a of arr) {
        result.push(fn(a))
    }
    return result
}
const val = map([1, 2, 3, 4], function(x) {
    return x * x
})
console.log(val)
```

## 箭头函数

[箭头函数表达式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)（也称胖箭头函数, **fat arrow function**）主要作用是是函数表达式更为简洁。

- 箭头函数总是匿名的
- 当箭头函数至少有一个参数时，可以省去小括号
- 当箭头函数只有一条语句时，可以省去大括号和return

```javascript
map = function(arr, fn) {
    const result = []
    for (let a of arr) {
        result.push(fn(a))
    }
    return result
}

// fn可以改写为箭头函数如下
const val1 = map([1, 2, 3, 4], x => {
    console.log(x)
    return x * x
})

// 更精简的箭头函数
const val2 = map([1, 2, 3, 4], x => x * x)
```

## 函数参数

**默认参数**

```javascript
const add = function (x, y = 5) {
    return x + y;
}
console.log(add(1))  // 6

// 到目前为止JavaScript仍然不支持位置参数跟在默认参数的后面
const add = function (x = 5, y) {
    return x + y;
}
console.log(add(y = 1))  // NaN
```

**可变参数**

参数前加`...`表示其是可变参数，可变参数在函数体内，表现为一个数组。

```javascript
// 求所有参数的和
const sum = function(...args) {
    let ret = 0;
    for (let v of args) {
        ret += v
    }
    return ret
}
const val  = sum(1, 2, 3)
console.log(val)
```

**参数结构**

```javascript
const add = function(x, y) {
    return x + y
}
console.log(add(...[1, 2]))  // 3
```

而且不能对object做参数解构，因为JavaScript还不支持关键字参数。

# JavaScript类

## 基本使用

- 使用class关键字定义类
- constructor方法是构造方法
- 使用new关键字创建对象，参数为constructor方法的参数
- 实例调用静态方法的时候需要通过constructor属性

**代码**：下面的代码会定一个点类Point

```javascript
class Point {
  	// 构造方法
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }

  	// 普通方法
    print() {
        console.log(`<${this.x}, ${this.y}>`)
    }

	// 静态方法
    static info() {
        console.log('this is static method')
    }
}

// 可以使用类直接调用静态方法
Point.info()

// 创建Point类的对象point
let point = new Point(1, 2)

// 对象调用普通方法
point.print()

// 实例不能直接调用静态方法
// point.info()	// 会报错：不存在point.info函数

// 实例的constructor属性等价于Python的__class__
point.constructor.info()
```

## 类表达式

和函数一样，类除了有上面的命名方式的定义之外，还可以有类表达式。

```javascript
const MyClass = class Me {
    getClassName() {
        return Me.name;
    }
};
const inst = new MyClass();

console.log(inst.getClassName()); // Me
console.log(Me.name); // ReferenceError: Me is not defined
```

类表达式本质上就是把一个类赋值给一个变量。

## 类的继承

- 继承采用extends关键字，借鉴于Java

```javascript
class Point3D extends Point {
    constructor(x, y, z) { // 重写构造函数
        console.log(`x: ${x}, y: ${y}`)
        // this.z = x + y  this 不能再super之前使用
        super(x, y) // 调用父类的构造函数
        this.z = z
        this.format = this.format.bind(this)
    }

    format() {
        return super.format() + 'xxx'
    }
}

let p = new Point3D(3, 5, 8)
p.print()
```

## 多继承-MixIn模式

JavaScript中子类的使用有两个原因：

- 接口继承：子类的实例肯定也是超类的实例（可以用instanceof运算符测试得到这条结论）。子类实例的行为类似于超类实例。但和超类实例相比，可以有一些额外的功能（即方法）。


- 实现继承：超类将功能传递给子类。实现继承的类的作用是有限的，因为只支持单继承，不可能从多个超类继承。

用通俗的话举例子来理解接口继承和实现继承，下面有三个类，分别是：

- Person：人类
- Storage：数据存储类，拥有一个存储数据的方法save
- Validation：数据验证类，拥有一个验证数据的方法validate

**代码**

```javascript
class Person { 

 }

class Storage {
    save(data) { 
        console.log('store data')
     }
}
class Validation {
    validate(schema) { 
        console.log('validate data')
     }
}
```

现在要定义一个职工类Employee ，那么这个职工类肯定是要继承自Person类的（难道你敢说职工不是人？），同时这个职工还需要有两个能力：一个是存储数据的能力，一个是验证数据的能力。那么就又需要继承自Storage类和Validation类。那么Employee 类从Person类继承就是接口继承，因为超类和子类的行为类似。Employee 类从Storage继承或者从Validation继承都是实现继承，因为超类只是将功能传递给子类。

如果我们想实现这样的一个Employee类，那么一个很自然的写法就是多继承，下面的这种写法虽然很自然，但是多数语言都是不支持的，因为多重继承的时候会出现继承冲突。关于多重继承的冲突举一个简单的例子：定义一个动物（类）既是狗（父类1）也是猫（父类2），两个父类都有“叫”这个方法。那么当我们调用“叫”这个方法时，它就不知道是狗叫还是猫叫了，这就是多重继承的冲突。

```javascript
// 在定义的时候就会抛出SyntaxError
class Employee extends Person, Storage, Validation {

}
em = new Employee()
em.save()
em.validate()
```

为了实现多继承，ES6中有自己独特的MinIn技术：将实现继承的类视作一个函数，输入是超类，输出是扩展该超类的子类

```javascript
class Person { 

 }

const Storage = Sup => class extends Sup {
    save(data) { 
        console.log('store data')
     }
}

const Validation = Sup => class extends Sup {
    validate(schema) { 
        console.log('validate data')
     }
}

class Employee extends Storage(Validation(Person)) { 

}

em = new Employee()
em.save()   // 输出store data
em.validate()   // 输出validate data
```

通过这样的MixIn技术给Person类混入了Storage类的save方法和Validation类的validate方法，成功的变相的实现了多继承。

下面再举一个例子

- Point类
- 可序列化类Serializable
- Point3D类：需要继承自Point类，然后还需要混入可序列化的功能

**代码**

```javascript
class Point {
    constructor(x ,y) {
        this.x = x
        this.y = y
    }

    print() {
        console.log(`(${this.x}, ${this.y})`)
    }
}

const Serializable  = Sup => class extends Sup {
    constructor(...args) {
        super(...args)
        if (typeof this.constructor.stringify !== 'function') {
            throw new ReferenceError('must be define stringify')
        }

        if (typeof this.constructor.parse !== 'function') {
            throw new ReferenceError('must be define parse')
        }
    }

    toString() {
        return this.constructor.stringify(this)
    }
}

class Point3D extends Serializable(Point){
    constructor(x, y, z) {
        super(x, y)
        this.z = z
    }

    static stringify(obj) {
        let {x, y, z} = obj
        return JSON.stringify({x, y, z})    // 返回值是一个string
    }

    static parse(data) {
        let {x, y, z} = JSON.parse(data)
        return new Point3D(x, y, z)
    }

}

p3d_obj = new Point3D(2,5,19)
str = p3d_obj.toString()
console.log(str) // {"x":2,"y":5,"z":19}
console.log(typeof str)  // string

new_P3d = Point3D.parse(str)	// //通过序列化反序列化复制对象
console.log(new_P3d)    // Point3D { x: 2, y: 5, z: 19 }
console.log(new_P3d instanceof Point3D) // true
console.log(new_P3d instanceof Point)   // true
console.log(new_P3d == p3d_obj)    // false
```

Serializable 在语义上变成一种装饰，用来装饰Person类，即 Employee 是一种可序列化的 Person。这种MixIn的思想就是Python装饰器在JavaScript里面的应用了，只是JavaScript没有像Python一样用语法糖的形式来实现。

---

参考：

1. [MDN-函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Functions)
2. [MDN-类](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes)
3. [**simple-mixins**](http://exploringjs.com/es6/ch_classes.html#sec_simple-mixins)
4. [月影大神-类的装饰器：ES6 中优雅的 mixin 式继承](https://www.h5jun.com/post/mixin-in-es6.html)
5. [ECMAScript 6 Class](http://www.w3cschool.cn/ecmascript/e7yk1q5x.html)
6. [ECMAScript 6入门](http://www.nodeclass.com/api/ECMAScript6.html)