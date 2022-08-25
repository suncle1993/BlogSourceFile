---
categories:
  - 协议
date: '2017-12-11T14:01:05'
description: ''
tags:
  - unix
  - socket
  - IPC
title: unix domain socket
---




# unix domain socket

unix domain socket 是在socket架构上发展起来的用于同一台主机的进程间通讯（IPC: Inter-Process Communication），它不需要经过网络协议栈，不需要打包拆包、计算校验和、维护序号和应答等，只是将应用层数据从一个进程拷贝到另一个进程。UNIX Domain Socket有SOCK_DGRAM或SOCK_STREAM两种工作模式，类似于UDP和TCP，但是面向消息的UNIX Domain Socket也是可靠的，消息既不会丢失也不会顺序错乱。

UNIX Domain Socket可用于两个没有亲缘关系的进程，是全双工的，是目前使用最广泛的IPC机制，比如X Window服务器和GUI程序之间就是通过UNIX Domain Socket通讯的。

UNIX Domain socket与网络socket类似，可以与网络socket对比应用。

上述二者编程的不同如下：

- address family为AF_UNIX
- 因为应用于IPC，所以UNIXDomain socket不需要IP和端口，取而代之的是文件路径来表示“网络地址”。这点体现在下面两个方面。
- 地址格式不同，UNIXDomain socket用结构体sockaddr_un表示，是一个socket类型的文件在文件系统中的路径，这个socket文件由bind()调用创建，如果调用bind()时该文件已存在，则bind()错误返回。
- UNIX Domain Socket客户端一般要显式调用bind函数，而不象网络socket一样依赖系统自动分配的地址。客户端bind的socket文件名可以包含客户端的pid，这样服务器就可以区分不同的客户端。

下面用python代码演示uds的使用

<!--more-->

# Python代码演示

服务端

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
Created on 12/11/17 11:55 AM
@author: Chen Liang
@function: socket_echo_server_uds
"""

import sys

reload(sys)
sys.setdefaultencoding('utf-8')
import socket
import os

server_address = './uds_socket'

# Make sure the socket does not already exist
try:
    os.unlink(server_address)
except OSError:
    if os.path.exists(server_address):
        raise

# Create a UDS socket
sock = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)

# Bind the socket to the address
print('starting up on {}'.format(server_address))
sock.bind(server_address)

# Listen for incoming connections
sock.listen(1)

while True:
    # Wait for a connection
    print('waiting for a connection')
    connection, client_address = sock.accept()
    try:
        print('connection from', client_address)

        # Receive the data in small chunks and retransmit it
        while True:
            data = connection.recv(16)
            print('received {!r}'.format(data))
            if data:
                print('sending data back to the client')
                connection.sendall(data)
            else:
                print('no data from', client_address)
                break

    finally:
        # Clean up the connection
        connection.close()

```

客户端

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
Created on 12/11/17 11:55 AM
@author: Chen Liang
@function: socket_echo_client_uds
"""

import sys

reload(sys)
sys.setdefaultencoding('utf-8')


import socket
import sys

# Create a UDS socket
sock = socket.socket(family=socket.AF_UNIX, type=socket.SOCK_STREAM)

# Connect the socket to the port where the server is listening
server_address = './uds_socket'
print('connecting to {}'.format(server_address))
try:
    sock.connect(server_address)
except socket.error as msg:
    print(msg)
    sys.exit(1)

try:

    # Send data
    message = b'This is the message.  It will be repeated.'
    print('sending {!r}'.format(message))
    sock.sendall(message)

    amount_received = 0
    amount_expected = len(message)

    while amount_received < amount_expected:
        data = sock.recv(16)
        amount_received += len(data)
        print('received {!r}'.format(data))

finally:
    print('closing socket')
    sock.close()

```

客户端一次发送，服务端分批返回。

服务端输出结果如下

```
root@ubuntu:~/PycharmProjects/python_scripts# python socket_echo_server_uds.py 
starting up on ./uds_socket
waiting for a connection
('connection from', '')
received 'This is the mess'
sending data back to the client
received 'age.  It will be'
sending data back to the client
received ' repeated.'
sending data back to the client
received ''
('no data from', '')
waiting for a connection
```

客户端输出结果如下

```
root@ubuntu:~/PycharmProjects/python_scripts# python socket_echo_client_uds.py 
connecting to ./uds_socket
sending 'This is the message.  It will be repeated.'
received 'This is the mess'
received 'age.  It will be'
received ' repeated.'
closing socket
```

查看套接字文件的类型如下

```
root@ubuntu:~/PycharmProjects/python_scripts# ls -l ./uds_socket
srwxr-xr-x 1 root root 0 Dec 11 13:45 ./uds_socket
```

可见**uds文件是socket类型**。具体的linux文件类型有以下几种：

Linux的文件类型有以下几种:

| 文件类型 | `ls -l`显示 |
| ---- | --------- |
| 普通文件 | `-`       |
| 目录   | `d`       |
| 符号链接 | `l`       |
| 字符设备 | `c`       |
| 块设备  | `b`       |
| 套接字  | `s`       |
| 命名管道 | `p`       |

---

**参考：**

- [Python实例浅谈之九使用本地socket文件](https://blog.csdn.net/taiyang1987912/article/details/46774319)
- [Linux下的IPC－UNIX Domain Socket](https://blog.csdn.net/guxch/article/details/7041052)
- [pymotw3 unix domain socket](https://pymotw.com/3/socket/uds.html)