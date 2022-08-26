---
abbrlink: 3953642204
alias: 2017/11/14/Python单元测试/index.html
categories:
- python
date: '2017-11-14T20:01:27'
description: ''
tags:
- 单元测试
- 测试
title: Python单元测试
---








# 单元测试

什么是单元测试, 维基百科上是这么定义的： unit testing is a method by which individual units of source code, sets of one or more computer program modules together with associated control data, usage procedures, and operating procedures, are tested to determine if they are fit for use.[1] Intuitively, one can view a unit as the smallest testable part of an application. 简而言之，就是验证系统中最小可测试单元的功能是否正确的自动化测试。因此，单元测试的目地就是“对被测试对象的职责进行验证”, 在写单元测试之前，先识别出被测试对象的职责，就知道该怎么写这个单元测试了。

根据被测试对象，单元测试可以分为两大类：

- 对不依赖于外部资源的组件的单元测试：使用unittest基本功能即可
- 对依赖于外部资源的组件的单元测试：需要使用mock

<!--more-->

# unittest使用

python单元测试库unittest的基本使用参见[廖雪峰Python单元测试](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/00143191629979802b566644aa84656b50cd484ec4a7838000)

具体使用参考以下资料

1. [Python中的单元测试](https://pm.readthedocs.io/unittest/python.html)
2. [ningning.today-flask项目单元测试实践](http://ningning.today/2016/11/22/python/flask-unittest/)
3. [Python unittest官方文档](https://docs.python.org/2/library/unittest.html)
4. [极客学院-单元测试](https://wiki.jikexueyuan.com/project/explore-python/Testing/README.html)
5. [nicholas-怎样写单元测试](http://nicholas.ren/2012/10/21/my-understanding-on-unit-testing.html)

# mock

为什么要用mock？

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/blog/%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95mock%E5%A4%A7%E7%89%9B%E5%9B%9E%E5%A4%8D.png)

看了很多篇mock的讲解，写的最好的一篇是[[Naftuli Kay-An Introduction to Mocking in Python](https://www.toptal.com/python/an-introduction-to-mocking-in-python)，以删除文件为例组成深入讲解mock的使用。其他资料可以参见：

1. [Python单元测试和Mock测试](http://andrewliu.in/2015/12/12/Python%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95%E5%92%8CMock%E6%B5%8B%E8%AF%95/)
2. [mock-autospec](http://www.voidspace.org.uk/python/mock/helpers.html#autospeccing)

仿照这篇文章改写qk_log日志模块，qk_log.py代码如下

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import os
import sys
import datetime
import logging
import logging.handlers

_console_logger = None
_warn_logger = None
_error_logger = None

CONSOLE_FILENAME = 'log/console.log'
WARNING_FILENAME = 'log/warn.log'
ERROR_FILENAME = 'log/error.log'


def log_init():
    if os.path.exists('log/') is True:
        pass
    else:
        os.mkdir('log/')
    global _console_logger, _warn_logger, _error_logger
    handler = logging.handlers.RotatingFileHandler(
        CONSOLE_FILENAME, maxBytes=20*1024*1024, backupCount=5)
    hdr = logging.StreamHandler()
    _console_logger = logging.getLogger('debug')
    _console_logger.addHandler(handler)
    _console_logger.addHandler(hdr)
    _console_logger.setLevel(logging.DEBUG)

    handler = logging.handlers.RotatingFileHandler(
        WARNING_FILENAME, maxBytes=20*1024*1024, backupCount=5)
    hdr = logging.StreamHandler()
    _warn_logger = logging.getLogger('warn')
    _warn_logger.addHandler(handler)
    _warn_logger.addHandler(hdr)
    _warn_logger.setLevel(logging.WARN)

    handler = logging.handlers.RotatingFileHandler(
        ERROR_FILENAME, maxBytes=20*1024*1024, backupCount=5)
    hdr = logging.StreamHandler()
    _error_logger = logging.getLogger('error')
    _error_logger.addHandler(handler)
    _error_logger.addHandler(hdr)
    _error_logger.setLevel(logging.ERROR)


def dlog(msg):
    file_name, file_no, unused = find_caler()
    time_str = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    _console_logger.debug('[%s] [%s] [%s,%d] %s' % (time_str, 'debug', file_name, file_no, msg))


def ilog(msg):
    file_name, file_no, unused = find_caler()
    time_str = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    _console_logger.info('[%s] [%s] [%s,%d] %s' % (time_str, 'info', file_name, file_no, msg))


def wlog(msg):
    file_name, file_no, unused = find_caler()
    time_str = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    _console_logger.warn('[%s] [%s] [%s,%d] %s' % (time_str, 'warning', file_name, file_no, msg))
    _warn_logger.warn('[%s] [%s] [%s,%d] %s' % (time_str, 'warning', file_name, file_no, msg))


def elog(msg):
    file_name, file_no, unused = find_caler()
    time_str = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    _console_logger.error('[%s] [%s] [%s,%d] %s' % (time_str, 'error', file_name, file_no, msg))
    _error_logger.error('[%s] [%s] [%s,%d] %s' % (time_str, 'error', file_name, file_no, msg))


def find_caler():
    f = sys._getframe(2)
    co = f.f_code
    return (os.path.basename(co.co_filename), f.f_lineno, co.co_name) if co != None else ('unknown', 0, 'unknown')


if __name__ == '__main__':
    log_init()
    dlog('test.log %d'%(123))
    ilog('test.log %d' % (123))
    wlog('test.log %d' % (123))
    elog('test.log %d' % (123))

```

单元测试代码如下：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
Created on 10/10/17 11:27 AM
@author: Chen Liang
@function: 日志模块  单元测试
"""

import sys

reload(sys)
sys.setdefaultencoding('utf-8')
import unittest
import mock
import datetime
from qk_log import log_init, dlog, ilog, wlog, elog


class TestQkLog(unittest.TestCase):
    dt_str = datetime.datetime.strptime('2017-10-11 11:08:59', '%Y-%m-%d %H:%M:%S')

    @mock.patch('qk_log.os.path')
    @mock.patch('qk_log.datetime.datetime')
    @mock.patch('qk_log.logging')
    @mock.patch('qk_log.find_caler')
    def test_dlog(self, mock_caler, mock_logging, mock_datetime, mock_path):
        mock_path.exists.return_value = True
        log_init()
        self.assertFalse(mock_logging.getLogger('debug').debug.called, "Failed to not write log.")

        mock_caler.return_value = ('qk_log_test', 12, '')
        mock_datetime.now.return_value = self.dt_str
        dlog('any msg')
        mock_logging.getLogger('debug').debug.assert_called_with(
            '[%s] [%s] [%s,%d] %s' % ('2017-10-11 11:08:59', 'debug', 'qk_log_test', 12, 'any msg'))

    @mock.patch('qk_log.os.path')
    @mock.patch('qk_log.datetime.datetime')
    @mock.patch('qk_log.logging')
    @mock.patch('qk_log.find_caler')
    def test_ilog(self, mock_caler, mock_logging, mock_datetime, mock_path):
        mock_path.exists.return_value = True
        log_init()
        self.assertFalse(mock_logging.getLogger('debug').info.called, "Failed to not write log.")
        mock_caler.return_value = ('qk_log_test', 12, '')
        mock_datetime.now.return_value = self.dt_str
        ilog('any msg')
        mock_logging.getLogger('debug').info.assert_called_with(
            '[%s] [%s] [%s,%d] %s' % ('2017-10-11 11:08:59', 'info', 'qk_log_test', 12, 'any msg'))

    @mock.patch('qk_log.os.path')
    @mock.patch('qk_log.datetime.datetime')
    @mock.patch('qk_log.logging')
    @mock.patch('qk_log.find_caler')
    def test_wlog(self, mock_caler, mock_logging, mock_datetime, mock_path):
        mock_path.exists.return_value = True
        log_init()
        self.assertFalse(mock_logging.getLogger('warn').info.called, "Failed to not write log.")
        mock_caler.return_value = ('qk_log_test', 12, '')
        mock_datetime.now.return_value = self.dt_str
        wlog('any msg')
        mock_logging.getLogger('warn').warn.assert_called_with(
            '[%s] [%s] [%s,%d] %s' % ('2017-10-11 11:08:59', 'warning', 'qk_log_test', 12, 'any msg'))

    @mock.patch('qk_log.os.path')
    @mock.patch('qk_log.datetime.datetime')
    @mock.patch('qk_log.logging')
    @mock.patch('qk_log.find_caler')
    def test_elog(self, mock_caler, mock_logging, mock_datetime, mock_path):
        mock_path.exists.return_value = True
        log_init()
        self.assertFalse(mock_logging.getLogger('error').info.called, "Failed to not write log.")
        mock_caler.return_value = ('qk_log_test', 12, '')
        mock_datetime.now.return_value = self.dt_str
        elog('any msg')
        mock_logging.getLogger('error').error.assert_called_with(
            '[%s] [%s] [%s,%d] %s' % ('2017-10-11 11:08:59', 'error', 'qk_log_test', 12, 'any msg'))

if __name__ == '__main__':
    unittest.main()

```

# 对单元测试的看法

在一次整体改造Python数据统计分析项目时打算引进单元测试，在写完公共库的单元测试之后发现花费在单元测试上的时间较多，而且公共库不常改动，业务逻辑有比较混乱，因此团队决定放弃单元测试。对于以快速上线的初创公司和初创团队的项目来说，可以不用急着写单元测试，因为在一切改动都可能发生的情况下，再代码丢弃的时候对应的单元测试也就被丢弃了，浪费了过多的人力。

因此，初创团队不建议写单元测试，做好程序埋点和监控报警即可。