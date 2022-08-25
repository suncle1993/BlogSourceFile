---
categories:
  - linux
date: '2016-12-29T14:39:17'
description: ''
tags:
  - Linux
  - C++
  - C++11
title: linux配置c++11编译环境
---



# linux配置c++11编译环境

## 配置yum源

此处我们使用163的yum源，配置如下

**首先备份/etc/yum.repos.d/CentOS-Base.repo**

```shell
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

下载对应版本repo文件, 放入/etc/yum.repos.d/(操作前请做好相应备份)，以下为下载链接

[https://mirrors.163.com/.help/CentOS6-Base-163.repo](https://mirrors.163.com/.help/CentOS6-Base-163.repo)

运行以下命令生成yum缓存

```shell
yum clean all
yum makecache
```

<!--more-->

## 使用yum配置c++编译环境

yum配置好之后，配置c++编译环境命令如下

```shell
yum -y install gcc gcc-g++
```

安装完成之后c++环境即可配置好。

写一个hello worl代码如下

```c++
#include<iostream>

using namespace std;
int main()
{
	cout<<"Hello World!"<<endl;
	return 0;
}
```

以上代码保存文件名为aa.cpp，用c++编译并执行的操作如下

```shell
g++ -o hello aa.cpp
./aa.cpp
```

## 源码编译安装c++11编译环境

因为yum自带的gcc版本过低，并且c++11需要gcc4.8以上版本支持，因此需要下载gcc4.8以上版本以支持c++11

**查看本地gcc版本**

```shell
gcc -v
```

本次版本为`gcc version 4.4.7 20120313 (Red Hat 4.4.7-17) (GCC)`

**获取gcc4.8.2版本的source code**

源码默认放在src目录下

```shell
cd /usr/local/src
wget http://gcc.skazkaforyou.com/releases/gcc-4.8.2/gcc-4.8.2.tar.gz
```

文件有100M，国外网站下载速度很慢，请耐心等待（可用国外vps下载中转）

下载完成后，放在/usr/local/src下

**解压缩**

```shell
tar -zxvf gcc-4.8.2.tar.gz
```

**编译源码并安装**

进入gcc目录

```shell
cd gcc-4.8.2
```

**下载配置安装gcc4.8.2的依赖库**

```shell
./contrib/download_prerequisites
```

**建立编译输出目录**

在当前路径下执行即可

```shell
mkdir gcc-build-4.8.2
```

**开始configure**

```shell
../configure --enable-checking=release --enable-languages=c,c++ --disable-multilib
```

- `--enable-languages`表示你要让你的gcc支持那些语言
- `--disable-multilib`不生成编译为其他平台可执行代码的交叉编译器
- `--disable-checking`生成的编译器在编译过程中不做额外检查

**编译**

在编译输出目录gcc-build-4.8.2直接make即可

```shell
make
```

源码make过程耗时较长，一般需要半个小时以上。

**安装**

```shell
make install
```

**验证是否升级成功**

使用`which gcc`检查gcc安装的为止，使用`gcc -v`检查版本，如果仍然没有变，请关闭当前会话重新连接看是否变成4.8.2，如果仍未变，需要重启系统

**验证C++11程序是否可用**

lambda表达式是C++11的新特性，以下程序即可验证c++11是否可用

参考：[http://en.cppreference.com/w/cpp/container/array](http://en.cppreference.com/w/cpp/container/array)

```c++
#include <iostream>

using namespace std;

int main()

{

   int n = [] (int x, int y) { return x + y; }(5, 4);

   cout << n << endl;

}
```

验证方法

```shell
g++ -std=c++11 -o lambda vv.cpp
```

如果使用g++不加`-std=c++11`参数，则会报错，报错如下

```shell
[root@host-192-168-150-182 tmp]# g++ -o lambda vv.cpp 
vv.cpp: In function ‘int main()’:
vv.cpp:9:46: warning: lambda expressions only available with -std=c++11 or -std=gnu++11 [enabled by default]
    int n = [] (int x, int y) { return x + y; }(5, 4);
                                              ^
```

**更新gcc动态链接库**

源码编译升级安装了gcc后，编译程序或运行其它程序时，有时会出现类似/usr/lib64/libstdc++.so.6: versionGLIBCXX_3.4.18' not found的问题。这是因为升级gcc时，生成的动态库没有替换老版本gcc的动态库导致的，将gcc最新版本的动态库替换系统中老版本的动态库即可解决。可参考以下链接

[http://itbilu.com/linux/management/NymXRUieg.html](http://itbilu.com/linux/management/NymXRUieg.html)