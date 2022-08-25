---
title: 关于Differentiated Services Field (DS Field)的诸多疑问
date: '2014-11-22T14:22:34'
categories:
  - 协议
tags:
  - Wireshark
---

先上疑问截图：

![](https://images0.cnblogs.com/blog/637108/201411/221335248908436.png)

**这是用wireshark抓包时协议树的某一项的展开结果：IPV4 header。其中有一项如下：**

![](https://images0.cnblogs.com/blog/637108/201411/221338404212533.png)

大家在抓包时看到这儿也许很多人和我一样都会有疑问，但是又不知道是什么原因，下面我们就来分析一下：

Differentiated Services Field（下面简称为DS Field）的意思是区分服务领域。DS Field总共有8位。下面是DS字段的结构：

```
0    1    2    3    4    5    6    7  
     +---+---+---+---+---+---+---+---+---+---+  
|          DSCP           |   CU   |  
     +---+---+---+---+---+---+---+---+---+---+
```
  
DSCP：区分服务代码点，即DS标记值，IETF于1998年12月发布了Diff-Serv（Differentiated Service）的QoS分类标准. 它在每个数据包IP头部的服务类别TOS标识字节中，利用已使用的6比特和未使用的2比特字节，通过编码值来区分优先级。用于选择PHB（单中断段行为）。PHB描述了DS节点对具有相同DSCP的分组采用的外部可见的转发行为  
CU：当前尚未使用

在这个IPV4 header中DS Field的8位为0x00。

低2位explicit congestion notification：显示拥塞通知（ECN），使 BIG-IP 系统能够前瞻性地向同类设备发出调度路由器将超载的信号，以便它们能够采取避退措施。

后面的两位0有四个不同编码codepoints（CU=ECN）:

-   `00`——Non-ECT非ECN-Capable运输
-   `10`——ECN运输能力等(0)
-   `01`——ECN运输能力等(1)
-   `11——遇到交通堵塞，CE`

 **总结：DS Field的两个部分DSCP和CU组合成一个可扩展性相对较强的方法以此来保证IP的服务质量。**

参考资料：

关于Explicit_Congestion_Notification：[http://en.wikipedia.org/wiki/Explicit_Congestion_Notification](http://en.wikipedia.org/wiki/Explicit_Congestion_Notification)

关于Definition of the Differentiated Services Field (DS Field) in the IPv4 and IPv6 Headers:[http://www.cnblogs.com/CHLL55/p/4115107.html](http://www.cnblogs.com/CHLL55/p/4115107.html)
