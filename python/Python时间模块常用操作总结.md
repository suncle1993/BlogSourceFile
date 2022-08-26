---
abbrlink: 2253555633
alias: 2017/09/07/Python时间模块常用操作总结/index.html
categories:
- python
date: '2017-09-07T10:43:32'
description: ''
tags:
- 时间
- Python
- 总结
title: Python时间模块常用操作总结
---









时间模块常用操作总结为下列各个函数：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import sys
reload(sys)
sys.setdefaultencoding('utf-8')
import time
import datetime
import calendar


def second_to_datetime_string(seconds):
    """
    将从公元0年开始的秒数转换为datetime的string形式
    :param seconds: 从公元0年开始的秒数
    :return: datetime的string形式
    """
    s = time.strftime('%Y-%m-%d %H:%M:%S', time.localtime(float(seconds)))
    year = s.split('-', 1)[0]
    rest = s.split('-', 1)[1]
    year = int(year) - 1970  # datetime是从1970开始的，因此计算时需要减去1970
    return '{}-{}'.format(str(year), rest)


def gregorian_date_to_str(year, month, day):
    """
    将公元日期转换成字符串，year占4位，month占2位，day占2位，位数不足补0
    :param year: 年份，例如2017
    :param month: 月份，例如8或者08
    :param day: 天，例如12或者02或者2
    :return: 返回位数固定的字符串，例如2017-08-22
    """
    return '{}-{}-{}'.format(
        year,
        month if len(str(month)) == 2 else '0{}'.format(month),
        day if len(str(day)) == 2 else '0{}'.format(day)
    )


def gregorian_date_to_str_1(year, month, day):
    """
    将公元日期转换成字符串，year占4位，month占2位，day占2位，位数不足补0
    :param year: 年份，例如2017
    :param month: 月份，例如8或者08
    :param day: 天，例如12或者02或者2
    :return: 返回位数固定的字符串，不带-，例如20170822
    """
    return '{}{}{}'.format(
        year,
        month if len(str(month)) == 2 else '0{}'.format(month),
        day if len(str(day)) == 2 else '0{}'.format(day)
    )


def get_element_from_date_str(date_str):
    """获取一个日期字符串的年月日"""
    return date_str[0:4], date_str[4:6], date_str[6:8]


def is_valid_date(date_str):
    """判断是否是一个有效的日期字符串"""
    try:
        time.strptime(date_str, '%Y%m%d')
        return True
    except ValueError:
        return False


def get_latter_1_day_str(date_str):
    """
    获取data_str后一天的日期字符串
    :param date_str: 指定日期字符串
    :return: 返回指定日期字符串后一天的日期字符串
    """
    dt = datetime.datetime.strptime(date_str, '%Y%m%d')
    one_day = datetime.timedelta(days=1)
    former_day = dt + one_day
    return former_day.strftime('%Y%m%d')


def get_latter_n_day_str(date_str, n):
    """
    获取data_str后n天的日期字符串
    :param date_str: 指定日期字符串
    :return: 返回指定日期字符串后n天的日期字符串
    """
    dt = datetime.datetime.strptime(date_str, '%Y%m%d')
    one_day = datetime.timedelta(days=days)
    former_day = dt + one_day
    return former_day.strftime('%Y%m%d')


def get_yesterday_str():
    """
    获取昨天的日期字符串
    :return: 返回昨天日期字符串
    """
    today = datetime.date.today()
    one_day = datetime.timedelta(days=1)
    yesterday = today - one_day
    return yesterday.strftime('%Y%m%d')


def get_former_1_day_str(date_str):
    """
    获取data_str前一天的日期字符串
    :param date_str: 指定日期字符串
    :return: 返回指定日期字符串前一天的日期字符串
    """
    dt = datetime.datetime.strptime(date_str, '%Y%m%d')
    one_day = datetime.timedelta(days=1)
    former_day = dt - one_day
    return former_day.strftime('%Y%m%d')


def get_former_n_day_str(date_str, n):
    """
    获取data_str前n天的日期字符串
    :param date_str: 指定日期字符串
    :param n: number of day
    :return: 返回指定日期字符串前n天的日期字符串
    """
    dt = datetime.datetime.strptime(date_str, '%Y%m%d')
    n_day = datetime.timedelta(days=n)
    former_n_day = dt - n_day
    return former_n_day.strftime('%Y%m%d')


def get_universal_time():
    """
    获取当前时间
    :return: 返回当前时间，格式：2017-08-29 02:43:19
    """
    t = time.gmtime()
    time_tuple = (t.tm_year, t.tm_mon, t.tm_mday, t.tm_hour, t.tm_min, t.tm_sec)
    dt = datetime.datetime(*time_tuple)
    return dt.strftime('%Y-%m-%d %H:%M:%S')


def datetime_to_gregorian_seconds(dt):
    """
    获取从公元0年1月1日开始到当天0点所经过的秒数
    :param dt: datetime.datetime类型
    :return: 返回从公元0年1月1日开始到当天0点所经过的秒数
    """
    d = dt.date()
    t = dt.time()
    # toordinal 从1年1月1日开始, erlang 的datetime_to_gregorian_seconds和date_to_gregorian_days从0年1月1日开始
    # 当天不算所以需要减1天
    return (d.toordinal() + 365 - 1) * 86400 + time_to_second(t.hour, t.minute, t.second)


def time_to_second(time_h, time_m, time_s):
    """
    根据给定的time_h, time_m, time_s计算当天已过去的时间，秒为单位
    :param time_h: 小时
    :param time_m: 分
    :param time_s: 秒
    :return: 返回计算的second
    """
    return int(time_h) * 3600 + int(time_m) * 60 + int(time_s)


def utc_time_to_second(utc_time):
    """
    根据给定的utc_time计算当天已过去的时间，秒为单位
    :param utc_time: utc时间戳，类似1464830584
    :return: 返回计算的second
    """
    t = datetime.datetime.fromtimestamp(int(utc_time))
    return t.hour * 3600 + t.minute * 60 + t.second


def get_today_start_time():
	"""获取当天开始时间"""
	dt = datetime.datetime.combine(datetime.date.today(), datetime.time.min)
	return dt.strftime('%Y-%m-%d %H:%M:%S')


def get_today_end_time():
	"""获取当天结束时间"""
	dt = datetime.datetime.combine(datetime.date.today(), datetime.time.max)
	return dt.strftime('%Y-%m-%d %H:%M:%S')


def get_final_day_of_current_week():
	"""获取本周最后一天：周天"""
	today = datetime.date.today()
	sunday = today + datetime.timedelta(6 - today.weekday())
	return sunday.strftime('%Y-%m-%d')


def get_final_day_of_current_month():
	"""获取本月最后一天"""
	today = datetime.date.today()
	_, last_day_num = calendar.monthrange(today.year, today.month)
	last_day = datetime.date(today.year, today.month, last_day_num)
	return last_day.strftime('%Y-%m-%d')


def get_final_day_of_last_month():
	"""获取上月最后一天，可能会跨年，需要用timedelta"""
	today = datetime.date.today()
	first = datetime.date(day=1, month=today.month, year=today.year)
	final_day_of_last_month = first - datetime.timedelta(days=1)
	return final_day_of_last_month.strftime('%Y-%m-%d')


def get_final_day_of_current_month(year, month):
	"""获取指定年指定月的最后一天"""
	_, last_day_num = calendar.monthrange(year, month)
	return last_day_num
	# last_day = datetime.date(year, month, last_day_num)
	# return last_day.strftime('%Y-%m-%d')

```

