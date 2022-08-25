---
title: Wireshark中的Checksum 0x90c5 validation disabled问题
date: '2014-11-21T20:57:34'
categories:
  - 协议
tags:
  - Wireshark
---

# Wireshark中的Checksum: 0x90c5 [validation disabled]问题

废话不多说先上问题图：

![](https://images0.cnblogs.com/blog/637108/201411/212041362505163.png)

这是我在做关于DNS协议PPT的时候出现的协议树第五项展开结果，可以发现其中有一行为：

**Header checksum:0x90c5[validation disabled]**

按正常情况来说中括号中出现的应该是[correct]而不是[validation disabled]，意识是验证禁用，在Wireshark官网上查询了到了这个问题，问题的链接如下：

　　https://ask.wireshark.org/questions/2253/tcp-checksum-validation-disabled

这是ask的问题：

> Is there any reason why the TCP checksum validation would be disabled. I believe I spotted a host communicating to a CnC server then being redirected to another potential drive by download site.
> 
> The TCP validation disabled checksum is for incoming traffic from the potential CnC server.
> 
> Thanks

这是其中的一个支持率比较高的answer：

> Yes. The reason is that Wireshark is very often used to capture the network frames of the same PC that is running Wireshark. This usually results in the checksums of outgoing frames being incorrect since they are only calculated for transmission by the network card after they were already recorded by Wireshark. To avoid constant "checksum error" messages it was decided to have the checksum validation disabled by default.
> 
> It may sound stupid to disabled checkum validation since we want to find damaged packets with Wireshark when tracking down errors. But the fact is that frames with damaged checksums won't survive much long anyway since every switch or router will probably drop them for being defective - and still, if the frame makes it to your network card it will still drop it before Wireshark even sees it. This is the reason why some commercial sniffers have specialized NIC drivers for certain cards that will allow capturing damaged frames with them.

大致意思就是：

　　有时候TCP和UDP校验和会由网卡计算，因此wireshark抓到的本机发送的TCP/UDP数据包的校验和都是错误的，这样检验校验和根本没有意义。所以Wireshark不自动做TCP和UDP校验和的校验

如果要校验校验和：可以在edit->preference->protocols中选择相应的TCP或者UDP协议，在相应的地方打钩。操作截图如下：

![](https://images0.cnblogs.com/blog/637108/201411/212108392656254.png)

好了，关于checksum的validation disabled问题就介绍到这里。