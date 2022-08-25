---
categories:
  - python
date: '2017-04-07T21:41:08'
description: ''
tags:
  - web
  - wsgi
title: Python-WSGI接口
---



Python WSGI规定了Web服务器和Python Web应用程序或Web框架之间的标准接口，主要是为了促进Web应用程序在各种Web服务器上的可移植性。

上述这句话翻译自Python官方的PEP333标准：[PEP 333 -- Python Web Server Gateway Interface v1.0](https://www.python.org/dev/peps/pep-0333/)

# WSGI接口概述

WSGI的含义：Web Server Gateway Interface（Web服务器网管接口）。

WSGI接口包含两方面：server/gateway端 及 application/framework端。后面直接使用server和application来说明，不再使用gateway和framework。server端直接调用application端提供的**可调用对象**。另外在server和application之间还可以有一种称作middleware的中间件。中间件对于server来说就是一个application，但是对于application来说中间件却是一个server。

上述可调用对象是指：函数、方法、类或者带有`__call__`方法的实例。

以下分别介绍application端，Server端和middleware三个部分

<!--more-->

# Application端

函数、方法、类及带有callable方法的实例等可调用对象都可以作为application对象。application对象接受两个参数并且可以被多次调用。

**参数**

- environ：environ参数是一个字典对象，该对象必须是内置的Python字典，应用程序可以任意修改该字典。字典还必须包含某些WSGI必需的变量。
- start_response：由server提供的回调函数，其作用是由application将状态码和响应头返回给server。这个函数有两个必需的位置参数和一个可选参数，三个参数分别为status，response_headers和exc_info

start_response的三个参数的意义如下：

- status：HTTP 响应码及消息，例如status = '200 OK'
- response_headers：提供给客户端的响应头，需要封装成list of tuple pairs 的形式

```python
response_headers = [('Content-Type', 'text/plain'), ('Content-Length', str(len(response_body)))]
```

- exc_info：Python sys.exc_info()元组

**返回值**

application对象必须返回一个响应体，响应体的形式是list of str，也就是说返回值是由一个或多个字符串组成的列表。

以下是一个函数作为application对象的例子

```python
def simple_app(environ, start_response):
    """最简单的application对象"""
    status = '200 OK'
    response_headers = [('Content-type', 'text/plain')]
    start_response(status, response_headers)
    return ['Hello world!\n']
```

以下是一个类作为application对象的例子

```python
class AppClass:
    """
    AppClass()会返回一个AppClass类对象作为application，然后在迭代的时候就会调用__iter__方法，然后就可以产生相同的输出。
    如果我们也可以实现__call__方法直接将实例当做application
    """

    def __init__(self, environ, start_response):
        self.environ = environ
        self.start = start_response

    def __iter__(self):
        status = '200 OK'
        response_headers = [('Content-type', 'text/plain')]
        self.start(status, response_headers)
        yield "Hello world!\n"
```

# Server端

WSGI server必须要调用application，而且要使用位置参数的形式调用application。同时，从application的协议要求可知：

- WSGI server必须向application提供环境参数，因此，自身也必须能够获取环境参数。


- WSGI server接收application的返回值作为响应体。

最简单的WSGI server为Python自带的wsgiref.simple_server。

**代码**

```python
from wsgiref.simple_server import make_server
server = make_server('localhost', 8080, application)
server.serve_forever()
```

# Middleware

中间件位于WSGI server和WSGI application之间，关于中间件的部分代码参考：

- [An Introduction to the Python Web Server Gateway Interface (WSGI)](http://ivory.idyll.org/articles/wsgi-intro/what-is-wsgi.html)

代码如下

```python
from wsgiref.simple_server import make_server

def application(environ, start_response):

    response_body = 'hello world!'

    status = '200 OK'

    response_headers = [
        ('Content-Type', 'text/plain'),
        ('Content-Length', str(len(response_body)))
    ]

    start_response(status, response_headers)
    return [response_body]

# 中间件
class Upperware:
   def __init__(self, app):
      self.wrapped_app = app

   def __call__(self, environ, start_response):
      for data in self.wrapped_app(environ, start_response):
        yield data.upper()

wrapped_app = Upperware(application)

httpd = make_server('localhost', 8051, wrapped_app)

httpd.serve_forever()

print 'end'
```