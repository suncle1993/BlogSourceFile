---
abbrlink: 599463755
alias: 2018/02/01/Benchmarking-Tencent-CLS-with-ab-and-wrk/index.html
categories:
- 效率
date: '2018-02-01T18:46:45'
description: ''
tags:
- benchmark
- ab
- wrk
title: 使用ab和wrk对腾讯CLS进行benchmark测试
---










# 使用ab和wrk对CLS进行benchmark测试

使用ab和wrk对腾讯云日志服务CLS进行压力测试，以此为例对ab和wrk进行说明

# ab

ab，全称是apache benchmark，是apache官方推出的工具。该工具是用来测试Apache服务器的性能的。查看安装的apache的服务器能提供的服务能力，每秒可以处理多少次请求。ab 执行时常用的选项如下表：

| 选项   | 作用                                       |
| ---- | ---------------------------------------- |
| -c   | 并发数， 一次发送的总请求数，默认是一次发一个请求。               |
| -k   | 打开keep-alive，在一个HTTP Session中请求多次。默认是关闭的。 |
| -n   | 请求数， 整个benchmark测试过程中需要发送的请求次数。默认是一次，默认情况下得到的性能参数没有代表性。 |
| -t   | 最大时间，benchmark测试最长时间，默认没有限制。             |
| -u   | 上传文件，PUT操作时使用，需要设置-T选项                   |
| -T   | 设置上传文件的Content-Type                      |
| -p   | postfile，指定包含post数据的文件                   |
| -r   | 当接收到socket错误的时候ab不退出                     |

<!--more-->

## 安装

```shell
apt-get install apache2-utils
```

## 注意事项

- 观察测试工具ab所在机器，以及被测试的前端机的CPU，内存，网络等都不超过最高限度的75%。
- 测试中可能出现端口不足导致的测试失败

需要调整内核参数以支持端口重用，在Linux平台下需要在`/etc/sysctl.conf`文件中添加如下内容

```
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_fin_timeout = 30
kernel.printk = 7 4 1 7
```

然后运行`sudo sysctl –p`生效

## 使用示例

```shell
ab -c 50 -t 60 -n 300000 -k -T 'application/x-protobuf' -p /tmp/post_data.txt -H 'Host: ap-shanghai.cls.myqcloud.com' -H 'Authorization: q-sign-algorithm=sha1&q-ak=AKIDMfonbuXfqpcFicn3YrzwivMelfNwFWcW&q-sign-time=1517472219;1517493819&q-key-time=1517472219;1517493819&q-header-list=content-type;host&q-url-param-list=&q-signature=4a4ed6ddc8ba1dfea73d2bee62def9dce8b0ca3c' http://ap-shanghai.cls.myqcloud.com/log
```

/tmp/post_data.txt数据为google protocol buffer格式的数据

## 结果分析

```
This is ApacheBench, Version 2.3 <$Revision: 1796539 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, https://www.apache.org/

Benchmarking ap-shanghai.cls.myqcloud.com (be patient)
Completed 30000 requests
Completed 60000 requests
Completed 90000 requests
Completed 120000 requests
Completed 150000 requests
Completed 180000 requests
Completed 210000 requests
Finished 223877 requests


Server Software:        openresty
Server Hostname:        ap-shanghai.cls.myqcloud.com
Server Port:            80

Document Path:          /log
Document Length:        0 bytes

Concurrency Level:      50
Time taken for tests:   60.001 seconds
Complete requests:      223877
Failed requests:        0
Keep-Alive requests:    223027
Total transferred:      38726471 bytes
Total body sent:        108604595
HTML transferred:       0 bytes
Requests per second:    3731.24 [#/sec] (mean)
Time per request:       13.400 [ms] (mean)
Time per request:       0.268 [ms] (mean, across all concurrent requests)
Transfer rate:          630.31 [Kbytes/sec] received
                        1767.63 kb/s sent
                        2397.94 kb/s total

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.5      0      34
Processing:     9   13   3.8     13     164
Waiting:        8   13   3.8     13     164
Total:          9   13   3.8     13     164

Percentage of the requests served within a certain time (ms)
  50%     13
  66%     14
  75%     14
  80%     14
  90%     15
  95%     17
  98%     22
  99%     26
 100%    164 (longest request)
```

从测试结果，我们可以看到

- 在50个并发请求的情况下，请求60秒，平均每秒可以处理3731次（也就是说，客户端在这种压力下，看到的QPS为3731）
- 平均每次请求处理的Latency为13.4ms
- 由于开启了keep-alive，连接几乎不耗时间
- 99%的请求都在26ms内完成，最长的请求是164ms

使用腾讯云主机测试结果如下

```
This is ApacheBench, Version 2.3 <$Revision: 1706008 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, https://www.apache.org/

Benchmarking ap-shanghai.cls.myqcloud.com (be patient)

Completed 30000 requests
Completed 60000 requests
Completed 90000 requests
Completed 120000 requests
Completed 150000 requests
Completed 180000 requests
Completed 210000 requests
Completed 240000 requests
Completed 270000 requests
Completed 300000 requests
Finished 300000 requests


Server Software:        openresty
Server Hostname:        ap-shanghai.cls.myqcloud.com
Server Port:            80

Document Path:          /log
Document Length:        0 bytes

Concurrency Level:      50
Time taken for tests:   40.095 seconds
Complete requests:      300000
Failed requests:        0
Keep-Alive requests:    298850
Total transferred:      51894250 bytes
Total body sent:        145500000
HTML transferred:       0 bytes
Requests per second:    7482.21 [#/sec] (mean)
Time per request:       6.683 [ms] (mean)
Time per request:       0.134 [ms] (mean, across all concurrent requests)
Transfer rate:          1263.94 [Kbytes/sec] received
                        3543.82 kb/s sent
                        4807.77 kb/s total
                       
Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.1      0       6
Processing:     4    7   2.9      6     157
Waiting:        4    7   2.9      6     157
Total:          4    7   2.9      6     157

Percentage of the requests served within a certain time (ms)
  50%      6
  66%      7
  75%      7
  80%      8
  90%      9
  95%     10
  98%     14
  99%     18
 100%    157 (longest request)
```

从结果我们可以看到，QPS是非腾讯云主机的2倍，为7482

# wrk

wrk是一个用来做HTTP benchmark测试的工具。可以产生显著的压力。

## 安装

```shell
apt-get install libssl-dev
git clone https://github.com/wg/wrk.git
cd wrk
make
cp wrk /usr/sbin
```

## 使用示例

```shell
wrk -c 50 -d 60 -t 5 -s /tmp/wrk_post.lua http://ap-shanghai.cls.myqcloud.com
```

请求的内容在`/tmp/wrk_post.lua`中规定，有5个线程，开启的连接有50个，运行60秒

其中`/tmp/wrk_post.lua`中的内容是

```lua
request = function()
  mypath = "/tmp/post_data.txt";
  local file = io.open(mypath, "r");
  assert(file);
  local body = file:read("*a");      -- 读取所有内容
  file:close();
  wrk.body = body
  path = "/log"
  wrk.headers["Content-Type"] = "application/x-protobuf"
  wrk.headers["Host"] = "ap-shanghai.cls.myqcloud.com"
  wrk.headers["Authorization"] = "q-sign-algorithm=sha1&q-ak=AKIDMfonbuXfqpcFicn3YrzwivMelfNwFWcW&q-sign-time=1517472219;1517493819&q-key-time=1517472219;1517493819&q-header-list=content-type;host&q-url-param-list=&q-signature=4a4ed6ddc8ba1dfea73d2bee62def9dce8b0ca3c"
  return wrk.format("POST", path)
end
```

## 结果分析

```
Running 1m test @ http://ap-shanghai.cls.myqcloud.com
  5 threads and 50 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    15.91ms   25.52ms 880.85ms   98.05%
    Req/Sec   745.32    105.11   848.00     94.79%
  221561 requests in 1.00m, 36.55MB read
Requests/sec:   3688.17
Transfer/sec:    623.03KB
```

从测试结果，我们可以看到

- 在5个并发请求的情况下，开启50个连接，请求60秒，平均每秒可以处理3688次（也就是说，客户端在这种压力下，看到的QPS为3688）
- 平均每次请求处理的Latency为15.91ms

使用腾讯云主机测试结果如下

```
Running 1m test @ http://ap-shanghai.cls.myqcloud.com
  5 threads and 50 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     6.77ms    3.42ms  90.45ms   94.82%
    Req/Sec     1.53k   119.04     1.74k    79.27%
  457574 requests in 1.00m, 75.48MB read
Requests/sec:   7623.03
Transfer/sec:      1.26MB
```

从结果我们可以看到，QPS是非腾讯云主机的2倍，为7623

# 总结

以上就是用开源的benchmark工具来从客户端的角度来衡量所能获取的QPS以及Latency。但从客户端看到的性能会受到各种因素的影响，例如请求的方式，本机的资源（CPU，内存，网络），CLS的网络状况，CLS的负载等都会影响客户端看到的性能指标。需要根据实际情况来查看性能瓶颈是来自于CLS还是来自于本机。

---

参考：

1. [使用ab和wrk对OSS进行benchmark测试](https://yq.aliyun.com/articles/35251)
