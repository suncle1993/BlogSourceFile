---
categories:
  - linux
date: '2016-01-14T23:20:34'
description: ''
tags:
  - 文件系统
  - Linux
title: Linux的文件和文件系统的管理
---



---

## 文件权限

### 什么是文件系统？

文件系统是操作系统在分区上保存文件信息的方法和数据结构。

### 文件有哪些权限？

![文件有那些权限](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Flowsnow%E6%96%87%E4%BB%B6%E6%9C%89%E5%93%AA%E4%BA%9B%E6%9D%83%E9%99%90.jpg)

 三种基本权限分别为：读、写、执行，说明如下：

| 代表字符 |  权限  |  对文件的意义   |    对目录的意义     |
| :--: | :--: | :-------: | :-----------: |
|  r   | 读权限  |  可以读文件内容  | 可以列出目录中的文件列表  |
|  w   | 写权限  | 可以修改和删除文件 | 可以在目录中创建和删除文件 |
|  x   | 执行权限 |  可以执行该文件  | 可以使用cd命令进入该目录 |

1. 目录上只有执行权限，便是可以进入或穿越他进入更深层次的子目录
2. 目录上只有执行权限，要访问该目录下的又读权限的文件，必须知道文件名才可以访问
3. 目录上只有执行权限，不能列出目录列表也不能删除该目录
4. 目录上执行权限和读权限，表示可以进入目录并且列出目录列表
5. 目录上执行权限和写权限的组合，表示可以在目录中创建、删除和重命名文件

### 特殊权限

有三种特殊权限suid、sgid和sticky bit。参见[Linux中的文件特殊权限](http://alan-hjkl.iteye.com/blog/1526858)

<!--more-->

#### suid和sgid

例如查看/usr/bin/passwd 与/etc/passwd文件的权限

``` shell
[root@MyLinux ~]# ls -l /usr/bin/passwd /etc/passwd
-rw-r--r-- 1 root root  1549 08-19 13:54 /etc/passwd
-rwsr-xr-x 1 root root 22984 2007-01-07 /usr/bin/passwd
```

众所周知，/etc/passwd文件存放的各个用户的账号与密码信息，/usr/bin/passwd是执行修改和查看此文件的程序，但从权限上看，/etc/passwd仅有root权限的写（w）权，可实际上每个用户都可以通过/usr/bin/passwd命令修改这个文件，于是这里就涉及了linux里的特殊权限setuid，如-rwsr-xr-x中的s

**suid就是：让普通用户拥有可以执行“只有root权限才能执行”的特殊权限，sgid同理指”组“**

作为普通用户是没有权限修改/etc/passwd文件的，但给/usr/bin/passwd以suid权限后，普通用户就可以通过执行passwd命令，临时的拥有root权限，去修改/etc/passwd文件了

#### sticky bit

例如查看/tmp目录的权限：

``` shell
[root@localhost test]# ll -d /tmp
drwxrwxrwt. 4 root root 4096 1月  18 03:53 /tmp
```

tmp目录是所有用户共有的临时文件夹，所有用户都拥有读写权限，这就必然出现一个问题，A用户在/tmp里创建了文件a.file，此时B用户看了不爽，在/tmp里把它给删了（因为拥有读写权限），那肯定是不行的。实际上是不会发生这种情况，因为有特殊权限sticky bit（粘贴位）权限，正如drwxrwxrwt中的最后一个t。

**sticky bit (粘贴位)就是：除非目录的属主和root用户有权限删除它，除此之外其它用户不能删除和修改这个目录**。

也就是说，在/tmp目录中，只有文件的拥有者和root才能对其进行修改和删除，其他用户则不行，避免了上面所说的问题产生。

**粘贴位的用途一般是把一个文件夹的的权限都打开，然后来共享文件，象/tmp目录一样。**

## 文件格式和类型

一切皆文件。

Linu系统下的文件类型包括：

``` 
普通文件(-)
目录(d)
符号链接(l)
字符设备文件(c)
块设备文件(b)
套接字(s)
命名管道(p)
```

### 链接文件

链接文件分为软链接和硬链接

#### 硬链接（Hard Link）

硬链接指通过索引节点来进行连接。在Linux的文件系统中，保存在磁盘分区中的文件不管是什么类型都给它分配一个编号，称为索引节点号(Inode Index)。在Linux中，多个文件名指向同一索引节点是存在的。一般这种连接就是硬链接。硬链接的作用是允许一个文件拥有多个有效路径名，这样用户就可以建立硬链接到重要文件，以防止“误删”的功能。其原因如上所述，因为对应该目录的索引节点有一个以上的连接。只删除一个连接并不影响索引节点本身和其它的连接，只有当最后一个连接被删除后，文件的数据块及目录的连接才会被释放。

也就是说，文件真正删除的条件是与之相关的所有硬链接文件均被删除。

#### 软链接（Symbolic Link）

除了硬链接之外的一种连接称之为符号连接（Symbolic Link），也叫软连接。软链接文件有类似于Windows的快捷方式。它实际上是一个特殊的文件。在符号连接中，文件实际上是一个文本文件，其中包含的有另一文件的位置信息。

#### 硬链接和软链接的区别演示

ln命令，参见[ln命令](https://man.linuxde.net/ln)

``` shell
[root@localhost test]# touch file1	#创建一个测试文件file1
[root@localhost test]# ln file1 file2	#创建file1的一个硬链接文件file2
[root@localhost test]# ln -s file1 file3	#创建file1的一个软链接文件file3
[root@localhost test]# ls -li	#-i参数显示文件的inode节点信息
总用量 0
528483 -rw-r--r--. 2 root root 0 1月  18 05:46 file1
528483 -rw-r--r--. 2 root root 0 1月  18 05:46 file2
528484 lrwxrwxrwx. 1 root root 5 1月  18 05:47 file3 -> file1
```

从上面的结果中可以看出，硬连接文件file2与原文件file1的inode节点相同，均为528483，然而符号连接文件的inode节点不同。

``` shell
[root@localhost test]# echo "hello liuyifei" >>file1	#向file1中输入信息
[root@localhost test]# cat file1
hello liuyifei
[root@localhost test]# cat file2
hello liuyifei
[root@localhost test]# cat file3
hello liuyifei
[root@localhost test]# rm -f file1
[root@localhost test]# ls -li
总用量 4
528483 -rw-r--r--. 1 root root 15 1月  18 05:53 file2
528484 lrwxrwxrwx. 1 root root  5 1月  18 05:47 file3 -> file1
[root@localhost test]# cat file2
hello liuyifei
[root@localhost test]# cat file3	#可以发现file3这个软链接虽然在，但是没有内容了
cat: file3: 没有那个文件或目录
```

## 改变文件权限

改变文件权限的命令：chmod，chown（更改所有者），chgrp（更改用户组）

三个命令使用方法参见：

- https://man.linuxde.net/chmod
- https://man.linuxde.net/chown
- https://man.linuxde.net/chgrp

### 权限范围的表示法

`u` User，即文件或目录的拥有者； 

`g` Group，即文件或目录的所属群组； 

`o` Other，除了文件或目录拥有者或所属群组之外，其他用户皆属于这个范围； 

`a` All，即全部的用户，包含拥有者，所属群组以及其他用户； 

`r` 读取权限，数字代号为“4”; 

`w` 写入权限，数字代号为“2”； 

`x` 执行或切换权限，数字代号为“1”； 

`-`不具任何权限，数字代号为“0”；

`s` 特殊功能说明：变更文件或目录的权限。

### chmod命令

语法：chmod (选项) (参数)

``` 
-f或--quiet或--silent：不显示错误信息； 

-R或--recursive：递归处理，将指令目录下的所有文件及子目录一并处理； 

-v或--verbose：显示指令执行过程； 

--reference=<参考文件或目录>：把指定文件或目录的所属群组全部设成和参考文件或目录的所属群组相同；

<权限范围>+<权限设置>：开启权限范围的文件或目录的该选项权限设置； 

<权限范围>-<权限设置>：关闭权限范围的文件或目录的该选项权限设置； 

<权限范围>=<权限设置>：指定权限范围的文件或目录的该选项权限设置；
```

例：

``` shell
chmod u+x,g+w file1　　#为文件file1设置自己可以执行，组员可以写入的权限 
chmod u=rwx,g=rw,o=r file1 
chmod 764 file1	#数字设定法，rwxrw-r--
chmod a+x file1	#对文件file1的u,g,o都设置可执行属性
```

### chown

改变某个文件或目录的所有者和所属的组，只有文件主和超级用户才可以使用该命令。

选项：

``` 
-f或--quite或——silent：不显示错误信息； 
-h或--no-dereference：只对符号连接的文件作修改，而不更改其他任何相关文件； 
-R或——recursive：递归处理，将指定目录下的所有文件及子目录一并处理； 
-v：显示指令执行过程；
```

参数：

> 用户:组：指定所有者和所属工作组。当省略“：组”，仅改变文件所有者； 
>
> 文件：指定要改变所有者和工作组的文件列表。支持多个文件和目标，支持shell通配符。

例：

``` shell
chown -R liuyifei /usr/test	#将目录/tmp/test及其下面的所有文件、子目录的文件主改成 liuyifei
chown liuyifei:mingxing file1　　//把文件file1属主改成liuyifei，属组改为mingxing
```

### chgrp

用来改变文件或目录所属的用户组，只有文件主和超级用户才可以使用该命令。

语法：chgrp(选项)(参数)

选项：类似chown

参数：

``` 
组：指定新组名称； 
文件：指定要改变所属组的文件列表。多个文件或者目录之间使用空格隔开。
```

例：

``` shell
chgrp -R mengxin .	#将当前目录及子目录下的所有文件的用户组改为liuyifei
```