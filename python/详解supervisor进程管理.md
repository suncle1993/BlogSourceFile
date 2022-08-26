---
abbrlink: 1143805574
alias: 2017/09/25/详解supervisor进程管理/index.html
categories:
- python
date: '2017-09-25T17:05:22'
description: ''
tags:
- supervisor
- 进程
title: 详解supervisor进程管理
---









# supervisor

使用Python编写的进程管理程序supervisor来管理Python程序那是最合适不过了，supervisor基于CS架构，主要有以下两个组成部分：

1. **supervisord**：supervisord是supervisor的服务端程序。负责启动子程序，应答客户端命令，子程序日志记录，对进程变化发送事件通知等
2. **supervisorctl：** 客户端命令行工具，可以连接服务器端，进行进程的启动、关闭、重启、状态查看等。重要的一点是，supervisorctl不仅可以连接到本机上的supervisord，还可以连接到远程的supervisord，当然在本机上面是通过UNIX socket连接的，远程是通过TCP socket连接的。supervisorctl和supervisord之间的通信，是通过xml_rpc完成的。 相应的配置在[supervisorctl]块里面

<!--more-->

# 安装

基于ubuntu16.04，不同linux发行版均可使用包管理器进行安装，也可以使用源码安装和pip安装

```
apt-get install python-setuptools
apt-get install supervisor
```

pip安装方法

```shell
pip install supervisor
```

当前最新版本3.3.3，supervisord.conf 和supervisord.d文件夹已自动生成在/etc/supervisor/目录下。可以使用echo_supervisord_conf命令将配置信息重定向到制定目录，比如/etc

```shell
echo_supervisord_conf > /etc/supervisor/supervisord.conf
```

服务端启动

```shell
supervisord -c /etc/supervisor/supervisord.conf
```

查看supervisord是否在运行

```
ps aux | grep supervisord
```

# supervisord.conf详解

使用echo_supervisord_conf查看supervisord.conf可选的配置项：

```shell
echo_supervisord_conf help
```

详情如下

```ini
; Sample supervisor config file.
;
; For more information on the config file, please see:
; http://supervisord.org/configuration.html
;
; Notes:
;  - Shell expansion ("~" or "$HOME") is not supported.  Environment
;    variables can be expanded using this syntax: "%(ENV_HOME)s".
;  - Comments must have a leading space: "a=b ;comment" not "a=b;comment".

[unix_http_server]
file=/tmp/supervisor.sock   ; socket 文件路径
;chmod=0700                 ; socket 文件 模式 (默认 0700)
;chown=nobody:nogroup       ; socket file uid:gid owner
;username=user              ; 使用supervisorctl连接的用户,默认没有用户
;password=123               ; 使用supervisorctl连接的用户密码,默认没有密码

;[inet_http_server]         ; Web Server和远程的supervisorctl 配置块(默认关闭)
;port=127.0.0.1:9001        ; 监听的地址和端口
;username=user              ; 登录用户，默认没有用户
;password=123               ; 登录密码，默认没有密码

[supervisord]
logfile=/tmp/supervisord.log ; supervisord进程日志路径
logfile_maxbytes=50MB        ; supervisord进程单个日志文件大小，默认为50M
logfile_backups=10           ; 日志文件个数，默认为10个
loglevel=info                ; 日志级别，默认为info，还支持debug,warn,trace)
pidfile=/tmp/supervisord.pid ; supervisord的pid文件路径。
nodaemon=false               ; 如果是true，supervisord进程将在前台运行 默认为false(后台运行)
minfds=1024                  ; 这个是最少系统空闲的文件描述符，低于这个值supervisor将不会启动
minprocs=200                 ; 最小可用的进程描述符，低于这个值supervisor也将不会正常启动
;umask=022                   ; 进程创建文件的掩码 (默认 022)
;user=chrism                 ; user参数指定的用户也可以对supervisord进行管理
;identifier=supervisor       ; supervisord的标识符，默认supervisor
;directory=/tmp              ; 当supervisord以守护进程运行的时候，启动supervisord进程之前，会先切换到这个目录
;nocleanup=true              ; false的时候 supervisord进程启动的时候 会在把以前子进程产生的临时文件清除掉(true不清除)
;childlogdir=/tmp            ; 当子进程日志路径为AUTO的时候，子进程日志文件的存放路径 (默认 $TMP)
;environment=KEY="value"     ; 这个是用来设置环境变量的，supervisord在linux中启动默认继承了linux的 环境变量，在这里可以设置supervisord进程特有的其他环境变量supervisord启动子进程时，子进程会拷贝父进程的内存空间内容。 所以设置的这些环境变量也会被子进程继承 (默认不设置)
;strip_ansi=false            ; 默认为falsh，如果设置为true，会清除子进程日志中的所有ANSI(\n,\t) 序列

; 这个选项是给XML_RPC用的，如果想使用supervisord或者web server 必须要开启
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ; supervisorctl本地连接supervisord，使用本地UNIX socket
;serverurl=http://127.0.0.1:9001 ; supervisorctl远程连接supervisord的时候，用到的地址和端口
;username=chris              ; 连接登录的用户名，需要和http_username一致
;password=123                ; 连接登录的密码，需要和http_password一致
;prompt=mysupervisor         ; 输入用户名密码时候的提示符 默认:mysupervisor 
;history_file=~/.sc_history  ; 指定历史命令的文件


;[program:theprogramname]      ; 指定program配置
;command=/bin/cat              ; 要执行的进程 可带参数 $1 $2 $3  注意!! 执行的进程不能是守护进程 ! !
;process_name=%(program_name)s ; numprocs参数为1时，就不用管这个参数 默认值%(program_name)s也就是上面的那个program冒号后面的名字
;numprocs=1                    ; 启动进程的数目。当不为1时，就是进程池的概念，默认为1
;directory=/tmp                ; 进程运行前，会前切换到这个目录
;umask=022                     ; 进程掩码 (default None)
;priority=999                  ; 子进程启动关闭优先级，优先级低的，最先启动，关闭的时候最后关闭 (default 999)
;autostart=true                ; 设置为true 子进程将在supervisord启动后被自动启动，默认为true
;startsecs=1                   ; 设置子进程启动多少秒之后，此时状态如果是running，则我们认为启动成功
;startretries=3                ; 进程启动失败后，最大尝试启动的次数 当超过3次后，supervisor将把此进程的状态置为FAIL
;autorestart=unexpected        ; 设置子进程挂掉后自动重启的情况，有三个选项，false,unexpected和true。如果为false的时候，无论什么情况下，都不会被重新启动，如果为unexpected，只有当进程的退出码不在上面的exitcodes里面定义的退出码的时候，&gt;才会被自动重启。当为true的时候，只要子进程挂掉，将会被无条件的重启。默认为unexpected
;exitcodes=0,2                 ; 注意和上面的的autorestart=unexpected对应 exitcodes里面的定义的退出码是expected的。
;stopsignal=QUIT               ; 进程停止信号，可以为TERM, HUP, INT, QUIT, KILL, USR1, or USR2等信号 默认为TERM 当用设定的信号去杀掉进程，退出码会被认为是expected
;stopwaitsecs=10               ; 这个是当我们向子进程发送stopsignal信号后，到系统返回信息给supervisord，所等待的最大时间。 超过这个时间，supervisord会向该子进程发送一个强制kill的信号(默认10秒)
;stopasgroup=false             ; 这个东西主要用于，supervisord管理的子进程，这个子进程本身还有子进程 那么我们如果仅仅干掉supervisord的子进程的话，子进程的子进程有可能会变成孤儿进程 所以咱们可以设置这个选项，把整个该子进程的整个进程组都干掉  设置为true的话，一般killasgroup也会被设置为true 该选项发送的是stop信号(def false)
;killasgroup=false             ; 这个和上面的stopasgroup类似，不过发送的是kill信号(def false)
;user=chrism                   ; 如果supervisord是root启动，我们在这里设置这个非root用户，可以用来管理该program 默认不设置
;redirect_stderr=true          ; 为true，则stderr的日志会被写入stdout日志文件中 (default false)
;stdout_logfile=/a/path        ; 子进程的stdout的日志路径，可以指定路径，AUTO，none等三个选项 设置为none的话，将没有日志产生。设置为AUTO的话，将随机找一个地方成日志文件，而且当supervisord重新启动的时候，以前的日志文件会被清空。当 redirect_stderr=true的时候，sterr也会写进这个日志文件
;stdout_logfile_maxbytes=1MB   ; 日志文件最大大小，和[supervisord]中定义的一样 (default 50MB)
;stdout_logfile_backups=10     ; 和[supervisord]定义的一样 (0 means none, default 10)
;stdout_capture_maxbytes=1MB   ; 这个东西是设定capture管道的大小，当值不为0的时候，子进程可以从stdout发送信息，而supervisor可以根据信息，发送相应的event  (default 0)
;stdout_events_enabled=false   ; 为ture的时候，当子进程由stdout向文件描述符中写日志的时候，将触发supervisord发送PROCESS_LOG_STDOUT类型的event(default false)
;stderr_logfile=/a/path        ; 设置stderr写的日志路径，当redirect_stderr=true。这个就不用设置了，设置了也是白搭。因为它会被写入stdout_logfile的同一个文件中 default AUTO(随便找个地存，supervisord重启被清空)
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups=10     ; # of stderr logfile backups (default 10)
;stderr_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A="1",B="2"       ; 这个是该子进程的环境变量，和别的子进程是不共享的
;serverurl=AUTO                ; override serverurl computation (childutils)

;[eventlistener:theeventlistenername]    ; eventlistener其实和program的地位是一样的，也是suopervisor启动的子进程，不过它干的活是订阅supervisord发送的event。他的名字就叫listener了。我们可以在listener里面做一系列处理，比如报警....
;command=/bin/eventlistener    ; 和上面的program一样，表示listener的可执行文件的路径
;process_name=%(program_name)s ; 这个也一样，进程名，当下面的numprocs为多个的时候，才需要。否则默认就OK了
;numprocs=1                    ; 相同的listener启动的个数，默认1
;events=EVENT                  ; event event事件的类型，也就是说，只有写在这个地方的事件类型。才会被发送
;buffer_size=10                ; event队列缓存大小 (default 10)
;directory=/tmp                ; 进程执行前，会切换到这个目录下执行 (def no cwd)
;umask=022                     ; umask for process (default None)
;priority=-1                   ; 启动优先级 (default -1)
;autostart=true                ; supervisord启动时一起启动 (default: true)
;startsecs=1                   ; 设置子进程启动多少秒之后，此时状态如果是running，则我们认为启动成功了 (def. 1)
;startretries=3                ; 失败最大尝试次数 (default 3)
;autorestart=unexpected        ; 和program一样 (def: unexpected)
;exitcodes=0,2                 ; 'expected' exit codes used with autorestart (default 0,2)
;stopsignal=QUIT               ; signal used to kill process (default TERM)
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;stopasgroup=false             ; send stop signal to the UNIX process group (default false)
;killasgroup=false             ; SIGKILL the UNIX process group (def false)
;user=chrism                   ; setuid to this UNIX account to run the program
;redirect_stderr=false         ; redirect_stderr=true is not allowed for eventlisteners
;stdout_logfile=/a/path        ; stdout log path, NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
;stderr_logfile=/a/path        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups=10     ; # of stderr logfile backups (default 10)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A="1",B="2"       ; process environment additions
;serverurl=AUTO                ; override serverurl computation (childutils)

;[group:thegroupname]          ; 给programs分组，划分到组里面的program。我们就不用一个一个去操作了 我们可以对组名进行统一的操作。 注意：program被划分到组里面之后，就相当于原来的配置从supervisor的配置文件里消失了supervisor只会对组进行管理，而不再会对组里面的单个program进行管理了
;programs=progname1,progname2  ; 组成员，用逗号分开
;priority=999                  ; 优先级，相对于组和组之间 (default 999)

;[include]                     ;包含的程序配置文件
;files = relative/directory/*.ini
```

# 提取supervisord.conf模板

采用supervisord.conf和program.conf分离的方式进行本机进程管理，从默认配置中抽取出常用的supervisord模板

```ini
; supervisor config file

[unix_http_server]
file=/var/run/supervisor.sock   ; (the path to the socket file)

[inet_http_server]         ; inet (TCP) server disabled by default
port=127.0.0.1:9001        ; (ip_address:port specifier, *:port for all iface)
username=erow              ; (default is no username (open server))
password=j                 ; (default is no password (open server))

[supervisord]
logfile=/var/log/supervisor/supervisord.log ; (main log file;default $CWD/supervisord.log)
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
loglevel=debug               ; (log level;default info; others: debug,warn,trace)
logfile_maxbytes=10MB        ; (max main logfile bytes b4 rotation;default 50MB)
logfile_backups=10           ; (num of main logfile rotation backups;default 10)

[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///var/run/supervisor.sock ; use a unix:// URL  for a unix socket

[include]
files = /etc/supervisor/conf.d/*.ini
```

模板抽取基于以下考虑

- 必填选项不可少
- 服务端支持本地supervisorctl客户端连接也支持远程连接
- 测试环境日志级别设置为debug，生产环境日志级别设置为info，修改loglevel字段即可
- 日志文件大小和个数需要参考服务端所在机器的磁盘大小，会产生stderr和stdout两种日志
- pid文件和log文件都不建议存放在/tmp下面，容易被误删除，因此放在var目录下

# 提取program.conf模板

从默认配置中抽取出常用的program模板

```ini
[program:concurrency_statistician]
directory = /root/qk_python/data/statistic/concurrency_statistician
command = python concurrency_statistician.py
autostart = true
startsecs = 5
autorestart = true
startretries = 3
user = root
```

如果单独调整program日志大小和个数，则加入stdout和stderr四个参数，新的配置文件如下

```ini
[program:concurrency_statistician]
directory = /root/qk_python/data/statistic/concurrency_statistician
command = python concurrency_statistician.py
autostart = true
startsecs = 5
autorestart = true
startretries = 3
user = root
stdout_logfile_maxbytes = 10MB
stdout_logfile_backups = 10
stderr_logfile_maxbytes = 10MB
stderr_logfile_backups = 10
```

一般情况下，不需要指定日志文件名称，默认的日志文件名称组成为

- 程序名称+日志类型（stdout和stderr）+ supervisor+进程号+日志编号

文件名称示例：concurrency_statistician-stderr---supervisor-OpZ5di.log.2

> 需要注意：supervisor服务端程序supervisord重新启动之后，会产生新的pid。因此日志只会在新的进程日志中产生，老的子进程日志不会被删除。因此如果需要重新启动supervisord，则需要注意是否要保留老进程日志，以免超过磁盘大小。
>
> 一般情况下，生产环境中不会经常重启supervisord

# 常用supervisorctl命令

可以进入 supervisorctl 的 shell 界面，也可以直接在 bash 终端运行

```shell
# 停止某一个进程，program_name 为 [program:x] 里的 x
supervisorctl stop program_name
# 启动某个进程
supervisorctl start program_name
# 重启某个进程
supervisorctl restart program_name
# 停止全部进程，注：start、restart、stop 都不会载入最新的配置文件
supervisorctl stop all
# 载入最新的配置文件，停止原有进程并按新的配置启动、管理所有进程
supervisorctl reload
# 根据最新的配置文件，启动新配置或有改动的进程，配置没有改动的进程不会受影响而重启
supervisorctl update
```

关于program groups的命令使用较少暂不介绍

---

参考资料：

1. [supervisor 官方文档](http://supervisord.org/)
2. [ 小强斋太-Supervisor简介、安装、配置](http://wangshengzhuang.com/2017/05/26/%E8%BF%90%E7%BB%B4%E7%9B%B8%E5%85%B3/Supervisor/Supervisor%E7%AE%80%E4%BB%8B/)
3. [使用supervisor管理进程](http://liyangliang.me/posts/2015/06/using-supervisor/)
4. [廖雪峰-Linux后台进程管理利器：supervisor](https://www.liaoxuefeng.com/article/0013738926914703df5e93589a14c19807f0e285194fe84000)
5. [乌龟运维-Supervisor 安装及配置](https://wuguiyunwei.com/index.php/2017/06/26/1028.html)
6. [oschina-SUPERVISOR进程管理器配置指南](https://my.oschina.net/crooner/blog/395069)