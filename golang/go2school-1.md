---
abbrlink: 108767130
alias: 2018/10/31/go2school-1/index.html
categories:
- golang
date: '2018-10-31T20:26:54'
description: ''
tags:
- Go
title: go2school-1
---









我司大佬[紫月苏](http://www.liriansu.com/)最近在QCon上听了关于Go语言的洗脑报告，回来之后给各位普及了go的一些基本情况和未来发展，感觉大家兴致很浓，于是就在我司内部gitlab上开了一个新的repo，叫做go2school，也就是Go语言学习计划。又因为对Java实在无爱，然后妥妥的就加入了Go的大军。

下面是我司学习Go的Chapter 1. Grammer。具体就是阅读 [**Learn Go in Y minutes**](https://learnxinyminutes.com/docs/zh-cn/go-cn/) 学习基本语法。然后练习：

- 用 Go 语言实现一个计算斐波那契数列的函数，从标准输入读入 n, 从标准输出返回 f(n)。（递归版、循环版都要实现）

<!--more-->

# 交作业

斐波那契数列的go语言实现：递归版和循环版

```go
package main // it means this package is an executable file, not a repo

import "fmt" // import fmt standard package

// fib function, by recursive
func fibRe(n int) (m int) {
	if n == 0 || n == 1 {
		return 1
	}
	return fib(n-2) + fib(n-1)
}

// fib function,  by loop
func fib(n int) (m int) {
	if n == 0 || n == 1 {
		return 1
	}
	var a, b, c int
	a = 1
	b = 1
	for i := 1; i < n; i++ {
		c = a + b
		a, b = b, c
	}
	return b
}

// program entry
func main() {
	var n, m int
	fmt.Scan(&n) // read from stdin
	m = fib(n)
	fmt.Printf("m: %d\n", m)
	m = fibRe(n)
	fmt.Printf("m: %d\n", m)
}
```

# 感觉不错

在Learn Go in Y minutes这个网站上学习基本的语法感觉很nice，一个完整的用例中几乎廊括了所有的语法，看完基本语法找到斐波拉契数列开始牛刀小试了，感觉很不错。和之前写C++的感觉很像，一种熟悉的感觉。

# 一些问题

- 代码格式化
- Goland配置

## 代码格式化

Go语言的格式化感觉很nice，我就不喜欢每个人都有自己的独特的风格，再次鄙视一下那些喜欢用vim还不格式化代码的人，哈哈。个人还是比较同意有约束整体才更容易前进。

使用Goland这个IDE写Go时可以在preference中设置使用gofmt这个格式化工具。每次在保存代码的时候都会格式化，感觉很赞，这个体验比写C++好

## GoLand配置

需要配置如下：

- GoROOT
- GoPATH
- 还有一些别的没有配的后续再看

本次就先水到这里了

---

参考：

1. https://learnxinyminutes.com/docs/zh-cn/go-cn/
