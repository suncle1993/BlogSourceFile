---
categories:
  - python
date: '2017-03-24T22:44:37'
description: ''
tags:
  - Python
  - 网络
  - socket
  - socketServer
title: Python网络编程
---



Python 提供了两个级别访问的网络服务。：

- 低级别的网络服务支持基本的 socket，，可以访问底层操作系统Socket接口的方法。
- 高级别的网络服务模块 socketserver， 可以简化网络服务器的开发。

# socket

查看socket类的帮助如下

```python
import socket  # 导入socket模块
>>> help(socket.socket)
```

重点关注初始化函数：

```python
__init__(self, family=<AddressFamily.AF_INET: 2>, type=<SocketKind.SOCK_STREAM: 1>, proto=0, fileno=None)
```

- family：网络协议簇，默认值为AF_INET
- type：套接字的类型，根据是面向连接的还是非连接分为`SOCK_STREAM`或`SOCK_DGRAM`
- proto：套接字协议，一般默认为0，表示
- fileno：套接字的int型的文件描述符

下面实现一个TCP聊天室和一个UDP聊天室

<!--more-->

## TCP聊天室

### 概要设计

**获取多个连接的处理**

开启accept线程，执行accept操作开始阻塞，有客户端连接时，再开启一个线程recv进行数据接收的处理。然后accept线程继续阻塞，等待后续客户端的连接。

**阻塞的处理**

服务端处理客户端的连接时，有两处存在阻塞，分别是：

- 获取连接时，socket.accept()会阻塞
- 每一个建立成功的连接在获取数据时，socket.recv(1024)

因此这两处都需要开启线程单独处理，否则会阻塞主线程。

**客户端主动断开的处理**

客户端主动断开时，如果不通知服务端，那么服务端上保存的客户端连接不会被清理，这是不合理的。因此客户端主动断开时，我们在应用层约定，客户端推出前需要发送`/quit`指令到服务端上，然后有服务端关闭socket。

### TCP聊天室-server

聊天室的server端主要是监听端口，处理来自client端的连接，并且分发数据到所有的client端

**代码**

```python
import socket
import threading


class TcpChatServer:
    def __init__(self, ip='192.168.110.13', port=9001):
        self.ip = ip
        self.port = port
        self.clients = {}
        self.sock = socket.socket(family=socket.AF_INET, type=socket.SOCK_STREAM)
        self.event = threading.Event()

    def recv(self, so, ip ,port):
        while not self.event.is_set():
            data = so.recv(1024).decode()  # 将接受到的字节数据bytes转化为utf-8格式的字符串
            if data.strip() == '/quit':  # 客户端主动断开时的处理
                so.close()
                self.clients.pop((ip, port))
                return 
            for s in self.clients.values():  # 广播发送
                s.send('{}:{}\n{}'.format(ip, port, data).encode())

    def accept(self):
        while not self.event.is_set():
            so, (ip, port) = self.sock.accept()
            self.clients[(ip, port)] = so
            # 因为so.recv会产生阻塞，因此单独开一个线程处理数据的接受部分。这样accept可以继续接受来自其他客户端的链接
            threading.Thread(target=self.recv, args=(so, ip, port), name='client-{}:{}'.format(ip, port)).start()

    def start(self):
        self.sock.bind((self.ip, self.port))
        self.sock.listen()
        t = threading.Thread(target=self.accept, daemon=True)  # 为了不阻塞主线程，单独开启一个线程处理accept（accept会阻塞线程）
        try:
            t.start()
            t.join()  # 阻塞直到获取到KeyboardInterrupt
        except KeyboardInterrupt:
            self.stop()

    def stop(self):
        for s in self.clients.values():
            s.close()
        self.sock.close()
        self.event.set()  # 停止所有的循环


if __name__ == '__main__':
    tcp_chat_server = TcpChatServer()
    tcp_chat_server.start()
```

### TCP聊天室-client

聊天室的client端主要是发起连接，连接到server端，并且要接受来自服务端广播分发的消息。

**代码**

```python
import socket
import threading


class TcpChatClient:
    def __init__(self):
        self.sock = socket.socket(family=socket.AF_INET, type=socket.SOCK_STREAM)
        self.event = threading.Event()

    def recv(self):  # 客户端需要一直接收服务端广播分发的消息
        while not self.event.is_set():
            data = self.sock.recv(1024).decode()
            data = data.strip()
            print(data)

    def send(self):  # 输入消息就发送
        while not self.event.is_set():
            data = input()
            self.sock.send(data.encode())
            if data.strip() == '/quit':  # 发送/quit的时候自身关闭
                self.stop()

    def start(self, ip, port):
        self.sock.connect((ip, port))
        s = threading.Thread(target=self.send, daemon=False)
        r = threading.Thread(target=self.recv, daemon=False)
        s.start()
        r.start()

    def stop(self):
        self.sock.close()
        self.event.set()


if __name__ == '__main__':
    tcp_chat_client = TcpChatClient()
    tcp_chat_client.start('192.168.110.13', 9001)
```

## UDP聊天室

### 概要设计

**阻塞的处理**

在UDP服务端接收客户端的消息时，采用`socket.recvfrom(1024)`这个方法以便保存客户端的地址信息，这个方法会阻塞当前线程，因此需要开启线程单独处理。

**客户端主动断开的处理**

UDP客户端主动关闭之后，服务端是无法检测到客户端已经关闭的。我们可以采用以下两种方法：

1. 如果类似于TCP采用约定退出指令的方法，那么客户端发送退出指令后就调用close方法，然后服务端根据得到的指令剔除客户端字典中对应的客户端。
2. 还可以通过客户端定时发送心跳给服务端，服务端通过心跳来判断客户端进程是否存活。

### UDP聊天室-server

UDP服务端程序开启线程等待接收客户端的数据，然后广播给其他的客户端，并且检查所有连接的心跳是否超时。

代码
```python
import socket
import datetime
import threading


class UdpChatServer:
    def __init__(self, ip='192.168.110.13', port=9001):
        self.addr = (ip, port)
        self.sock = socket.socket(family=socket.AF_INET, type=socket.SOCK_DGRAM)
        self.clients = {}
        self.event = threading.Event()
    
    def recv(self):
        while not self.event.is_set():
            data, addr = self.sock.recvfrom(1024)
            data = data.decode().strip()
            now = datetime.datetime.now()
            if data == '#ping#':  # 判断是否收到心跳
                self.clients[addr] = now  # 收到心跳则保存客户端地址，并且更新时间戳
                continue
                
            disconnected = set()  # 没收到一次数据就判断所有的失效链接
            for addr, timestamp in self.clients.items():
                if (now - timestamp).total_seconds() > 10:  # 失效条件：2次（即10s）没收到心跳就判断客户端关闭
                    disconnected.add(addr)
                else:
                    self.sock.sendto('{}:{}\n{}'.format(addr[0], addr[1], data).encode(), addr)
                    
            for addr in disconnected:
                self.clients.pop(addr)
            
    def start(self):
        self.sock.bind(self.addr)  # 绑定端口之后就开启线程一直接受客户端的数据
        t = threading.Thread(target=self.recv(), daemon=True)
        try:
            t.start()
            t.join()
        except KeyboardInterrupt:
            self.stop()
        
    def stop(self):
        self.event.set()
        self.sock.close()

if __name__ == '__main__':
    udp_chat_server = UdpChatServer()
    udp_chat_server.start()
```

### UDP聊天室-client

UDP的客户端的主线程一直在等待用户输入数据然后将数据发送到服务端，同时开启了一个心跳进程和一个接受服务端广播数据的线程。

**代码**
```python
import socket
import threading
import time


class UdpChatClient:
    def __init__(self, ip, port):
        self.addr = (ip, port)
        self.sock = socket.socket(family=socket.AF_INET, type=socket.SOCK_DGRAM)
        self.event = threading.Event()
    
    def heartbeat(self):  # 心跳线程函数：每5s发一次心跳
        while not self.event.wait(5):
            self.sock.sendto(b'#ping#', self.addr)
            
    def recv(self):  # 一直等待接受udp服务器广播的数据
        while not self.event.is_set():
            data = self.sock.recv(1024)
            print(data.decode())
    
    def start(self):
        threading.Thread(target=self.heartbeat, name='heartbeat', daemon=True).start()
        threading.Thread(target=self.recv, name='recv', daemon=True).start()
        print('请在5s后发言')
        time.sleep(5)  # 因为服务端必须收到一个心跳之后才会保存次客户端，因此需要等待5s
        print('请开始发言')
        while not self.event.is_set():
            data = input('')
            data = data.strip()
            if data == '/quit':
                self.event.set()
                self.sock.close()
                return
            self.sock.sendto(data.encode(), self.addr)

if __name__ == '__main__':
    udp_chat_client = UdpChatClient('192.168.110.13', 9001)
    udp_chat_client.start()
```

# SocketServer

> TODO(Flowsnow)：改写聊天室程序的TcpChatServer和UdpChatServer

---

**附一：TCP和UDP的本质区别**

- udp：所有的客户端发来的数据报都堆积在队列上，然后服务端一个一个的处理


- tcp：每一个客户端和服务端都有一个连接通道，只处理对应客户端的数据流

**附二：参考资料**

1. [socketserver — A framework for network servers](https://docs.python.org/3.5/library/socketserver.html)