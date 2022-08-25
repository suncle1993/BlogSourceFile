---
categories:
  - erlang
date: '2017-12-14T14:33:30'
description: ''
tags:
  - Erlang
  - 总结
  - 入门
title: Erlang入门路线
---



间歇性的学了一些Erlang，写了一个直播cdn网关的程序，也算是贡献了代码，完成了第一个项目。结束之际写一个入门路线，记录学习过程。

主要根据个人经验介绍最佳的学习路线，包括环境，Erlang语法，OTP和rebar构建调试打包过程等几个部分。

<!--more-->

# Erlang环境

主要是Erlang环境搭建和Erlang shell的使用

## 环境搭建

在[Erlang官网下载](https://www.erlang.org/downloads)Erlang源码包或者二进制包进行安装，因项目需要选择OTP18/erl7.3。

从源码安装Erlang/OTP的方法参见[github仓库](https://github.com/erlang/otp/blob/maint/HOWTO/INSTALL.md)

> windows安装完成之后需要配置环境变量，最终以在命令行输入erl能进入erlang shell为准

## Erlang shell

开始第一个程序：Erlang程序的编写，编译，执行。

输入下面的程序，把它存成一个叫做 hello.erl 的文件。

```erlang
-module(hello).
-compile(export_all).
start() ->
  "hello world".
```

启动 erlang shell，输入以下命令：

```erlang
root@ubuntu:/tmp# erl
Erlang/OTP 18 [erts-7.3] [source] [64-bit] [async-threads:10] [kernel-poll:false]

Eshell V7.3  (abort with ^G)
1> c(hello).
{ok,hello}
2> hello:start().
"hello world"
```

第一句是编译这个程序。第二句是执行命令，这就是所有要做的。

# 基本语法

## 学习语法

推荐阅读Erlang之父Joe Armstrong编写的**Erlang程序设计**这本书，网上有pdf版本。也可参见[Erlang学习笔记1](https://suncle.me/2017/08/11/Erlang%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0(1)/)。 

需要重点掌握Erlang内置数据结构如tuple，list，record，function，case/if等知识点的使用。

此外需要学习以下几项：

1. ets
2. dets
3. gen_tcp
4. database

## 练习题

学习基础语法之后需要做一些练习题，练习题来源有

1. **Erlang程序设计**书后练习题
2. [Erlang 官方练习题](https://www.erlang.org/course/exercises.html)
3. 常用oj上的简单算法题使用Erlang实现

后续更新一些习题（**挖坑**）

# OTP

什么是OTP？和Erlang的区别是什么？

> OTP即Open Telecom Platform（开放电信平台），不用理会OTP的名称，OTP的本质是一个应用程序操作系统，还包含大量库和程序用来构建大规模的分布式容错系统（这就是OTP的目的）。使用OTP写程序关键在于OTP中的行为（即behavior）。一个行为封装了某种常见的行为模糊。可以把这些行为理解为常见的编程套件，或者程序框架，只是使用这些框架方式是通过回调模块。
>
> 直接使用Erlang原语而不使用OTP编写Erlang程序是完全可行的，只是需要自己考虑容错、扩容和动态代码升级等等非功能性特性。也就是使用OTP编写Erlang程序，OTP的行为解决问题的非功能性部分，功能性的部分留给程序猿根据业务自己写回调模块来实现——因为对于所有的系统来说，非功能性的部分都是一样的。

需要掌握的OTP常见的行为有：

1. gen_server：服务器/客户端模型
2. supervisor：监控树
3. application：应用
4. gen_fsm：有限状态机
5. gen_event：事件处理器

掌握前三项就可以写普通的服务，包括tcp，http服务，amqp消息处理的服务等等。

# 集成开发环境

分为使用rebar从零构建Erlang项目和调试打包发布这2块。

## 使用rebar从零构建Erlang项目

开发环境推荐使用IDEA + Erlang + rebar:

- Erlang/OTP语言
- rebar工具构建Erlang项目
- IDE选择IDEA

OTP的application构建时需要遵循一定的约定来组织项目，具体的约定参考：[OTP应用设计原则](https://www.erlang.org/doc/design_principles/applications.html)。rebar遵循OTP设计原则，因此rebar构建的项目目录需要下列子文件夹：

```
─ ${application}
      ├── doc
      ├── include
      ├── priv
      ├── src
      │   └── ${application}.app.src
      └── test
```

rebar具体的构建方法参见：[**Rebar：Erlang构建工具**](https://www.cnblogs.com/panfeng412/archive/2011/08/14/compile-erlang-with-rebar.html)

构建完成之后，加上自己编写的模块，一个完整的Erlang示例项目目录结构如下（有省略）：

```
cdn_gateway
├── app.config
├── build
├── cdn_gateway.cmd
├── cdn_gateway.iml
├── deps
│   ├── amqp_client
│   ├── erldis
│   ├── goldrush
│   ├── lager
│   ├── meck
│   ├── rabbit_common
│   └── rfc4627_jsonrpc
├── ebin
├── include
├── Makefile
├── priv
├── rebar.config
├── rel
├── reltool.config
├── run.bat
├── src
│   ├── cdn_gateway_app.erl
│   ├── cdn_gateway.app.src
│   ├── cdn_gateway.erl
│   ├── cdn_gateway.hrl
│   ├── cdn_gateway_sup.erl
│   ├── cdn_gw_amqp_agent.erl
│   ├── cdn_gw_cluster.erl
│   ├── cdn_gw_http_acceptor.erl
│   ├── cdn_gw_live_operator.erl
│   ├── cdn_gw_live_plan_auth_mgr.erl
│   ├── cdn_gw_live_plan_online_mgr.erl
│   ├── cdn_gw_notice_http_handler.erl
│   └── cdn_gw_notice_http_handler_sup.erl
└── vm.args

211 directories, 1799 files

```

## 调试打包

调试脚本run.bat的功能主要是编译源代码，开启Erlang shell

```bash
call rebar compile skip_deps=true
cd ..
start werl -name cdn_gateway@127.0.0.1 -setcookie cdn_gateway -pa cdn_gateway\ebin cdn_gateway\deps\amqp_client\ebin cdn_gateway\deps\qk_ipdb\ebin cdn_gateway\deps\erldis\ebin cdn_gateway\deps\gen_server3\ebin cdn_gateway\deps\goldrush\ebin deps\mysql_agent\ebin cdn_gateway\deps\lager\ebin cdn_gateway\deps\mysql\ebin cdn_gateway\deps\pb\ebin cdn_gateway\deps\protobuffs\ebin cdn_gateway\deps\qukan_lib\ebin cdn_gateway\deps\rabbit_common\ebin cdn_gateway\deps\rfc4627_jsonrpc\ebin cdn_gateway\deps\qk_odbc_pool\ebin cdn_gateway\deps\qk_json\ebin -s cdn_gateway
cd cdn_gateway
```

打包的makefile文件如下：

```makefile
node_name := cdn_gateway

release:
	rm -rf deps
	rm -rf test/logs
	rm -rf build
	rm -f test/*.beam
	rebar clean
	rebar get-deps compile ct
	mkdir -p ./build/$(node_name);cp -rf ./ebin ./build/$(node_name)/;cp -rf ./priv ./build/$(node_name)/;cp -rf ./include ./build/$(node_name)/
	mkdir -p ./rel;cd ./rel;rm -rf *;rebar create-node nodeid=$(node_name);cp ../reltool.config ./;rebar generate;tar -cjf $(node_name).tar.bz2 $(node_name)
	mkdir -p ../build_linux
	mv rel/$(node_name).tar.bz2 ../build_linux
```

windows上调试，打包最好再对应的生产环境系统（比如ubuntu16.04）中打包，避免有坑。



















---

参考：

- [erlang工作前新手学习指引路线](https://www.yuanmas.com/info/nmaW66VBz3.html)
- [Erlang 中的并发 -- Actor 模型](https://blog.csdn.net/wwh578867817/article/details/49774169)
- [云栖社区-Erlang入门（二）—并发编程](https://yq.aliyun.com/articles/83126)
- [为什么我们放弃了Erlang技术栈](https://yq.aliyun.com/articles/229322?spm=5176.8067842.tagmain.121.dwjreE)

