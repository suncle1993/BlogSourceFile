---
categories:
  - 前端
date: '2017-04-18T23:00:20'
description: ''
tags:
  - JavaScript
  - 基础
title: JavaScript基础
---



JavaScript 从 Java 中借用其大部分语法，但也受 Awk，Perl 和 Python的影响。因此本篇博客基于对Java和Python的认识来记录JavaScript的差异点。

# [0x00] let,var,const

- var 的方式是为了兼容旧版。因为var定义变量， 会提升作用域，var定义的变量，都是全局作用域。


- let 是ES6引入的，如果没有特殊原因， 变量都应该是用let来定义
- const是定义常量

# [0x01] JavaScript数据类型

JavaScript总共有六种基本数据类型

- [Boolean](https://developer.mozilla.org/en-US/docs/Glossary/Boolean).  布尔值，`true` 和 `false`.
- [null](https://developer.mozilla.org/en-US/docs/Glossary/null). 一个表明 null 值的特殊关键字。 JavaScript 是大小写敏感的，因此 `null `与 `Null`、`NULL`或其他变量完全不同。
- [undefined](https://developer.mozilla.org/en-US/docs/Glossary/undefined).  变量未定义时的属性。
- [Number](https://developer.mozilla.org/en-US/docs/Glossary/Number).  表示数字，例如： `42` 或者 `3.14159。`
- [String](https://developer.mozilla.org/en-US/docs/Glossary/String).  表示字符串，例如："Howdy"
- [Symbol](https://developer.mozilla.org/en-US/docs/Glossary/Symbol) ( 在 ECMAScript 6 中新添加的类型).。一种数据类型，它的实例是唯一且不可改变的。

> 在bool运算中，null和undefined都等价于false

除了六种基本数据类型之外，还有复合的object类型。


<!--more-->
# [0x02] 字符串单引号，双引号和倒引号

- 字符串可以用双引号，也可以用单引号， 没有任何区别
- 倒引号定义的字符串可以写在多行
- 倒引号定义的字符串可以插值，使用 ${name}的方式，把变量插入到字符串中，借鉴于Perl

**${name}代码**

```javascript
let x = 'world'
let z = `hello ${x}`
console.log(z)  // 输出hello world
```

# [0x03] 自增自减运算符

 两个连续加号表示自增操作，两个连续减号表示自减操作。借鉴于Java。

以自增操作为例：

- 加号在后，表示先求值，后自增


- 加号在前，表示先自增，后求值

# [0x04] 双等号和三等号

console.log( '3' == 3);  // == 转化为相同类型之后再比较值，也就是不比较类型

console.log( '3' === 3); // === 值和类型都比较，所以常用===

!= 和 !== 的规则同 == 和 === 的规则

# [0x05] 作用域

- Python的作用域以def为最小单位
- 从ES6开始， es开始支持块级作用域。JavaScript的大括号就会开始一个新的语句块

```javascript
let x = 3; {
    let x = 4;
    console.log(`x in block is ${x}`);  // x=4
}
console.log(`x is ${x}`)  // x=3

let x = 3; {
	x = x + 1
    console.log(`x in block is ${x}`)  // x=3 外层变量可以传入内层
}
console.log(`x is ${x}`)  // x=3
```

# [0x06] 分支

**if语句**

- if 语句可以不带else子句


- if 语句可以带else子句实现双分支


- if 语句可以带else if 子句实现多分支

```javascript
if (condition_1) {
  statement_1;
}
[else if (condition_2) {
  statement_2;
}]
...
[else if (condition_n_1) {
  statement_n_1;
}]
[else {
  statement_n;
}]
```

**switch语句**

和java类似

```python
switch (expression) {
   case label_1:
      statements_1
      [break;]
   case label_2:
      statements_2
      [break;]
   ...
   default:
      statements_def
      [break;]
}
```

# [0x07] 循环

- c风格for循环


- for..in循环
- for..of循环
- for each..in循环：目前已禁用
- do..while循环
- while循环

c风格for循环和两种while循环的区别都是c语言风格的，Java也类似。再次单独介绍for..in和for..of循环

**for..in和for..of循环**

for of是ES6新加的语法，用来遍历数组元素值，而for in是用来遍历对象的索引。代码如下：

```javascript
// for in会遍历对象所有的属性,即会遍历数组的元素以及属性
let myArray = [1, 2, 3, 4, 5, 6, 7]
myArray.name = "数组"
for (let index in myArray) {
    console.log(myArray[index]);  // 会输出myArray的name属性
}

// for of遍历的只是数组内的元素，而不包括数组的原型属性method和索引name
let myarray = [1, 2, 3, 4, 5, 6, 7]
myarray.name = '数组'
for (let value of myarray) {
    console.log(value)
}
```

# [0x08] 解构

JavaScript的解构借鉴于Python，但是和Python相比JavaScript的解构更加强大。

**Array解构**

```javascript
// 普通解构
arr = [1, 2]
const [a, b] = arr
console.log(a, b)  // 输出1 2


// 普通解构支持默认值
arr = [1]
const [a=3, b=4] = arr
console.log(a, b)  // 输出1 4
```

**Object解构**

Python不支持字典解构，但是JavaScript支持的对象解构就包含字典解构，只是在JavaScript中不叫字典。

```javascript
// object 解构
obj = {
    a: 1,
    b: 2,
    c: 3
}
const {a: A, b: B, c: C} = obj
console.log(A, B, C)  // 输出1 2 3

// object 解构支持默认值
obj = {
    a: 1,
    b: 2,
    c: 3
}
const {a: A, b: B, c: C, d: D = 18} = obj
console.log(A, B, C, D)  // 输出1 2 3 18
```

**嵌套解构**

```javascript
// 嵌套解构
arr = [1, [2, 3], 4]
const [a, [b, c ,d = 18]] = arr
console.log(a, b, c, d)  // 输出1 2 3 18
```

更复杂的嵌套解构如下（MDN上的一个例子）：

```javascript
// 解构Json文件
const metadata = {
    title: "Scratchpad",
    translations: [
       {
        locale: "de",
        localization_tags: [ ],
        last_edit: "2014-04-14T08:43:37",
        url: "/de/docs/Tools/Scratchpad",
        title: "JavaScript-Umgebung"
       }
    ],
    url: "/en-US/docs/Tools/Scratchpad"
};

const {title: Title, translations: [{title: TransTitle}]} = metadata
console.log(Title, TransTitle)  // 输出Scratchpad JavaScript-Umgebung
```

---

参考资料

1. [MDN-JavaScript指南](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide)
2. [W3S-JavaScript 教程](https://www.w3school.com.cn/js/index.asp)
3. [MDN-解构赋值](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)