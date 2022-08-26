---
abbrlink: 858751997
alias: 2016/12/04/Python答疑解惑/index.html
categories:
- python
date: '2016-12-04T22:47:22'
description: ''
tags:
- Python
- 经验
title: Python答疑解惑
---








# Python自动化运维答疑解惑

以下为Python入门的几个常见疑惑，现在统一在下面列出。

**1、如果使用Python3.5.2，但是一般公司的生产环境上都是linux默认的Python，一般是2.6.6，而且没有权限更改，这种情况下我们有什么好的办法吗？**

>  python2一般都有的第三方库，在Python3中都会有的，而且Python2在2020年就彻底停止支持了，所以没有特殊情况，直接选择python最新版本即可。除非你的项目必须依赖一个python2的第三方库且这个库在python3中并没有。

**2、Python在现在以及未来主要应用的方向是什么？**

> Python的应用的主要方向就是两类，一类是爬虫方向，一类是自动化方向。至于自动化方向，分为自动化运维和自动化测试。Python做web开发在未来几年都不会是主流，web开发的主流还是会Java这种工业语言。
<!--more-->

**3、自动化运维方面主要项目是哪些？**

> CMDB(Configuration Management Database)->远程执行和调度系统->自动化流程平台。其中CMDB主要是存储与管理企业IT架构中设备的各种配置信息，远程执行和调度系统主要是负责对各个设备进行调度并执行相关命令，所以难点在于调度。自动化流程平台主要是定义日常操作的具体流程控制。做好这三点之后就有了一个基本的运维自动化管理平台，然后再集成自动化监控平台和运维安全方面的认证堡垒机，即可形成一个比较完善的运维自动化管理平台。

**4、运维开发日常工作是什么？**

> 写代码，包括CMDB，任务调度、流程系统、DB管理、日志分析

**5、Python在网上的练习题较少，那平时需要去哪儿练手，才能达到一天一百行代码？**

> 网上有不少的在线online judge支持Python语言，比如：
>
> 高中生NOIP常用的tyvj：[http://www.tyvj.cn/Problem](http://www.tyvj.cn/Problem)，是中文网站
>
> 平时经常听说的LeetCode OJ：[https://leetcode.com/problemset/algorithms](https://leetcode.com/problemset/algorithms)
>
> 浙江大学的在线OJ： [http://acm.zju.edu.cn](http://acm.zju.edu.cn/) 超过2000题，支持/Python在内的主流语言
>
> 国外的Sphere online judge：[http://www.spoj.com/problems/classical](http://www.spoj.com/problems/classical)，几乎什么语言都支持
>
> 再有来源就是书籍里面的题目了，推荐：cookbook和a byte of python

**6、Python做GUI应用程序的时候用什么比较好？**

> 推荐使用Pyqt，使用过qt开发C++GUI程序的人都知道。

**7、Python的版本有很多，做不同的项目都需要不同的版本，需要准备多套环境，如何做版本管理？**

> 可以使用pyenv管理Python环境。版本再多都没事，而且永远都不要动系统自带的Python环境，有很多程序依赖于系统自带Python

**8、Python面试有什么技巧？**

面试还是得看基础，看编程功底。比如以下几个面试题就很考验功底：

- 实现一个栈


- 栈的应用，括号匹配的检测


- 四则运算的解析：优先级，括号，中括号，大括号


- 写一个正则引擎