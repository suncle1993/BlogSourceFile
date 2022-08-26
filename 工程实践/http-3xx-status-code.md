---
abbrlink: 3040323093
alias: 2019/11/13/http-3xx-status-code/index.html
categories:
- 工程实践
date: '2019-11-13T20:14:44'
description: ''
tags:
- http
- 状态码
- 重定向
title: 一次奇怪的http状态码改变
---









最近将一个很久没有更新的部署在物理机上的一个老服务迁移到k8s时，发现在gitlab ci跑pytest的过程中出现以下报错：

```
___________________________ HelloTests.test_redirect ___________________________

self = <tests.test_hello.HelloTests testMethod=test_redirect>

    def test_redirect(self):
        resp = self.get('/api/hello')
>       self.assertEqual(http.HTTPStatus.MOVED_PERMANENTLY.value, resp.status_code)
E       AssertionError: 301 != 308

tests/test_hello.py:18: AssertionError
```

按照python系的习惯，一般而言，我们习惯在接口最后加上一个slash，因此会将所有不带slash的接口301重定向到带slash。但是这一次提示重定向的状态码是308。

<!--more-->

### 问题排查

经排查，发现是使用的WSGI服务器WerkZeug的版本发生了升级由`Werkzeug-0.12.0`升级到`Werkzeug-0.16.0`，通过查看`Werkzeug-0.12.0`的重定向确实是301，源代码如下：

```python
class RequestRedirect(HTTPException, RoutingException):

    """Raise if the map requests a redirect. This is for example the case if
    `strict_slashes` are activated and an url that requires a trailing slash.

    The attribute `new_url` contains the absolute destination url.
    """
    code = 301

    def __init__(self, new_url):
        RoutingException.__init__(self, new_url)
        self.new_url = new_url

    def get_response(self, environ):
        return redirect(self.new_url, self.code)
```

而`Werkzeug-0.16.0`的重定向却变成了308，源代码如下：

```python
class RequestRedirect(HTTPException, RoutingException):
    """Raise if the map requests a redirect. This is for example the case if
    `strict_slashes` are activated and an url that requires a trailing slash.

    The attribute `new_url` contains the absolute destination url.
    """

    code = 308

    def __init__(self, new_url):
        RoutingException.__init__(self, new_url)
        self.new_url = new_url

    def get_response(self, environ):
        return redirect(self.new_url, self.code)
```

查阅Werkzeug代码的更新历史，在[https://github.com/pallets/werkzeug/pull/1342/](https://github.com/pallets/werkzeug/pull/1342/)这个pr中将301改成了308。pr中的conversation的讨论主要是有以下观点：

- 根据MDN，除Windows <= 8上的IE之外，所有浏览器均支持308。Windows 8也快完蛋了。 如果仍然有人需要支持非常老的浏览器，则可以修改RequestRedirect.code = 301支持301重定向
- 由/a重定向到/a/并不会改变method，301和308对于多数人没有什么影响

相关pr：https://github.com/pallets/werkzeug/pull/1402/files

### http 3xx 介绍

介绍3xx之前，再大致过一遍目前的状态码复习一遍

- 1xx：临时的响应，旨在在服务器继续处理请求时使用。很少被用到
- 2xx：成功-事情按预期工作时使用的状态码。 根据请求返回不同的成功代码
- 3xx：重定向—用于告诉客户端在其他地方查找所请求资源的状态
- 4xx：客户端错误-这些状态码告诉客户端它做错了什么
- 5xx：服务端错误-服务器上某些东西无法正常工作时的状态码

关于3xx的http状态码，简单做一下介绍：

#### HTTP 301 Moved Permanently

永久重定向：被请求的资源已永久移动到新位置，并且将来任何对此资源的引用都应该使用本响应返回的若干个 URI 之一。如果可能，拥有链接编辑功能的客户端应当自动把请求的地址修改为从服务器反馈回来的地址。除非额外指定，否则这个响应也是可缓存的。

#### HTTP 302 Found

临时重定向：请求的资源现在临时从不同的 URI 响应请求。由于这样的重定向是临时的，客户端应当继续向原有地址发送以后的请求。只有在Cache-Control或Expires中进行了指定的情况下，这个响应才是可缓存的。

#### HTTP 303 See Other

对应当前请求的响应可以在另一个 URI 上被找到，而且客户端应当采用 GET 的方式访问那个资源。这个方法的存在主要是为了允许由脚本激活的POST请求输出重定向到一个新的资源。

#### HTTP 304 Not Modified

如果客户端发送了一个带条件的 GET 请求且该请求已被允许，而文档的内容（自上次访问以来或者根据请求的条件）并没有改变，则服务器应当返回这个状态码。304 响应禁止包含消息体，因此始终以消息头后的第一个空行结尾。

#### HTTP 305 Use Proxy

被请求的资源必须通过指定的代理才能被访问。Location 域中将给出指定的代理所在的 URI 信息，接收者需要重复发送一个单独的请求，通过这个代理才能访问相应资源。只有原始服务器才能建立305响应。

#### HTTP 306 Switch Proxy

这个状态最初是指后续请求应使用指定的代理。但是已经被弃用了

#### HTTP 307 Temporary Redirect

临时重定向且不能修改之后的请求方法，302的扩充

#### HTTP 308 Permanent Redirect

永久重定向且不能修改之后的请求方法，301的扩充

