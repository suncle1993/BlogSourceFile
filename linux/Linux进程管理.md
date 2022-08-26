---
abbrlink: 973368527
alias: 2016/01/22/Linux进程管理/index.html
categories:
- linux
date: '2016-01-22T19:59:28'
description: ''
tags:
- Linux
- CentOS
- 进程管理
title: Linux进程管理
---








## 进程的概念

### Linux系统中进程的类型

分为三种不同的类型，分别是:

- 交互进程：由一个启动的进程，交互进程既可以在前台运行，也可以后台运行。


- 批处理进程：不与特定的终端相关联，提交到等待队列中顺序执行的进程。


- 守护进程：在Linux在启动时初始化，需要时运行于后台的进程。

### 进程的启动方式

手工启动：1、前台启动  2、后台启动

调度启动：事先进行设置，根据用户要求自行启动

<!--more-->

## 查看系统中的进程

### ps命令：Process Status

ps命令的使用参见[ps命令](https://man.linuxde.net/ps)和[每天一个linux命令：ps命令](https://www.cnblogs.com/peida/archive/2012/12/19/2824418.html)

ps命令用于报告当前系统的进程状态。可以搭配kill指令随时中断、删除不必要的程序。使用该命令可以确定有哪些进程正在运行和运行的状态、进程是否结束、进程有没有僵死、哪些进程占用了过多的资源等等。





### top命令 - display Linux tasks

top命令的使用参见[top命令](https://man.linuxde.net/top)和[每天一个linux命令：top命令](https://www.cnblogs.com/peida/archive/2012/12/24/2831353.html)

top命令可以实时动态地查看系统的整体运行情况，是一个综合了多方信息监测系统性能和运行信息的实用工具，类似于Windows的任务管理

#### 命令格式

> top [参数]

#### 命令功能

显示当前系统正在执行的进程的相关信息，包括进程ID、内存占用率、CPU占用率等

#### 命令参数

``` 

```

#### 使用实例





## 控制系统中的进程



## 了解守护进程