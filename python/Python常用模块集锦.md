---
abbrlink: 2568280681
alias: 2018/03/12/Python常用模块集锦/index.html
categories:
- python
date: '2018-03-12T17:51:35'
description: ''
tags:
- Python
- 总结
title: Python常用模块集锦
---









Python常用模块集锦

常用模块主要分为以下几类（缺失的后续再补充）：

- 时间转换
- 时间计算
- 序列化和反序列化：`json`，`pickle`
- 编解码：`unicode`，`base64`
- 加解密：`md5`，`sha1`，`hmac_sha1`，`aes`
- 常见装饰器：
  - 计算执行时间装饰器
  - 缓存装饰器
  - 错误重试装饰器
  - 延迟装饰器
  - 尾递归优化装饰器
- `ini`配置文件读取

<!--more-->

代码整合如下：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
Created on 9/21/17 1:46 PM
@author: Chen Liang
@function: python常用模块集锦，util.py
"""

import time
import datetime
import ConfigParser
import ast
import sys
import json
import pickle
import base64
import hashlib
from Crypto.Cipher import AES
from binascii import b2a_hex, a2b_hex
from functools import wraps


BEFORE = 1
LATER = 2


class CommonUtil(object):
    """Python通用单元：不好归类但常用的方法此处添加"""
    pass


class TimeTransferUtil(object):
    """时间相关的常见转换方法"""

    
class TimeUtil(object):
    """时间相关的常见计算方法"""
    @staticmethod
    def str_to_date():
        pass


class SerializeUtil(object):
    """序列化和反序列化：json, pickle"""
    @staticmethod
    def json_loads(json_str, encoding=None):
        try:
            obj = json.loads(s=json_str, encoding=encoding)
            return True, obj
        except ValueError as e:
            return False, str(e)
        except Exception as e:
            return False, str(e)

    @staticmethod
    def json_dumps(obj):
        try:
            json_str = json.dumps(obj=obj)
            return True, json_str
        except TypeError as e:
            return False, str(e)
        except Exception as e:
            return False, str(e)

    @staticmethod
    def pickle_loads(pickle_str):
        try:
            obj = pickle.loads(pickle_str)
            return True, obj
        except IndexError as e:
            return False, str(e)
        except Exception as e:
            return False, str(e)

    @staticmethod
    def pickle_dumps(obj):
        try:
            pickle_str = pickle.dumps(obj)
            return True, pickle_str
        except Exception as e:
            return False, str(e)


class CodecUtil(object):
    """编解码相关常见方法：base64 unicode"""
    @staticmethod
    def base64_encode(data):
        try:
            return True, base64.b64encode(data)
        except TypeError as e:
            return False, str(e)
        except Exception as e:
            return False, str(e)

    @staticmethod
    def base64_decode(encoded_data):
        try:
            return True, base64.b64decode(encoded_data)
        except TypeError as e:
            return False, str(e)
        except Exception as e:
            return False, str(e)

    @staticmethod
    def to_unicode(s, encoding='utf-8'):
        return s if isinstance(s, unicode) else unicode(s, encoding)

    @staticmethod
    def unicode_to(unicode_s, encoding='utf-8'):
        return unicode_s.encode(encoding)


class CryptoUtil(object):
    """加解密相关常见方法： md5 aes"""
    @staticmethod
    def md5(str_object):
        """md5"""
        m = hashlib.md5()
        m.update(str_object)
        return m.hexdigest()

    @staticmethod
    def aes_encrypt(s, key, salt, mode=AES.MODE_CBC):
        """
        aes加密
        :param s: 待加密字符串
        :param key: 密钥
        :param salt: 盐, 16bit eg. b'0000000101000000'
        :param mode: AES模式
        :return: 加密后的字符串
        """
        cipher = AES.new(hashlib.md5(key).hexdigest(), mode, salt)
        n_text = s + ('\0' * (16 - (len(s) % 16)))
        return b2a_hex(cipher.encrypt(n_text))

    @staticmethod
    def aes_decrypt(s, key, salt, mode=AES.MODE_CBC):
        """
        aes解密
        :param s: 待解密字符串
        :param key: 密钥
        :param salt: 盐, 16bit eg. b'0000000101000000'
        :param mode: AES模式
        :return: 解密后的字符串
        """
        cipher = AES.new(hashlib.md5(key).hexdigest(), mode, salt)
        return cipher.decrypt(a2b_hex(s)).rstrip('\0')


class TailRecurseException:
    """尾递归异常"""
    def __init__(self, args, kwargs):
        self.args = args
        self.kwargs = kwargs


class DecoratorUtil(object):
    """常见装饰器： 执行时间timeit，缓存cache，错误重试retry"""

    __cache_dict = {}

    @staticmethod
    def timeit(fn):
        """计算执行时间"""
        @wraps(fn)
        def wrap(*args, **kwargs):
            start = time.time()
            ret = fn(*args, **kwargs)
            end = time.time()
            print "@timeit: {0} tasks, {1} secs".format(fn.__name__, str(end - start))
            return ret
        return wrap

    @staticmethod
    def __is_expired(entry, duration):
        """是否过期"""
        if duration == -1:
            return False
        return time.time() - entry['time'] > duration

    @staticmethod
    def __compute_key(fn, args, kw):
        """序列化并求其哈希值"""
        key = pickle.dumps((fn.__name__, args, kw))
        return hashlib.sha1(key).hexdigest()

    @classmethod
    def cache(cls, expired_time=-1):
        """
        缓存
        :param expired_time: 过期时间，-1 表示不过期
        :return: 返回缓存的结果或者计算的结果
        """
        def _cache(fn):
            @wraps(fn)
            def wrap(*args, **kwargs):
                key = cls.__compute_key(fn, args, kwargs)
                if key in cls.__cache_dict:
                    if cls.__is_expired(cls.__cache_dict[key], expired_time) is False:
                        return cls.__cache_dict[key]['value']
                ret = fn(*args, **kwargs)
                cls.__cache_dict[key] = {
                    'value': ret,
                    'time': time.time()
                }
                return ret
            return wrap
        return _cache

    @staticmethod
    def retry(exceptions, retry_times=3, time_pause=3, time_offset=1):
        """
        错误重试
        :param exceptions: 单个异常比如ValueError, 或者tuple,元组元素是异常，比如(ValueError, TypeError)
        :param retry_times: 重试次数
        :param time_pause: 初始暂停时间
        :param time_offset: 暂停时间的偏移倍数，默认不偏移
        :return: 返回成功的值，或者重拾次数结束时抛出异常
        """
        def _retry(fn):
            @wraps(fn)
            def wrap(*args, **kwargs):
                retry_times_tmp, time_pause_tmp = retry_times, time_pause
                while retry_times_tmp > 1:
                    try:
                        return fn(*args, **kwargs)
                    except exceptions:
                        time.sleep(time_pause_tmp)
                        retry_times_tmp -= 1
                        time_pause_tmp *= time_offset
                return fn(*args, **kwargs)
            return wrap
        return _retry

    @staticmethod
    def delay(delay_time=3, mode=BEFORE):
        """
        延迟装饰器，支持在函数执行之前和之后加延时，如果想在前后同时加，可以使用两次装饰。
        time.sleep只会阻塞当前线程不会阻塞整个进程，其它线程不受影响
        :param delay_time: 延迟时间，是float类型
        :param mode: 模式，指定是在函数执行之前加延时还是在执行之后加，值为BEFORE(1)或者LATER(2)
        :return:
        """
        def _delay(fn):
            @wraps(fn)
            def wrap(*args, **kwargs):
                if mode == BEFORE:
                    time.sleep(delay_time)
                ret = fn(*args, **kwargs)
                if mode == LATER:
                    time.sleep(delay_time)
                return ret
            return wrap
        return _delay

    @staticmethod
    def tail_call_optimized(fn):
        """尾递归优化装饰器，如果被装饰函数不是尾递归函数则会报错"""
        @wraps(fn)
        def wrap(*args, **kwargs):
            f = sys._getframe()
            if f.f_back and f.f_back.f_back and f.f_back.f_back.f_code == f.f_code:
                raise TailRecurseException(args, kwargs)
            else:
                while True:
                    try:
                        return fn(*args, **kwargs)
                    except TailRecurseException as e:
                        args = e.args
                        kwargs = e.kwargs
        return wrap


class IniConfigParserUtil(object):
    """ini配置文件读取"""
    def __init__(self, *file_names):
        """
        init
        :param file_names: 包含多个元素的可迭代对象
        """
        self.config = ConfigParser.ConfigParser()
        for file_name in file_names:
            try:
                self.config.readfp(open(file_name, 'rb'))
                break
            except IOError:
                continue
        else:
            sys.exit('All files have failed to read')

    def get_string(self, section, option):
        return self.config.get(section, option)

    def get_int(self, section, option):
        return self.config.getint(section, option)

    def get_float(self, section, option):
        return self.config.getfloat(section, option)

    def get_boolean(self, section, option):
        return self.config.getboolean(section, option)

    def get_list(self, section, option):
        return ast.literal_eval(self.config.get(section, option))

    def get_dict(self, section, option):
        return ast.literal_eval(self.config.get(section, option))
```

缺失部分后续待添加，记得填坑。