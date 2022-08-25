---
categories:
  - python
date: '2017-05-22T00:52:13'
description: ''
tags:
  - web
  - 框架
  - 路由
  - Python
title: Python实现通用web框架
---




# 概述

Python WSGI规定了Web服务器和Python Web应用程序或Web框架之间的标准接口，主要是为了促进Web应用程序在各种Web服务器上的可移植性。在WSGI的规范下，各种各样的Web服务器和Web框架都可以很好的交互。由于WSGI的存在，用Python写一个简单的Web框架也变得容易了。

Web框架的核心部分是路由系统，客户端的请求Request发到wsgi服务器之后Web框架根据请求中的method和path进行多级路由后找到对应的handle方法，在调用方法处理Request得到Response。

本次主要依赖于两个库：

- WebOb库，用于封装wsgi的environ参数。
- re库，使用正则表达式匹配URL中的路径。

# 层次结构图

下面是客户端发送请求到WSGI服务器经过Web框架处理的整个流程的层次结构和数据流向图。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/Web%E6%A1%86%E6%9E%B6%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84%E5%9B%BE.jpg)

# Web框架实现

代码：[https://github.com/Flowsnow/web_framework](https://github.com/Flowsnow/web_framework)

参见每一次commit。

---

参考

- [Python-WSGI接口](https://suncle.me/2017/04/07/Python-WSGI%E6%8E%A5%E5%8F%A3/)


- [comyn-web](https://coding.net/u/comyn/p/web/git)


- [How to write a web framework in Python](http://anandology.com/blog/how-to-write-a-web-framework-in-python/)
- [用Python写一个简单的Web框架](https://www.cnblogs.com/russellluo/p/3338616.html)