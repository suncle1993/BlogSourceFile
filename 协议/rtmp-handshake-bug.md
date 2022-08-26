---
abbrlink: 2249737360
alias: 2018/03/20/rtmp-handshake-bug/index.html
categories:
- 协议
date: '2018-03-20T12:45:29'
description: ''
tags:
- rtmp
- 握手
title: 排查rtmp协议推流时握手bug
---









# 概况

转推流程序的过程：从一个观看地址拉流，然后推流到另一个推流地址。主要用于cdn之间转推，目前市面上大多数cdn厂商都愿意不支持动态转推，因此只能通过转推流程序进行转推。

bug现象：使用obs studio推流到微赞可以成功，但是使用Erlang版本的转推流程序推流到微赞却失败。

日志如下：

```
14:12:35.926 [debug] payload [{amf,["onBWDone",0,null]}], msgtype[command_msg_4_amf0] 
14:12:35.949 [debug] play succ ======> url ["/live-sz/w1520993434573948"] 
14:12:35.949 [debug] {rtmp_msg,4,0,data_msg_4_amf0,1,{amf,["|RtmpSampleAccess",true,true]}}
14:12:35.949 [debug] {rtmp_msg,4,0,data_msg_4_amf0,1,{amf,["onStatus",{object,[{"code","NetStream.Data.Start"}]}]}}
14:12:36.038 [error] gen_server <0.122.0> terminated with reason: no match of right hand value <<0,0,0,0,0,0,0,0,113,142,194,240,185,25,41,180,242,33,5,112,128,97,178,8,79,179,28,53,152,242,82,43,234,104,113,246,170,189,182,146,122,36,155,3,152,180,226,122,36,97,52,67,53,158,107,170,178,119,209,132,40,233,102,182,142,233,218,71,55,8,121,67,117,58,130,91,107,224,202,5,1,132,37,245,143,231,20,198,121,204,57,80,102,165,104,245,79,71,254,169,15,3,166,12,148,45,24,62,253,66,93,139,84,139,54,236,47,5,98,95,51,231,222,144,8,153,232,166,227,151,57,98,214,63,238,167,212,49,51,160,83,248,246,199,...>> in rtmp_handshake:create_c2/2 line 61
```

<!--more-->

很显然是`rtmp_handshake:create_c2/2`函数出现匹配错误，对应代码如下：

```erlang
-spec create_c2(C0C1, S0S1S2) -> Result when
          C0C1 :: iodata(),
          S0S1S2 :: binary(),
          Result :: {ok, C2},
          C2 :: iolist().
create_c2(C0C1, S0S1S2) when is_list(C0C1) ->
    create_c2(iolist_to_binary(C0C1), S0S1S2);
create_c2(<<_C0:1/binary, C1:16#600/binary>>, <<S0:1/binary, S1:16#600/binary, S2:16#600/binary>>) ->
    <<3>> = S0,
    case C1 of
        <<_:32, 0:32, _/binary>> ->
            S2 = C1,
            {ok, S1};
        _ ->
            {ok, S1DigestData} = verify_s1(S1),    
            DigestKey = crypto:hmac(sha256, ?C2_PUBLIC_KEY, S1DigestData),
            C2Len = 16#600,
            C2DigestDataLen = 32,
            RandomBin = random_binary(C2Len - 8 - C2DigestDataLen),
            {T1, T2, _} = now(),
            Epoch = <<(1000000 * T1 + T2):32/little>>,
            Data = [Epoch, binary:part(S1, 0, 4), RandomBin],
            S2DigestData = crypto:hmac(sha256, DigestKey, Data),
            {ok, [Data, S2DigestData]}
    end.
```

rtmp握手过程中C1数据包匹配`<<_:32, 0:32, _/binary>>`格式后和S2数据包匹配不成功，程序直接crash dump。因此需要弄清楚rtmp握手过程中是否有对S2和C1进行匹配验证。

# Rtmp握手过程

此处重点关注rtmp握手过程中的C1和S2数据包。

先看官方文档中的握手过程，中文翻译版本可以参见：[rtmp规范1.0](https://suncle.me/2018/03/09/rtmp%E8%A7%84%E8%8C%831-0/)。 官方文档中对于是否要保证C1和S2完全一致，并没有明确说法。因此可以先参见obs studio依赖的**`librtmp`**库，看握手过程是如何处理的。obs studio依赖的**`librtmp`**的代码如下连接：

1. https://github.com/obsproject/obs-studio/blob/master/plugins/obs-outputs/librtmp/rtmp.c
2. https://github.com/obsproject/obs-studio/blob/master/plugins/obs-outputs/librtmp/handshake.h

第一个链接rtmp.c中的代码是推流地址中没有加密串的情况下的握手过程代码，第二个链接handshake.h中的代码是推流地址中有加密串的情况下的握手过程代码。代码中使用条件编译`CRYPTO`宏来选择编译不同的代码。其中`HandShake`函数属于客户端的握手函数，`SHandShake`属于服务端的握手函数。非加密版本具体C语言代码如下（已添加对应的中文注释进行说明）：

```c
#ifndef CRYPTO
static int
HandShake(RTMP *r, int FP9HandShake)
{
    //C0,C1 -- S0, S1, S2 -- C2 消息握手协议
    int i;
    uint32_t uptime, suptime;
    int bMatch;
    char type;
    char clientbuf[RTMP_SIG_SIZE + 1], *clientsig = clientbuf + 1;
    char serversig[RTMP_SIG_SIZE];

    clientbuf[0] = 0x03;  //C0, 一个字节。03代表协议版本号为3 		/* not encrypted */

    uptime = htonl(RTMP_GetTime());  //这是一个时间戳，放在C1消息头部  
    memcpy(clientsig, &uptime, 4);

    memset(&clientsig[4], 0, 4);  //后面放4个字节的空数据然后就是随机数据  

    //后面是随机数据，总共1536字节的C1消息 
#ifdef _DEBUG
    for (i = 8; i < RTMP_SIG_SIZE; i++)
        clientsig[i] = 0xff;
#else
    for (i = 8; i < RTMP_SIG_SIZE; i++)
        clientsig[i] = (char)(rand() % 256);
#endif

    //发送C0， C1消息  
    if (!WriteN(r, clientbuf, RTMP_SIG_SIZE + 1))
        return FALSE;

    //下面读一个字节也就是S0消息，看协议是否一样  
    if (ReadN(r, &type, 1) != 1)	/* 0x03 or 0x06 */
        return FALSE;

    RTMP_Log(RTMP_LOGDEBUG, "%s: Type Answer   : %02X", __FUNCTION__, type);

    if (type != clientbuf[0])  //C/S版本不一致 
        RTMP_Log(RTMP_LOGWARNING, "%s: Type mismatch: client sent %d, server answered %d",
                 __FUNCTION__, clientbuf[0], type);

    //读取S1消息，里面有服务器运行时间  
    if (ReadN(r, serversig, RTMP_SIG_SIZE) != RTMP_SIG_SIZE)
        return FALSE;

    /* decode server response */

    memcpy(&suptime, serversig, 4);
    suptime = ntohl(suptime);

    RTMP_Log(RTMP_LOGDEBUG, "%s: Server Uptime : %d", __FUNCTION__, suptime);
    RTMP_Log(RTMP_LOGDEBUG, "%s: FMS Version   : %d.%d.%d.%d", __FUNCTION__,
             serversig[4], serversig[5], serversig[6], serversig[7]);

    /* 2nd part of handshake */
    if (!WriteN(r, serversig, RTMP_SIG_SIZE))  //发送C2消息，内容就等于S1消息的内容。  
        return FALSE;

    //读取S2消息  
    if (ReadN(r, serversig, RTMP_SIG_SIZE) != RTMP_SIG_SIZE)
        return FALSE;

    //服务端返回的S2消息和C1消息进行了比对，但是即使没有match，也是返回TRUE，只是打印了log 
    bMatch = (memcmp(serversig, clientsig, RTMP_SIG_SIZE) == 0);
    if (!bMatch)
    {
        RTMP_Log(RTMP_LOGWARNING, "%s, client signature does not match!", __FUNCTION__);
    }

    /* er, totally unused? */
    (void)FP9HandShake;
    return TRUE;
}

static int
SHandShake(RTMP *r)
{
    ...
    return TRUE;
}
#endif

```

rtmp握手过程中确实存在对S2和C1进行匹配验证的操作，但是这个操作并不影响握手是否是成功的，只是添加了一条warnning日志而已。因此obs studio还是能推流成功。相对应的在我们的转推流程序中，需要针对这个情况不进行强认证，删除掉匹配的操作即可。

# 抓包分析

以微赞和网宿为例

- [obs推流网宿握手成功的包点此下载](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/rtmp_deliver/file/obs-netcenter-handshake-success.pcapng)
- [obs推流微赞握手成功的包点此下载](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/rtmp_deliver/file/obs-vzan-handshake-success.pcapng)

网宿推流没有走加密流程，S2和C1匹配，具体数据包截图如下：

![obs-netcenter-handshake-success.jpg](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/rtmp_deliver/image/obs-netcenter-handshake-success.jpg)

微赞推流走加密流程，S2和C1不匹配，具体数据包截图如下：

![obs-vzan-handshake-success.jpg](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/rtmp_deliver/image/obs-vzan-handshake-success.jpg)

到此，整个rtmp推流握手过程就比较清楚了。

因此只需要将Erlang代码的流程更改下即可（删除S2和C1的匹配过程），见下面的Erlang代码：

```erlang
-spec create_c2(C0C1, S0S1S2) -> Result when
          C0C1 :: iodata(),
          S0S1S2 :: binary(),
          Result :: {ok, C2},
          C2 :: iolist().
create_c2(C0C1, S0S1S2) when is_list(C0C1) ->
    create_c2(iolist_to_binary(C0C1), S0S1S2);
create_c2(<<_C0:1/binary, C1:16#600/binary>>, <<S0:1/binary, S1:16#600/binary, _S2:16#600/binary>>) ->
    <<3>> = S0,
    case C1 of
        <<_:32, 0:32, _/binary>> ->
          {ok, S1};
        _ ->
            {ok, S1DigestData} = verify_s1(S1),    
            DigestKey = crypto:hmac(sha256, ?C2_PUBLIC_KEY, S1DigestData),
            C2Len = 16#600,
            C2DigestDataLen = 32,
            RandomBin = random_binary(C2Len - 8 - C2DigestDataLen),
            {T1, T2, _} = now(),
            Epoch = <<(1000000 * T1 + T2):32/little>>,
            Data = [Epoch, binary:part(S1, 0, 4), RandomBin],
            S2DigestData = crypto:hmac(sha256, DigestKey, Data),
            {ok, [Data, S2DigestData]}
    end.
```

至此，转推流成功，示例图如下：

![rtmp_deliver_success.jpg](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/rtmp_deliver/image/rtmp_deliver_success.jpg)

# 结论

虽然Adobe公司自己出的rtmp协议不是iso标准的，但是你们这些公司好歹也尽量按照规定来啊，贼鸡儿坑。

---

参考：

- obs-studio: https://github.com/obsproject/obs-studio
- obs studio握手：https://github.com/obsproject/obs-studio/blob/master/plugins/obs-outputs/librtmp/handshake.h
- RTMPdump（libRTMP）握手源代码：https://blog.csdn.net/leixiaohua1020/article/details/12954329
- rtmp协议过程分析：https://www.cnblogs.com/lidabo/p/7355262.html
- librtmp使用示例： https://github.com/leixiaohua1020/simplest_librtmp_example
- rtmplib rtmp协议过程分析：https://www.cnblogs.com/lidabo/p/7355262.html