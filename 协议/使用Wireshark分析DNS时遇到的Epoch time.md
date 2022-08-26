---
abbrlink: 3968172928
alias: 2014/11/21/使用Wireshark分析DNS时遇到的Epoch time/index.html
categories:
- 协议
date: '2014-11-21T19:09:00'
tags:
- Wireshark
title: 使用Wireshark分析DNS时遇到的Epoch time
---






# 使用Wireshark分析DNS时遇到的Epoch time

首先看一下Wireshark分析DNS的情况（如下图）：
![](https://images0.cnblogs.com/blog/637108/201411/211855521096466.png)

这是协议树的第一项，第一项中的第五行出现了Epoch Time，查阅资料之后才知道：

**Epoch指的是一个特定的时间（新纪元时间）：1970-01-01 00:00:00 UTC。（协调世界时Universal Time Coordinated）**

图片中的epoch time是1416328469.028274000seconds，假如我们将一年算作365天

1416328469.028274000/3600/24/365约等于44.91148.当然后面还有很多小数了。

也就是从1970年1月1号开始的44.91148年，推到现在刚好是2014年11月左右，这个数据肯定是不完全准确的，因为一年可能有366天。

**wireshark中显示新纪元时间`Epoch Time`的方法**

依次选择菜单中的View–>Time Display Fromat–>Seconds Since Epoch (1970-01-01): 1234567890.123456

即可将wireshark显示的时间格式设置为新纪元时间（Epoch Time），自 1970 年 1 月 1 日（00:00:00 GMT）以来的秒数。

**最后，epoch time涉及到一个2038 bug**

32位二进制数字表示时间，它们最多只能表示至协调世界时间2038年1月19日3时14分07秒，目前解决方案是把系统由32位转为64位系统。在64位系统下，此时间最多可以表示到292,277,026,596年12月4日15时30分08秒。