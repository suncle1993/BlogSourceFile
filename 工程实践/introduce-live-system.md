---
abbrlink: 2029292735
alias: 2018/12/18/introduce-live-system/index.html
categories:
- 工程实践
date: '2018-12-18T20:48:22'
description: ''
tags:
- 直播
- cdn
- livego
title: 直播系统介绍
---









# 演示

本地演示：ffmpeg/obs + livego + mpv

1. 推流选择ffmpeg或者obs
2. 流媒体服务直接使用livego
3. 播放使用mpv

推流截图：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/image/tech/live-20181218/obs-push.jpg)

拉流截图：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/image/tech/live-20181218/hls-pull.jpg)

<!--more-->

## livego

```shell
git clone https://github.com/gwuhaolin/livego.git
go build livego.go  # 编译
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build livego.go  # 编译成linux环境下的可执行文件
./livego
```

## 流地址

```
rtmp推流：rtmp://127.0.0.1:1935/live/taylor
rtmp拉流：rtmp://127.0.0.1:1935/live/taylor
hdl拉流：http://127.0.0.1:7001/live/taylor.flv
hls拉流：http://127.0.0.1:7002/live/taylor.m3u8
```

如果使用ffmpeg推流

```
ffmpeg -re -i ~/Documents/Taylor\ Swift\ -\ You\ Belong\ With\ Me.mp4 -c copy -f flv rtmp://localhost:1935/live/taylor
```
# 视频流

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/image/tech/live-20181218/%E7%9B%B4%E6%92%AD%E7%B3%BB%E7%BB%9F%E6%B5%81%E7%A8%8B.png)

# 直播系统组成

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/image/tech/live-20181218/%E7%9B%B4%E6%92%AD%E7%B3%BB%E7%BB%9F%E7%BB%84%E6%88%90.png)

# 直播流协议

## RTMP

Rtmp规范1.0：https://suncle.me/2018/03/09/rtmp%E8%A7%84%E8%8C%831-0/

Rtmp规范1.0 en：http://wwwimages.adobe.com/www.adobe.com/content/dam/acom/en/devnet/rtmp/pdf/rtmp_specification_1.0.pdf

1. RTMP协议是应用层协议，基于下层的传输层协议TCP
2. 要建立一个有效的RTMP Connection链接，首先要握手。但是实际使用过程中对握手数据校验不严格
3. Adobe公司
4. 低延迟，内容延迟可以低于3秒
5. 需要编解码
6. 几乎所有的稳定推流协议都是RTMP

## HDL

HDL协议中封装格式使用的是FLV，HDL又叫做HTTP-FLV

1. 基于HTTP
2. 低延迟，内容延迟可以低于3秒
3. 需要编解码

## HLS

Http Live Streaming。

1. 苹果公司
2. 基于HTTP
3. HTML5可以直接播放，不需要编解码，需要在服务端切片，有Stream Segmenter的概念
4. 格式：
   1. m3u8：索引文件，以m3u8为后缀。用文本方式对媒体文件进行描述，由一系列标签组成
   2. ts：传输流文件，视频编码主要格式h264/mpeg4，音频为acc/MP3。
5. 延迟较高，一般在10秒左右

## 使用情况

对于正常的直播场景，多数都是推流使用Rtmp协议，拉流使用HLS协议

---

参考：

1. SRS的C++版本：https://github.com/ossrs/srs
2. SRS的Golang版本：https://github.com/gwuhaolin/livego
3. 云直播系统架构与实施：https://blog.csdn.net/qiansg123/article/details/80124296
4. UCloud：http://blog.ucloud.cn/archives/699