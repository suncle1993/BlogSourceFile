---
categories:
  - 协议
date: '2018-03-09T13:52:45'
description: ''
tags:
  - 规范
  - rtmp
title: rtmp规范1.0
---



![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/rtmp_illustration.jpg)

RTMP（real time messaging protocol）协议

本文为Adobe rtmp规范1.0的中文介绍，其中内容大部分都是翻译自rtmp官方文档[rtmp_specification_1.0.pdf](http://wwwimages.adobe.com/www.adobe.com/content/dam/acom/en/devnet/rtmp/pdf/rtmp_specification_1.0.pdf)

> 具体文章目录参见文章内侧边栏

<!--more-->

# 介绍

Adobe的实时消息传输协议（`RTMP`）通过可靠的流传输（如`TCP [RFC0793]`）提供双向消息多路传输服务，用于在端到端之间传输带有时序信息的视频，音频和数据消息的并行流。 穿过多层流，`RTMP`消息块流不提供任何控制的优先级别和相似形式，但是可以用于高层协议提供这样的优先级，例如：一段实时视频服务会选择丢弃给缓慢的客户的视频信息确保音频信息可以及时被接收。`RTMP消息块流`包含它自己的入队协议控制消息，也提供一个高层协议机制用于嵌入用户的控制消息。

# 定义

**有效负载：Payload**

包含在包中的数据，就像音频样本或者压缩的视频数据。

**包：Packet**

一个数据包由固定的包头和有效负载数据组成，一些底层协议或许需要包的封装来被定义。

**端口：Port**

在`TCP/IP`协议中定义的用正整数表示的端口号用于在传输中提取以区分目标主机的不同应用，用于`OSI`传输层的传输选择（`TSEL`）就是端口。

**传输地址：Transport address**

网络地址和端口的组合识别一个传输层终端端口，例如一个IP地址和TCP端口，数据包从一个源传输层地址传送到目标段的传输层地址。

**消息流：Message stream**

一个通信的逻辑通道，允许消息流通。

**消息流ID：Message stream ID**

每一个消息拥有一个分配的ID识别跟随的消息流。

**消息块：Chunk**

消息的片段，消息被分成小的部分，在他们在网络中发送之前交叉存储。消息块确保定制时间戳的端到端全消息传送，穿过多层流。

**消息块流：Chunk stream**

一个通信的逻辑通道，允许消息块在一个特定的方向上流通，消息块流可以从客户端传送到服务器，也可以相反。

**消息块流ID：Chunk stream ID**

每一个消息块有一个分配的ID用于识别更随的消息块流。

**复合技术：Multiplexing**

把分开的音视频数据组合成一条音视频流的过程，使同时传送许多音视频数据成为可能。

**逆复合技术：DeMultiplexing**

复合的反向过程，交叉存取组装的音频视频数据，使他们成为最初的音视频数据

**远程过程调用：Remote Procedure Call (RPC)**

允许客户端或服务器在对等端调用子例程或过程的请求。

**Action Message Format (AMF)**

一种紧凑的二进制格式，用于序列化`ActionScript object graphs`。 可以透过 `AMF overHTTP`的方式将`flash`端资料编码后传回server，server端的`remoting adaptor`接收到资料后则会译码回正确的`native`对象，交给正确的程序处理。

# 字节顺序，对齐和时间格式

所有的整数字段都被引入到了字节顺序当中，字节0是第一个显示出来的，也是一个词和一个字段中最重要的。这种顺序就是通常所说的“大端”。如果没有特殊说明，在本文档中数字常量都是用十进制表示。

除另有规定外，`RTMP`中的所有数据都是字节对齐的。例如，一个16位字段可能处于奇数字节偏移处。 在指定填充的地方，填充字节应该是0。

`RTMP`中的时间戳相对于未指定的时期是以整数毫秒为单位给出的。 通常，每个流将以时间戳0开始，但这不是必需的，只要两个终端在时间点上达成一致。 请注意，这意味着跨多个流（尤其是来自不同主机）的任何同步都需要一些`RTMP`外的其他机制。

时间戳必须始终在线性的增加，允许应用程序处理异步传输，带宽度量，检测，和流控制。

由于时间戳长度为32位，因此它们每隔49天，17小时，2分钟，47.296秒滚动一次。 由于流可以连续运行，可能持续数年，`RTMP`应用程序应该在处理时间戳时使用序列号算法`[RFC1982]`，并且应该能够处理回绕。 例如，假定所有相邻的时间戳都在`2^31 - 1`毫秒之间，所以10000会在4000000000之后，而3000000000会在4000000000之前。

时间戳增量delta也被指定为相对于先前时间戳的无符号整数毫秒数。 时间戳增量delta可以是24位或32位。

# RTMP块流

本节介绍实时消息传送协议块流（`RTMP块流`）。 它为更高级别的多媒体流协议提供复用和打包服务。 虽然`RTMP Chunk Stream`旨在与实时消息传送协议配合使用，但它可以处理发送消息流的任何协议。 每条消息都包含时间戳和有效负载类型标识。 `RTMP Chunk Stream`和`RTMP`一起适用于各种音频 - 视频应用，从一对一和一对多实时广播到视频点播服务，再到交互式会议应用。

当与可靠的传输协议（如`TCP [RFC0793]`）一起使用时，`RTMP块流`提供了保证所有消息在多个流中按时间排序的端到端传送。 `RTMP块流`不提供任何优先级或类似的控制形式，但可以由更高级别的协议提供这种优先级。

## 消息格式

可以拆分成块以支持复用的消息格式取决于更高级别的协议。 但是，消息格式应该包含下列创建块所必需的字段。

**时间戳：**

消息的时间戳，这个字段可以传输4个字节。

**长度：**

消息的有效负载的长度，如果消息头不能被省略，它应该包含在长度中，这个字段在消息块包头中占有3个字节。

**类型ID：**

协议控制消息的类型字段的范围是被保留的，这些传播信息的消息由`RTMP消息块`和高层协议处理，所有其他的类型ID可被高层协议使用，对`RTMP消息块`来说当做不透明的值，实际上，`RTMP Chunk Stream`中的任何内容都不需要将这些值用作类型; 所有（非协议）消息可以是相同类型的，或者应用程序可以使用类型id来区分同步踪迹而不是类型。 该字段占用块头中的1个字节。

**消息流ID:**

消息流ID可以是任意的值。 复合到相同块流上的不同消息流可以基于它们的消息流ID进行逆复合操作。 除此之外，就`RTMP`块流而言，这是一个不透明的值。 该字段以小尾数格式占用块头中的4个字节。

## 握手

`RTMP`连接始于握手。 `rtmp`握手与其他协议的握手不同; 它由三个相同大小的块组成，而不是由可变大小的块组成。

客户端（连接已初始化的终端）和服务器都发送相同的三个块。 为了说明，由客户端发送的3个块分别为`C0`, `C1`, `C2`，由服务端发送的3个块分别为`S0`, `S1`, `S2`。

### 握手的顺序

握手以客户端发送`C0`和`C1`消息块位开始，客户端必须等到`S1`到达在发送`C2`。客户端必须等到`S2`接收到才可以发送其他的数据；服务端必须等到`C0`到达才发送`S0`和`S1`，在`C1`之后也会等待。服务端必须等到`C1`到达才发送`S2`，服务端必须等到`C2`到达后才发送其他数据。

### C0和S0格式

`C0`和`S0`都是单个8位字节，可以看成一个8位整形字段。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/C0_and_S0_bits.jpg)

8比特版本：在C0中，这个字段识别客户端需求的RTMP的版本，在S0中，这个字段识别服务器端选择的RTMP的版本，被定义的是版本3，0到2是早前的版本使用的，4到31保留用于未来使用，32到255还没有被允许。不能区分客户的请求的版本的服务应该以3返回，客户端可以选择降级到版本3，或放弃握手。

### C1和S1格式

`C1`和`S1`包长度为1536个8位字节，包含以下字段：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/C1_and_S1_bits.jpg)

**time（4个字节）**：这个字段包含时间戳，被当做后续消息块从终端发送的时间点，也许是0，或者一些任意的值。为了同步多路消息块流，终端或许希望发送其他消息块流的时间戳的当前值。

**zero（4各个字节）**：这个字段必须全0。

**random data（1528个字节）**：这个字段可以包含任何任意的值，因为每个终端必须区分自己初始化的握手的返回数据和对方初始化的握手的返回数据，这个数据应该发送一些随机数。但是没有必要用密码保护随机数和动态值。

### C2和S2格式

`C2`和`S2`包长度为1536个8位字节，分别类似于`S1`和`C1`的原样返回，由一下几个字段组成：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/C2_and_S2_bits.jpg)

**time（4个字节）**：

这个字段必须包含由对端发送的`S1`（对应`C2`）或者`C1`（对应`S2`）的时间戳.

**time2（4个字节）**：

这个字段必须包含先前的由对端发送的数据包（`S1`或者`C1`）被读取的时间戳。

**random echo（1528个字节）**：

这个字段必须包含在对端发送的`S1`（对应`C2`）或`S2`（对应`C1`）数据包中的随机数据字段。 任何一方都可以使用`time`和`time2`字段与当前时间戳一起快速估算连接的带宽和/或延迟，但这不太可能有用。

### 握手过程图

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Pictorial_Representation_of_Handshake.jpg)

下面的表格描述了握手过程的几个阶段

| 阶段      | 描述                                       |
| ------- | ---------------------------------------- |
| 未初始化    | 协议版本在此阶段发送。 客户端和服务器都未初始化。 客户端发送数据包`C0`中的协议版本。 如果服务器支持该版本，则发送`S0`和`S1`作为响应。 否则，服务器将采取适当的措施进行响应。 在`RTMP`中，此操作正在终止连接。 |
| 版本发送    | 在未初始化状态之后，客户端和服务器都处于版本已发送状态。 客户端正在等待数据包`S1`，服务器正在等待数据包`C1`。 在接收到等待的数据包时，客户端发送`C2`并且服务器发送`S2`。 状态然后变成`Ack`发送。 |
| `ACK`发送 | 客户端和服务端分别等待`S2`和`C2`。                    |
| 握手完成    | 客户端和服务交换消息。                              |

## 消息分块

握手后，连接复用一个或多个消息块流。每个块流从一个消息流携带一种类型的消息。每个创建的块都有一个与其关联的唯一ID，称为块流ID。这些块通过网络传输。发送时，每个块必须在下一个块之前全部发送。在接收端，根据块流ID将块组合成消息。

分块允许将较高级别协议中的大的型消息分解为较小的消息，例如防止较大的低优先级消息（例如视频）阻塞较小的高优先级消息（如音频或控制）。

分块还允许以较少的开销发送小消息，因为分块头包含信息的压缩表示信息，这些压缩消息本来应该包含在消息本身的。

块大小是可配置的。它可以使用`Set Chunk Size`控制消息进行设置。

### 消息块格式

每一个消息块有头部和数据组成，头部自身可以被分割成三个部分：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Chunk_Header.jpg)

**消息块基本头（1到3个字节）**：这个字段编码了消息块流的ID和消息块的类型，消息块类型决定了消息包头的编码格式，长度完全取决于可变长的消息块流ID。

**消息块消息头（0,3,7或11字节）**：这个字段编码正在传送的消息的信息，长度可以利用在消息块头中详细的消息块类型来决定。

**扩展时间戳（0或4字节）**：此字段在某些情况下是存在的，取决于消息块消息头中的编码时间戳或时间戳增量字段。

**消息块块数据（可变大小）**：该块的有效负载，直至配置的最大块大小。

####  消息块基本头

消息块基本头对消息块流的ID和消息块的类型进行编码（在下面的图表中用`fmt`表示），消息块类型决定了编码的消息头的格式，消息块基本头字段可以是1,2或者3个字节长，取决于消息块流ID。

该协议支持多达65597个ID为3-65599的流。 ID0,1和2被保留。 值0指示2字节形式和64-319范围内的ID（`the second byte + 64`）。 值1表示3字节形式，ID在64-65599（`(the third byte) * 256 + the second byte + 64`）范围内。 在3-63范围内的值表示完整的流ID。 块ID为2的流ID保留，用于低级别的协议控制消息和命令。

在消息块基本头中0-5比特(最不重要的)代表了消息块流ID。

消息块流ID 2-63 可以被编码成这个字段的单字节的版本号。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Chunk_basic_header_1.jpg)

块流ID 64-319可以以2字节的形式被编码。 ID计算为（第二个字节+ 64）。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Chunk_basic_header_2.jpg)

可以在此字段的3字节版本中对块流ID 64-65599进行编码。 ID计算为（（第三字节）* 256 +（第二字节）+64）。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Chunk_basic_header_3.jpg)

**cs id（6比特）**：这个字段包含了消息块流ID，值从2到63，值0和1用于代表这个字段的2个或者3个字节的版本号。

**fmt（2比特）**：这个字段标识消息块消息头使用的四种格式之一。见下一小节

**cs id -64（8或者16个比特）**：

这个字段包含了消息块流ID减64，例如ID 365在`cs id`段用1表示，在16比特的`cs id -64`段用301表示。

值为64到319的消息块流ID可以被2字节或者3字节的版本号来表示。

#### 消息块消息头

在消息块消息头中有四种不同的格式，由消息块基本头的`fmt`字段选择。应该使用最简洁的表达方式表示每一个消息块消息头。

##### 类型0

类型0的消息块有11个字节长，这个类型必须在消息块流开始时和消息流的时间戳回溯时使用

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Chunk_Message_Header_Type_0.jpg)

**时间戳（3个字节）**：对于类型0的块，消息的绝对时间戳发送到此处。 如果时间戳大于或等于16777215（十六进制`0xFFFFFF`），则该字段必须是16777215，表示存在扩展时间戳字段以编码完整的32位时间戳。 否则，这个字段应该是整个时间戳。

##### 类型1

类型1的消息块有7个字节长，消息流ID没有被包含，这个消息块得到和先前消息块同样的流ID，带有可变长的消息的流（例如许多视频格式）在类型0消息块后应该使用这种格式作为每一个消息的第一个消息块。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Chunk_Message_Header_Type_1.jpg)

##### 类型2

类型2块头长度为3个字节。 流ID和消息长度都不包含在内; 该块与前面的块具有相同的流ID和消息长度。 具有固定大小消息的流（例如，某些音频和数据格式）应该在第一个消息之后使用这种格式作为每个消息的第一个块。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Chunk_Message_Header_Type_2.jpg)

##### 类型3

类型3 的消息块没有头，流ID，消息长度和时间戳delta，这个类型的消息块在之前的消息块中取值，当单一的消息被分裂成消息块，所有的消息块除了第一个，其余都应该使用这种类型，流由同样大小的消息组成。

##### 公共头字段

块消息头中每个字段的描述：

| 字段                          | 描述                                       |
| --------------------------- | ---------------------------------------- |
| timestamp delta (3 bytes)   | 对于类型1或类型2块，此处可以看到发送前一个块的时间戳和当前块的时间戳之间的差异。 如果增量大于或等于16777215（十六进制`0xFFFFFF`），则该字段必须是16777215，表示扩展时间戳字段以编码完整的32位增量的形式存在。 否则，这个字段应该是实际的增量。 |
| message length (3 bytes)    | 对于类型0和类型1的消息块，消息的长度会出现在这里。注意这一般和消息块有效负载长度是不一样的。消息块的有效负载长度是除了最后一个消息块的所有消息块的最大的长度。 |
| message type id (1 byte)    | 对于类型0和类型1的消息块，消息的类型出现在这里。                |
| message stream id (4 bytes) | 对于类型为0的块，存储消息流ID。 消息流ID以`little-endian`格式存储。 通常，同一块流中的所有消息将来自同一个消息流。 虽然可以将单独的消息流多路复用到同一个块流中，但这会破坏头压缩的优点。 但是，如果一个消息流关闭并且另一个随后打开，则没有理由通过发送新的类型为0的块来重新使用现有的块流。 |

#### 扩展时间戳

`Extended Timestamp`字段用于编码大于16777215（`0xFFFFFF`）的时间戳或时间戳增量; 也就是说，对于时间戳或时间戳增量，它们不适合类型0,1或2块的24位字段。 该字段对完整的32位时间戳或时间戳增量进行编码。 这个字段用于表示将类型0块的时间戳字段或类型1或2块的时间戳增量字段设置为16777215（`0xFFFFFF`）。 当相同块流ID的最新类型0,1或2的块指示存在扩展时间戳字段时，该字段出现在类型3的块中。

### 示例

共有2个示例

#### 示例1

本例给出了一个简单的音频消息流，这个例子示范了信息的冗余。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Sample_audio_messages_to_be_made_into_chunks.jpg)

下表显示了在此流中生成的块。 从消息3开始，数据传输得到优化。 除此之外，每消息只有1字节的开销。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Format_of_each_of_the_chunks_of_audio_messages.jpg)

####  示例2

本例说明一个很长的消息被分割成很多消息块。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Sample_Message_to_be_broken_to_chunks.jpg)

这里是分割出来的消息块

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Format_of_each_of_the_chunks.jpg)

消息块1的包头数据详细介绍了307个字节的消息的全部。

注意这两个例子，类型3消息块可以用作两种不同的方式，第一种是表示一条消息的延续，第二种是表示一条新消息的开始，这个新消息可以从已经存在的数据中衍生出来。

## 协议控制消息

`RTMP`块流使用消息类型ID 1,2,3,5和6作为协议控制消息。 这些消息包含`RTMP Chunk Stream`协议所需的信息。

这些协议控制消息务必具有消息流ID 0 （称为控制流）并且以块流ID 2 发送。协议控制消息一旦被接收就会立即生效，同时时间戳被忽略。

### 设置消息块大小

协议控制消息1：设置消息块大小。用来通知对方新的最大的消息块大小。

消息块的大小可以被设置成一个默认的值，128字节，但是客户端或者服务端可以改变这个值，并且发送消息通知对方更新。例如：假设一个客户端想要发送131字节的音频数据，消息块的大小为128字节，在这种情况下，客户端可以发送这个协议控制消息给服务端以通知消息块的大小被设置成了131字节，那么客户端就可以用一个消息块发送音频数据。

最大块大小应该不能小于128个字节，并且必须不能小于1个字节。 每个方向的最大块大小都是独立维护的。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Payload_for_the_Set_Chunk_Size_protocol_message.jpg)

**0**： 这一位必须为0。

**chunk size 块大小（31位）**：该字段保存新的最大块大小（以字节为单位），这将用于发件人的所有后续块，直至另行通知。 有效大小为1到2147483647（`0x7FFFFFFF`）（含）; 但是，大于16777215（`0xFFFFFF`）的所有大小都是等效的，因为没有块大于一条消息，并且没有消息大于16777215字节。

### 中断消息

协议控制消息2：中止消息。用于通知对方是否正在等待块完成消息，然后丢弃部分接收到的消息。 对方接收块流ID作为该协议消息的有效载荷。 应用程序可能会在关闭时发送此消息，以指示不需要进一步处理消息。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Payload_for_the_Abort_Message_protocol_message.jpg)

**`chunk stream ID` 块流ID （32 位）**：  该字段保存块流ID，对应的当前消息将被丢弃。

### Acknowledgement

客户端或服务器在收到等于窗口大小的字节后，必须向对端发送`Acknowledgement`确认。 窗口大小是发送方未收到接收方确认而发送的最大字节数。 该消息指定了序列号，它是到当前为止收到的字节数。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Payload_for_the_Acknowledgement_protocol_message.jpg)

**`sequence number` 序列号（32 位）**：字段表示到当前为止收到的字节数。

### Window Acknowledgement Size

客户端或服务器发送此消息以通知对方在发送`Acknowledgement` 确认之间使用的窗口大小。 发送人希望在发送窗口大小字节后得到对方的确认。 

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Payload_for_the_Window_Acknowledgement_Size_protocol_message.jpg)

### 设置对等带宽

客户端或服务器发送此消息来限制另一方的输出带宽。 收到此消息的另一方通过将已发送但未确认的数据量限制为此消息中指示的窗口大小这种方式用来限制其输出带宽。如果窗口大小与发送给此消息发送者的最后一个窗口大小不同，那么接收此消息的另一方应该使用`"Window Acknowledgement Size"`消息进行响应。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Payload_for_the_Set_Peer_Bandwidth_protocol_message.jpg)

限制类型`Limit Type`是以下值之一：

- 0 - Hard：对方应该限制其输出带宽到指定的窗口大小。
- 1 - Soft：对方应该限制其输出带宽到本消息中指示的窗口或已经生效的限制，以较小者为准。
- 2 - Dynamic：如果以前的限制类型是Hard，请将此消息视为Hard类型消息，否则可以直接忽视。

# RTMP消息格式

本部分主要介绍`RTMP`消息的格式，在网络实体之间使用较低级传输层（如`RTMP块流`）传输这些消息。

虽然`RTMP`旨在与`RTMP块流`一起使用，但它可以使用任何其他传输协议发送消息。 `RTMP Chunk Stream`和`RTMP`一起适用于各种音视频应用，从一对一和一对多实时广播到视频点播服务，再到交互式会议应用。

## RTMP消息格式

服务器和客户端通过网络发送`RTMP`消息以相互通信。 消息可能包括音频，视频，数据或任何其他消息。

`RTMP`消息有两部分，头部和有效负载。

### 消息头

消息头包含以下字段：

| 字段                | 描述                                       |
| ----------------- | ---------------------------------------- |
| Message Type      | 一个字节的字段来表示消息类型。 一系列的类型ID（1-6）被保留用于协议控制消息 |
| Length            | 三字节字段，以字节表示有效负载的大小。 它以`big-endian`格式设置。  |
| Timestamp         | 包含消息时间戳的四字节字段。 这4个字节按`big-endian`顺序排列。   |
| Message Stream Id | 标识消息流的三字节字段。 这些字节以`big-endian`格式设置。      |

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Message_Header.jpg)

### 消息有效负载

消息的另一部分是有效负载，它是消息中包含的实际数据。 例如，它可能是一些音频样本或压缩的视频数据。

## 用户控制消息

RTMP使用消息类型ID 4 作为用户控制消息。 这些消息包含RTMP流层使用的信息。 带有ID 1,2,3,5和6的协议消息由RTMP块流协议使用。

用户控制消息应该使用消息流ID 0（称为控制流），并且当通过RTMP块流发送时，在消息流ID 2上发送。用户控制消息在流中被接收时生效， 他们的时间戳被忽略。

客户端或服务器发送此消息以通知对端用户控制事件。 该消息携带事件类型和事件数据。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Payload_for_the_User_Control_protocol_message.jpg)

消息数据`Event Data`的前2个字节用于标识事件类型`Event Type`。 事件类型后面跟着事件数据。 事件数据字段的大小是可变的。 但是，在消息必须通过RTMP块流层的情况下，最大块的大小应该足够大，以允许这些消息适合单个块。

# RTMP命令消息

本节介绍在服务器和客户端之间用于相互通信的不同类型的消息和命令。

在服务器和客户端之间交换的不同类型的消息包括用于发送音频数据的音频消息，用于发送视频数据的视频消息，用于发送任何用户数据的数据消息，共享对象消息和命令消息。 共享对象消息提供了一种通用的方式来管理多个客户端和服务器之间的分布式数据。 命令消息在客户端和服务器之间传送`AMF`编码的命令。 客户端或服务器可以通过流使用命令消息请求对方的远程过程调用（`RPC`）。

## 消息类型

服务器和客户端通过网络发送消息以相互通信。 消息可以是任何类型，包括音频消息，视频消息，命令消息，共享对象消息，数据消息和用户控制消息。

### 命令消息

命令消息在客户端和服务器之间传送`AMF`编码命令。 这些消息的`AMF0`编码的消息类型值为20，`AMF3`编码的消息类型值为17。 这些消息被发送来执行一些操作，例如`connect`，`createStream`，`publish`， `play`， `pause`等。 诸如`onstatus`，`result`等命令消息用于通知发送者有关请求的命令的状态。 命令消息由命令名称，事务ID和包含相关参数的命令对象组成。 客户端或服务器可以通过流使用命令消息请求对方的远程过程调用（`RPC`）。

### 数据消息

客户端或服务器发送此消息用于向对方发送元数据或任何用户数据。 元数据包括有关数据（音频，视频等）的详细信息，如创建时间，持续时间，主题等。 `AMF0`的消息类型值为18，`AMF3`的消息类型值为15。

### 共享对象消息

共享对象是一个Flash对象（name-value对的集合），在多个客户端，实例等之间同步的。 `AMF0`的消息类型19和`AMF3`的消息类型16保留用于共享对象事件。 每条消息可以包含多个事件。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/The_shared_object_message_format.jpg)

支持以下事件类型：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/supported_event_type.jpg)

### 音频消息

客户端或服务器发送此消息来向对等方发送音频数据。 消息类型值8保留给音频消息。

### 视频消息

客户端或服务器发送此消息以向对等方发送视频数据。 消息类型值9保留给视频消息。

### 聚合消息

聚合消息是单个消息。消息类型22用于聚合消息。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/The_Aggregate_Message_format.jpg)

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/The_Aggregate_Message_body_format.jpg)

聚合消息的消息流ID会覆盖聚合内的子消息的消息流ID。

聚合消息的时间戳与第一个子消息之间的差异是用于将子消息的时间戳重新归一化为流时间尺度的偏移量。 将偏移量添加到每个子消息的时间戳以达到标准化的流时间。 第一个子消息的时间戳应该与聚合消息的时间戳相同，所以偏移量应该为零。

后向指针包含前一个消息的大小，包括其头部。 它被包含来匹配`FLV`文件的格式并用于向后搜索。

使用聚合消息有几个性能优势：

- 消息块流最多可以在块内发送单个完整的消息。 因此，增加块大小并使用聚合消息可减少发送的块的数量。
- 子消息可以连续地存储在内存中。 在进行系统调用向网络上发送数据时效率更高。

### 用户控制消息事件

客户端或服务器发送此消息以通知对端关于用户控制事件。

支持以下用户控制事件类型：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/supported_user_control_event_types.jpg)

## 命令类型

客户端和服务器交换`AMF`编码的命令。发送方发送一条命令消息，其中包含命令名称，事务ID和包含相关参数的命令对象。例如，`connect`命令包含`'app'`参数，它告诉客户端连接到的服务器应用程序名称。接收方处理该命令并以相同的事务ID发送响应。响应字符串可以是`_result`，`_error`或方法名称，例如`verifyClient`或`contactExternalServer`。

`_result`或`_error`命令字符串表示响应。事务ID指示响应引用的未完成的命令。它与`IMAP`和许多其他协议中的标签相同。命令字符串中的方法名称指示发送方正试图在接收方端运行方法。

以下类对象用于发送各种命令：

- `NetConnection`：一个对象，它是服务器和客户端之间连接的更高级别表示。
- `NetStream`：表示发送音频流，视频流和其他相关数据的通道的对象。我们还发送控制数据流的播放，暂停等命令。

### NetConnection命令

`NetConnection`管理客户端应用程序和服务器之间的双向连接。 另外，它为异步远程方法调用提供支持。

以下命令可以在`NetConnection`上发送：

- `connect`
- `call`
- `close`
- `createStream`

#### connect

客户端向服务端发送连接（`connect`）命令请求连接一个服务器应用实例。以下为命令的结构：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/connect_command_structure.jpg)

以下是连接命令的命令对象中使用的`name-value`对的描述：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/description_of_the_name-value_pairs_used_in_Command_Object_of_the_connect_command.jpg)

`audioCodecs`属性的标志值：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Flag_values_for_the_audioCodecs_property.jpg)

`videoCodecs`属性的标志值：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Flag_values_for_the_videoCodecs_Property.jpg)

`videoFunction`属性的标志值：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Flag_values_for_the_videoFunction_property.jpg)

对象编码(`object Encoding`)属性的值：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Values_for_the_object_encoding_property.jpg)

以下是服务端到客户端命令的结构：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/The_command_structure_of_connect_command_from_server_to_client.jpg)

以下是连接命令中的消息流：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Message_flow_in_the_connect_command.jpg)

命令执行期间的消息流是：

1. 客户端将连接命令发送到服务器以请求连接服务器应用程序实例。
2. 接收到连接命令后，服务器将协议消息`'Window Acknowledgement Size'`发送给客户端。 服务器还连接到connect命令中提到的应用程序。
3. 服务器将协议消息 `'Set Peer Bandwidth'` 发送给客户端。
4. 在处理协议消息`'Set Peer Bandwidth'`后，客户端向服务器发送协议消息`'Window Acknowledgement Size'`。
5. 服务器将类型为`User Control Message(StreamBegin)`的另一个协议消息发送给客户端。
6. 服务器发送结果命令消息，通知客户端连接状态（成功/失败）。 该命令指定事务ID（连接命令始终等于1）。 该消息还指定了属性，例如`Flash Media Server`版本（字符串）。 此外，它还指定其他连接响应相关信息，如级别（字符串），代码（字符串），描述（字符串），对象编码（数字）等。

#### Call

`NetConnection`对象的调用方法在接收端运行远程过程调用（`RPC`）。 被调用的`RPC`名称作为参数传递给`call`命令。

从发送方到接收方的命令结构如下：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/The_command_structure_from_the_sender_to_the_receiver.jpg)

响应的命令结构如下：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/The_command_structure_of_the_response.jpg)

#### createStream

客户端将此命令发送到服务器以创建用于消息通信的逻辑通道。音频，视频和元数据的发布是通过使用`createStream`命令创建的流通道执行的。

`NetConnection`是默认通信通道，其流ID为0。协议和一些命令消息（包括`createStream`）使用默认通信通道。

从客户端到服务器的命令结构如下所示：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/The_command_structure_of_createStream_command_from_the_client_to_the_server.jpg)

从服务器到客户端的命令结构如下：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/The_command_structure_of_createStream_command_from_the_server_to_the_client.jpg)

### NetStream命令

`NetStream`定义了流式音频，视频和数据消息可以通过将客户端连接到服务器的`NetConnection`流动的通道。 一个`NetConnection`对象可以为多个数据流支持多个`NetStream`。

以下命令可以由客户端在`NetStream`上发送到服务器：

- `play`
- `play2`
- `deleteStream`
- `closeStream`
- `receiveAudio`
- `receiveVideo`
- `publish`
- `seek`
- `pause`

服务器使用`'onStatus'`命令将`NetStream`状态更新发送到客户端：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Format_of_NetStream_status_message_commands.jpg)

#### play

客户端将此命令发送到服务器以播放流。 播放列表也可以使用此命令多次创建。

如果您想要创建一个可在不同直播流或录像流之间切换的动态播放列表，请多次调用`play`，每次给`reset`传递`false`。相反，如果要立即播放指定的数据流，请清空播放队列中的其他流，给`reset`传递`true`。

从客户端到服务器的命令结构如下所示：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/The_command_structure_of_play_command_from_the_client_to_the_server.jpg)

`Play`命令中的消息流：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Message_flow_in_the_play_command.jpg)

命令执行期间的消息流是：

1. 在客户端收到服务器返回的`createStream`命令的成功结果后，客户端就开始发送`play`命令。
2. 在收到`play`命令后，服务器发送协议消息来设置块大小。
3. 服务器发送另一个协议消息（用户控制），用于指定事件`'StreamIsRecorded'`的和该消息中的流ID。 该消息在前2个字节中携带事件类型，在最后4个字节中携带流ID。
4. 服务器发送另一个指定事件`'StreamBegin'`的协议消息（用户控制），以指示流传输到客户端的开始。
5. 如果客户端发送的播放命令成功了，则服务器发送`onStatus`命令消息`NetStream.Play.Start`和`NetStream.Play.Reset`。 `NetStream.Play.Reset`只有在客户端发送的播放命令设置了重置标志时才由服务器发送。 如果没有找到要播放的流，则服务器发送`onStatus`消息`NetStream.Play.StreamNotFound`。
6. 在此之后，服务器发送客户端播放所需的音频和视频数据。

#### play2

与播放命令不同，`play2`可以切换到不同的比特率流，而不改变播放内容的时间线。 服务器维护多个文件，用于支持客户端在`play2`中请求的所有比特率。

从客户端到服务器的命令结构如下所示：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/The_command_structure_of_play2_command_from_the_client_to_the_server.jpg)

在`ActionScript 3`语言参考[`AS3`]中描述了`NetStreamPlayOptions`对象的公共属性。

下图显示了该命令的消息流：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Message_flow_in_the_play2_command.jpg)

#### deleteStream

当`NetStream`对象被破坏时，`NetStream`发送`deleteStream`命令。

从客户端到服务器的命令结构如下所示：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/The_command_structure_of_deleteStream_command_from_the_client_to_the server.jpg)

服务器不发送任何响应。

#### receiveAudio

`NetStream`发送`receiveAudio`消息以通知服务器是否将音频发送给客户端。

从客户端到服务器的命令结构如下所示：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/The_command_structure_of_receiveAudio_command_from_the_client_to_the_server.jpg)

如果在将`Bool Flag`设置为false的情况下发送`receiveAudio`命令，则服务器不会发送任何响应。 如果`Bool Flag`设置为true，则服务器会使用状态消息`NetStream.Seek.Notify`和`NetStream.Play.Start`作为响应。

#### receiveVideo

`NetStream`发送`receiveVideo`消息以通知服务器是否将视频发送给客户端。

从客户端到服务器的命令结构如下所示：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/The_command_structure_of_receiveVideo_command_from_the_client_to_the_server.jpg)

如果在将`Bool Flag`设置为false的情况下发送`receiveVideo`命令，则服务器不会发送任何响应。 如果`Bool Flag`设置为true，则服务器会使用状态消息`NetStream.Seek.Notify`和`NetStream.Play.Start`作为响应。

#### publish

客户端发送`publish`命令以将已命名的流发布到服务器。 使用该名称，任何客户端都可以播放此流并接收已发布的音频，视频和数据消息。

从客户端到服务器的命令结构如下所示：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/The_command_structure_of_publish_command_from_the_client_to_the_server.jpg)

服务器使用`onStatus`命令进行响应以标记`publish`的开始。

#### seek

客户端发送`seek`命令来查找媒体文件或播放列表中的偏移量（以毫秒为单位）。

从客户端到服务器的命令结构如下所示：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/The_command_structure_of_seek_command_from_the_client_to_the_server.jpg)

当`seek`成功时，服务器发送状态消息`NetStream.Seek.Notify`。 如果失败，它将返回一个`_error`消息。

#### pause

客户端发送暂停命令以通知服务器暂停或开始播放。

从客户端到服务器的命令结构如下所示：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/The_command_structure_of_pause_command_from_the_client_to_the_server.jpg)

当流暂停时，服务器发送状态消息`NetStream.Pause.Notify`。当流处于未暂停状态时发送`NetStream.Unpause.Notify`。 如果失败，将返回一个`_error`消息。

## 消息交换示例

以下是几个解释RTMP消息交换的示例。

### 发布录制的视频

此示例说明发布者如何发布流并将视频流式传输到服务器。 其他客户端可以订阅此发布的流并播放视频。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Message_flow_in_publishing_a_video_stream.jpg)

### 广播共享对象消息

此示例说明在创建和更改共享对象期间交换的消息。 它还说明了共享对象消息广播的过程。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Shared_object_message_broadcast.jpg)

### 从录像的流发布元数据

本示例描述了发布元数据的消息交换。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/image/rtmp_img/Publishing_metadata.jpg)

> FMS: Flash Media Server

---

参考：

- [rtmp_specification_1.0.pdf](http://wwwimages.adobe.com/www.adobe.com/content/dam/acom/en/devnet/rtmp/pdf/rtmp_specification_1.0.pdf)