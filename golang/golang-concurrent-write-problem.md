---
categories:
  - golang
date: '2021-08-16T19:34:34'
tags:
  - 并发
title: Golang Concurrent Write Problem
---


分享几个golang并发写入的坑

## 并发读写map

在golang的实际项目中经常需要并发写数据，并且将数据塞到一个map中作为一个整体返回。

分为2种情况：这两种情况都会造成panic

1. 并发读写map
2. 并发写map

并发读写map的示例代码：

```golang
package util

import "testing"

func TestMap(t *testing.T) {
	m := make(map[int]int)

	go func() {
		// 不停地对map进行写入
		for {
			m[1] = 1
		}
	}()

	go func() {
		// 不停地对map进行读取
		for {
			_ = m[1]
		}
	}()
  
	select {}
}
```

运行代码之后会报错：`fatal error: concurrent map read and map write`

<!--more-->

错误信息显示，并发的 map 读和 map 写，也就是说使用了两个并发函数不断地对 map 进行读和写而发生了竞态问题，map 内部会对这种并发操作进行检查并提前发现。

并发写map的示例代码：

```golang
func TestMap(t *testing.T) {
	m := make(map[int]int)

	go func() {
		// 不停地对map进行写入
		for {
			m[1] = 1
		}
	}()

	go func() {
		// 不停地对map进行写入
		for {
			m[1] = 1
		}
	}()

	// 无限循环, 让并发程序在后台执行
	select {}
}
```

运行代码之后会报错：`fatal error: concurrent map writes`

错误信息显示，并发的 map 写，也就是说使用了两个并发函数不断地对 map 进行写而发生了竞态问题

需要并发读写或者并发写时，一般的做法是加锁，但这样性能并不高，Go语言在 1.9 版本中提供了一种效率较高的并发安全的 sync.Map，sync.Map 和 map 不同，不是以语言原生形态提供，而是在 sync 包下的特殊结构。

sync.Map 有以下特性：

- 无须初始化，直接声明即可。
- sync.Map 不能使用 map 的方式进行取值和设置等操作，而是使用 sync.Map 的方法进行调用，Store 表示存储，Load 表示获取，Delete 表示删除。
- 使用 Range 配合一个回调函数进行遍历操作，通过回调函数返回内部遍历出来的值，Range 参数中回调函数的返回值在需要继续迭代遍历时，返回 true，终止迭代遍历时，返回 false。

并发安全的 sync.Map 演示代码如下：

```golang
func TestSyncMap(t *testing.T) {
	var m sync.Map

	go func() {
		// 不停地对map进行写入
		for {
			m.Store(1, 1)
		}
	}()

	go func() {
		// 不停地对map进行读取
		for {
			_, _ = m.Load(1)
		}
	}()
	// 无限循环, 让并发程序在后台执行
	select {}
}
```

这段代码会无限循环，并且不会有并发读写的错误

sync.Map 没有提供获取 map 数量的方法，替代方法是在获取 sync.Map 时遍历自行计算数量，sync.Map 为了保证并发安全有一些性能损失，因此在非并发情况下，使用 map 相比使用 sync.Map 会有更好的性能。

## 并发写slice

一般不太会有并发写slice的，因为slice和map不同，对加入的先后顺序是敏感的，因此目前的实际应用场景没有使用到并发读写slice

## 并发写string

string是Go的内建类型，但对它的读写操作并非线程安全的，原因在于它的内部实际上是通过struct存储的，我们可以在runtime/string.go里面看到它的内部定义。

```golang
type stringStruct struct {
	str unsafe.Pointer
	len int
}

func stringStructOf(sp *string) *stringStruct {
	return (*stringStruct)(unsafe.Pointer(sp))
}
```

对于这样一个 struct ，go 无法保证原子性地完成赋值，因此可能会出现goroutine 1 刚修改完指针（str）、还没来得及修改长度（len），goroutine 2 就读取了这个string 的情况。

我们可以通过一个测试代码发现并发读写string的问题：

```golang
func TestString(t *testing.T) {
	s := "0"
	ch := make(chan string)
	go func() {
		i := 1
		for {
			if i%2 == 0 {
				s = "0"
			} else {
				s = "aa"
			}
			i++
			time.Sleep(1 * time.Microsecond)
		}
	}()

	go func() {
		for {
			b := s
			if b != "0" && b != "aa" {
				ch <- b
			}
		}
	}()

	for i := 0; i < 10; i++ {
		fmt.Println("Got strange string: ", <-ch)
	}
}
```

执行这个代码得到下面的输出结果：

```
=== RUN   TestString
Got strange string:  a
Got strange string:  a
Got strange string:  a
Got strange string:  01
Got strange string:  a
Got strange string:  a
Got strange string:  01
Got strange string:  a
Got strange string:  a
Got strange string:  a
--- PASS: TestString (0.01s)
```

通过`go tool compile -S`查看执行的汇编代码，可以发现，string的写入是分为写入长度和写入指针2个部分的。

![](https://suncle-public.oss-cn-shenzhen.aliyuncs.com/uPic/htZm5E-1629119549723.png)

因此在频繁的写入操作中，可能会出现写入了一部分数据就被读取出去了，自然就会读取到脏数据

仔细看上述示例代码，会发现在写入协程中有一个多余的sleep操作，如果把这个sleep去掉，运行的结果是永远读不到脏数据，这是为什么呢？原因在于编译器的优化。编译器优化之后会直接改写频繁赋值的逻辑，而不是持续写入长度和指针

![image-20210816212544508](https://suncle-public.oss-cn-shenzhen.aliyuncs.com/uPic/image-20210816212544508-1629120344614-1629120356605.png)

## 并发写interface

将上述并发写string代码中的类型改为interface就可以复现并发写interface的问题。



---

了解上面4中并发读写会造成panic或者脏读的情况之后，在后续的日常开发中，需要十分注意这样的情况

