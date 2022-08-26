---
abbrlink: 3013774428
alias: 2018/01/29/Erlang-test-knowledge-collection/index.html
categories:
- erlang
date: '2018-01-29T18:31:37'
description: ''
tags:
- Erlang
- 测试
- EUnit
- Common Test
title: Erlang测试全集(挖坑)
---









# Erlang测试全集(挖坑)

本次只是简要列举Erlang测试相关的框架和概念，Erlang测试的详细使用在实际使用时再进行补充（**挖坑**），目前所有的Erlang程序中除部分公共基础app需要写单元测试和覆盖率报告之外其他都不需要写。

后续会按照以下四个部分介绍Erlang测试相关知识

- EUnit
- Common Test
- Cover
- Quality Control

<!--more-->

# EUnit

白盒测试，使用EUnit框架，主要参考以下资料：

1. [rebar的开始wiki-使用EUnit为例讲解](https://github.com/rebar/rebar/wiki/Getting-started)
2. [博客园-Rebar：Erlang构建工具](https://www.cnblogs.com/panfeng412/archive/2011/08/14/compile-erlang-with-rebar.html)

# Common Test

Common Test简写为CT

黑盒测试，使用CT框架，趣看公司所有的单元测试都是基于CT，即只写黑盒

1. [CT入门示例-Erlang Common Test Examples](https://github.com/Eonblast/Trinity)
2. [淘宝储霸-rebar和common_test使用实践和疑惑澄清](http://blog.yufeng.info/archives/1711)
3. [**淘宝储霸-Erlang开发实践-可重点阅读**](https://www.google.com/url?q=http://blog.yufeng.info/wp-content/uploads/2009/11/Erlang%25E5%25BC%2580%25E5%258F%2591%25E5%25AE%259E%25E8%25B7%25B5.pptx&sa=U&ved=0ahUKEwjBzo-OxvLYAhVXHGMKHZuNAzQQFggTMAU&client=internal-uds-cse&cx=011994216034381653247:b54vq7hrjse&usg=AOvVaw0WIFSQAi_m-43HdKJGEZcl)
4. [Erlang官方-Running Tests and Analyzing Results](https://www.erlang.org/doc/apps/common_test/run_test_chapter.html)
5. [EUC2009-CommonTestPresentation.pdf](http://www.erlang-factory.com/upload/presentations/204/EUC2009-CommonTestPresentation.pdf)

# Cover

代码测试覆盖率，代码覆盖率的意义在于：

1. 分析未覆盖部分的代码，从而反推在前期测试设计是否充分，没有覆盖到的代码是否是测试设计的盲点，为什么没有考虑到？需求/设计不够清晰，测试设计的理解有误，工程方法应用后的造成的策略性放弃等等，之后进行补充测试用例设计。
2. 检测出程序中的废代码，可以逆向反推在代码设计中思维混乱点，提醒设计/开发人员理清代码逻辑关系，提升代码质量。
3. 代码覆盖率高不能说明代码质量高，但是反过来看，代码覆盖率低，代码质量不会高到哪里去，可以作为测试自我审视的重要工具之一。

EUnit和Common Test都可以产生coverage report，参考：

1. [rebar的开始wiki-使用EUnit为例讲解](https://github.com/rebar/rebar/wiki/Getting-started)
2. [Stack Overflow-Cover report from Common Test when using rebar](https://stackoverflow.com/questions/28969405/cover-report-from-common-test-when-using-rebar)

# Quality Control

QC，即质量控制，主要是压力测试，可以使用Tsung压力测试工具（由Erlang编写）

- api
- mysql
- console


# Erlang的优点

偶然间看到淘宝储霸关于Erlang的优点的阐述，觉得很精辟，所以写在最后

Erlang的优点（为什么选择使用Erlang实现）

- 高并发、高性能、集群易扩展
- 时间检验的高可靠
- 强大的管理功能，方便的问题定位支持
- 强大的交互性，与其他系统整合能力
- Erlang独特的世界观
  - 世界是并行的
  - 万物皆独善其身
  - 万物皆通讯
  - 天有不测风云