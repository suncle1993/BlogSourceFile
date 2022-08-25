---
categories:
  - linux
date: '2016-01-10T14:15:34'
description: ''
id: 179
tags:
  - CentOS
  - Linux
  - vi
  - vim
  - 总结
  - 模式
  - 配置
title: Linux vi的使用
---



# Linux vi的使用

* * *

## vi模式转换

经常使用的三种基本模式：命令模式(Command Mode)，输入模式(Input Mode)，末行模式(Last Line Mode)，其他的9种模式不做介绍，很少会使用。

三种基本模式的相互转换如下图：

![vi模式转换](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Flowsnowvi%E6%A8%A1%E5%BC%8F%E8%BD%AC%E6%8D%A2.jpg)

<!--more-->

## vi文件保存和退出

- :w 保存文件


- :q 退出文件，若文件有改动则提示不能退出


- :q! 强制退出，即不保存就退出


- :wq 保存并且退出

## vi常用操作

### 1、插入文本(i,I,a,A,o,O)

#### 添加：

- 输入a后，在光标的右边插入文本


- 输入A，在一行的结尾处添加文本

#### 插入：

- 通过在命令模式下输入i，在光标的左边插入文本


- 通过在命令模式下输入I，在行首插入文本

#### 插入新行：

- 输入o，在当前光标位置下面打开一行


- 输入O，在当前光标位置上面打开一行

### 2、撤消更改

- 撤消前一个命令：在最后一个命令之后立即输入u来撤消该命令


- 重复某个命令：“.”


- 撤消对一行的更改：输入U来撤消你对一行所做的所有更改，这个命令只有在你没将光标移动到该行以外时才生效

### 3、删除文本

#### 删除一个字符

- 为删除一个字符，需将光标放置在要删除的字符上并输入x


- 为删除光标之前（其左边）的一个字符，需输入X

#### 删除一个词或词的部分内容

- 为删除一个词，需将光标放置到该词的开头并输入dw


- 为删除词的部分内容，需将光标放置到该词要保存部分的右边。输入dw来删除该词余下的部分

#### 删除一行

- 将光标放置到该行的任意处并输入dd

#### 删除多行

- ndd　　　　包括当前行

#### 删除到文件的结尾

- 为删除从当前行到文件结尾的所有内容(包括当前行)，需输入dG

### 4、复制

- 复制一行命令：yy


- 粘贴命令：p　　（粘贴到当前行的下一行）


- 复制指定文件的内容　　: r filename

### 5、查找一个字符串

- 输入/，并在/后面输入要查找的串，然后按下回车


- 输入“n”跳转到该串的下一个出现处，跳到最后一个时会循环跳到第一个


- 输入“N”跳转到该串的上一个出现处

### 6、替换一个字符串

- 在一行内替换头一个字符串old为新的字符串new    :s/old/new


- 在一行内替换所有的字符串old为新的字符串new    :s/old/new/g


- 在文件内替换所有的字符串old为新的字符串new    :%s/old/new/g


- 进行全文替换时询问用户确认每个替换需添加c选项     :%s/old/new/gc

## vim配置

vimrc文件

找到vim配置文件的位置。如果是默认安装，CentOS和RHEL一般在/etc/vimrc下面，Debian和Ubuntu一般在/usr/share/vim/vimrc

## vi的使用参考资料

[http://wiki.dzsc.com/info/7313.html](http://wiki.dzsc.com/info/7313.html)

[https://www.cnblogs.com/mahang/archive/2011/09/01/2161672.html](https://www.cnblogs.com/mahang/archive/2011/09/01/2161672.html)

[http://blog.chinaunix.net/uid-384966-id-2411343.html](http://blog.chinaunix.net/uid-384966-id-2411343.html)

[https://www.cnblogs.com/zgx/archive/2011/04/12/2013356.html](https://www.cnblogs.com/zgx/archive/2011/04/12/2013356.html)