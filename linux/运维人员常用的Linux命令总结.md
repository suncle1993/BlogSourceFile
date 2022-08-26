---
abbrlink: 161615340
alias: 2016/03/28/运维人员常用的Linux命令总结/index.html
categories:
- linux
date: '2016-03-28T10:35:56'
description: ''
tags:
- 运维
- Linux
- 命令
- 总结
title: 运维人员常用的Linux命令总结
---







---

## 目录结构

| 目录               | 说明                                       |
| ---------------- | ---------------------------------------- |
| /bin             | 存放可执行文件                                  |
| /boot            | 核心与启动相关文件                                |
| /dev             | 设备有关的文件                                  |
| /etc             | 相关的配置信息                                  |
| /etc/rc.d        | 存放开关机过程中用到的脚本文件                          |
| /etc/rc.d/init.d | 所以服务默认的启动脚本都放在这里                         |
| /etc/xinetd.d    | 启动服务可在此找到                                |
| /etc/X11         | 与X windows有关的配置文件                        |
| /lib             | 执行或编译某些程序时用到的函数库                         |
| /proc            | 系统核心与执行程序所需要的一些信息。都是内存中的数据               |
| /root            | 系统管理员根目录                                 |
| /sbin            | 系统管理常用的程序                                |
| /tmp             | 存放临时文件的地方                                |
| /usr             | 存放系统信息，用来存放程序与指令。类似windows下的program  flies |

其中重点需要掌握的是/etc目录和/proc目录。

<!--more-->

## 监控

### 查看CPU详细信息

cpu相关信息存放在/proc/cpuinfo目录中，所以要查看cpu信息就可以用以下命令：

```shell
cat /proc/cpuinfo
```

得到相关的cpu信息如下：

```
processor	: 0
vendor_id	: GenuineIntel
cpu family	: 6
model		: 60
model name	: Intel(R) Pentium(R) CPU G3260 @ 3.30GHz
stepping	: 3
cpu MHz		: 800.000
cache size	: 3072 KB
physical id	: 0
siblings	: 2
core id		: 0
cpu cores	: 2
apicid		: 0
initial apicid	: 0
fpu		: yes
fpu_exception	: yes
cpuid level	: 13
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good xtopology nonstop_tsc aperfmperf pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 cx16 xtpr pdcm pcid sse4_1 sse4_2 movbe popcnt tsc_deadline_timer xsave rdrand lahf_lm abm arat epb xsaveopt pln pts dts tpr_shadow vnmi flexpriority ept vpid fsgsbase erms invpcid
bogomips	: 6584.81
clflush size	: 64
cache_alignment	: 64
address sizes	: 39 bits physical, 48 bits virtual
power management:

processor	: 1
vendor_id	: GenuineIntel
cpu family	: 6
model		: 60
model name	: Intel(R) Pentium(R) CPU G3260 @ 3.30GHz
stepping	: 3
cpu MHz		: 800.000
cache size	: 3072 KB
physical id	: 0
siblings	: 2
core id		: 1
cpu cores	: 2
apicid		: 2
initial apicid	: 2
fpu		: yes
fpu_exception	: yes
cpuid level	: 13
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good xtopology nonstop_tsc aperfmperf pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 cx16 xtpr pdcm pcid sse4_1 sse4_2 movbe popcnt tsc_deadline_timer xsave rdrand lahf_lm abm arat epb xsaveopt pln pts dts tpr_shadow vnmi flexpriority ept vpid fsgsbase erms invpcid
bogomips	: 6584.81
clflush size	: 64
cache_alignment	: 64
address sizes	: 39 bits physical, 48 bits virtual
power management:

```

上面的这些cpu信息我们需要关注的是processor，physical id，siblings，core id，cpu cores这几个字段。这几个字字段的含义如下图：

![Linux-cpuinfo](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Linux-cpuinfo.png)

根据上面的介绍可知我这台服务器上有一个物理cpu，2个逻辑处理器（逻辑cpu），这个物理cpu有两个内核。

可以通过以下方法查询CPU状态。

#### 查询逻辑CPU个数

```
cat /proc/cpuinfo | grep "processor" | wc -l
```

#### 查询物理CPU个数

```
cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l
```

#### 查询每个物理cpu中core的个数

```
cat /proc/cpuinfo | grep "core id" | wc -l
```

### 查看cpu利用率

查看cpu利用率可以用top命令。top命令可以显示当前系统正在执行的进程的相关信息，包括进程ID、内存占用率、CPU占用率等。

关于cpu利用率和cpu负载的详细计算方法可以参见：[Load和CPU利用率是如何算出来的](http://www.penglixun.com/tech/system/how_to_calc_load_cpu.html)

### linux版本信息   

两种方法：查看`cat /proc/version`文件或者 `lsb_release -a`命令

```
[root@localhost /]# cat /proc/version 
Linux version 2.6.32-431.el6.x86_64 (mockbuild@x86-023.build.eng.bos.redhat.com) (gcc version 4.4.7 20120313 (Red Hat 4.4.7-4) (GCC) ) #1 SMP Sun Nov 10 22:19:54 EST 2013

[root@localhost /]# lsb_release -a
LSB Version:	:base-4.0-amd64:base-4.0-noarch:core-4.0-amd64:core-4.0-noarch:graphics-4.0-amd64:graphics-4.0-noarch:printing-4.0-amd64:printing-4.0-noarch
Distributor ID:	RedHatEnterpriseServer
Description:	Red Hat Enterprise Linux Server release 6.5 (Santiago)
Release:	6.5
Codename:	Santiago
```

### 内存信息

```
cat/proc/meminfo   
free –m  
top    
```

下面列出free -m的结果：

```
[root@localhost proc]# free -m
             total       used       free     shared    buffers     cached
Mem:          7747       5392       2355          0        230       3258
-/+ buffers/cache:       1903       5843
Swap:         7999          0       7999
```

**Mem行:**

| 指标      | 含义           | 大小    |
| ------- | ------------ | ----- |
| total   | 内存总数         | 7747M |
| used    | 已经使用的内存数     | 5392M |
| free    | 空闲的内存数       | 2355M |
| shared  | 当前已经废弃不用，总是0 | 0     |
| buffers | Buffer 缓存内存数 | 230   |
| cached  | Page 缓存内存数   | 3258  |

所以有关系：total(7747M) = used(5392M) + free(2355M)

**(-/+ buffers/cache)行:**

- (-buffers/cache) used内存数（已占用）：1903M(指的Mem行中的used - buffers - cached)


- (+buffers/cache) free内存数（可使用）：5843M (指的Mem行中的free + buffers + cached)

可见-buffers/cache反映的是被程序实实在在吃掉的内存，而+buffers/cache反映的是可以挪用的内存总数。

**swap行：**

swap内存如果经常是使用很多，就表示内存不足需要加物理内存了。

**内存使用率的计算：**

- 内存使用率=真实内存占用/内存总数
- 真实内存占用=used-buffers-cached
- 空闲内存=free + buffers + cached

### 磁盘情况

- `df -h` ：按照G显示，`fdisk`和`lsblk`没有权限时最好使用`df -h`


- `df -l` ：按照K显示       
- `fdisk -l` ：显示磁盘详细信息     
- **`lsblk` ：格式整齐，最为推荐使用**

```
[root@localhost /]# lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0   461G  0 disk 
├─sda1   8:1    0 402.9G  0 part /
├─sda2   8:2    0  50.4G  0 part /home
└─sda3   8:3    0   7.8G  0 part [SWAP]
sr0     11:0    1  1024M  0 rom 

[root@localhost /]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       397G   21G  356G   6% /
tmpfs           3.8G     0  3.8G   0% /dev/shm
/dev/sda2        50G  180M   47G   1% /home

[root@localhost /]# df -l
Filesystem     1K-blocks     Used Available Use% Mounted on
/dev/sda1      415787952 21556856 373110280   6% /
tmpfs            3966492        0   3966492   0% /dev/shm
/dev/sda2       51999916   184304  49174156   1% /home


[root@localhost /]# fdisk -l

Disk /dev/sda: 495.0 GB, 495041143296 bytes
255 heads, 63 sectors/track, 60185 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disk identifier: 0x66cbb80d

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *           1       52589   422416384   83  Linux
/dev/sda2           52589       59166    52829184   83  Linux
/dev/sda3           59166       60186     8192000   82  Linux swap / Solaris
```

## 字符集相关

### 查看当前字符集

```
echo $LANG
```

### 系统所有字符集

```
locale -a
```

### 临时修改字符集

```
export LANG=字符集
```

### 字符集配置文件修改

```
vi /etc/sysconfig/i18n
```

## 服务

### 防火墙开关

```
#查看防火墙状态
service iptables status
#开启防火墙
service iptables start
#关闭防火墙
service iptables stop
#重启防火墙
service iptables restart
```

### ftp服务开关

```
#查看ftp服务状态
service vsftpd status
#开启/关闭/重启防火墙
service vsftpd start/stop/restar
```

## 常用命令

###查看文本命令：cat  、tail、vim

例子:`cat/tail/vim  a.txt`

```
#查看/proc/cpuinfo的最后五行
[root@localhost proc]# tail -n 5 /proc/cpuinfo
clflush size	: 64
cache_alignment	: 64
address sizes	: 39 bits physical, 48 bits virtual
power management:
```

### 文本处理sed

sed命令的使用参考：[https://man.linuxde.net/sed](https://man.linuxde.net/sed)

后续会自己总结一篇sed命令的使用心得。

### 修改系统时间

date：系统时间

clock：硬件时间

hwclock：同步二者的命令

**修改系统时间**：

```
date -s "20160408 12:52:00"	#修改系统时间为20160408 12:52:00

hwclock --systohc	#将硬件时钟调整为与目前的系统时钟一致

hwclock --hctosys	#将系统时钟调整为与目前的硬件时钟一致

（这两个选项很容易理解反）
```

### 文件查找find

在/home目录下查找以.txt结尾的文件名

```
find/home -name "*.txt"
```

### 远程拷贝scp

**从远处复制到本地**

```
scp -r root@192.168.118.1:/opt/soft/mongodb /opt/soft/
```

其中-r表示递归复制，类似cp，目录必须已存在。

**从本地复制到远处**

```
scp /opt/soft/mysql-5.6.0.tar.gz root@192.168.118.1:/opt/soft/scptest
```

**从远程复制到远程**

```
scp -r root@192.168.118.1:/opt/soft/mongodb root@192.168.118.3:/opt/soft
```

### 目录创建删除

#### mkdir

```
mkdir -p /tmp/aa/bb/cc
```

#### rmdir

只能删除空目录。`rmdir /tmp/aa`会报错

### 文件删除rm

递归的删除文件或目录

```
rm -rf /tmp/aa
```

### 文件移动mv

**文件移动**

```
mv /tmp/test.file /tmp/lib/
```

**文件更名**

```
mv /tmp/test.file /tmp/lib/test1.file
```

### 查看登陆用户who

```
[weblogic@gssbf01 /]$ who
weblogic pts/2        2016-04-08 10:01 (ip不显示了(*^__^*) 嘻嘻……)
weblogic pts/3        2016-04-08 13:18 (ip不显示了(*^__^*) 嘻嘻……)
weblogic pts/4        2016-04-08 14:11 (ip不显示了(*^__^*) 嘻嘻……)
[weblogic@gssbf01 /]$ whoami
weblogi
```

### 系统重启

```
reboot
```

### 改变权限chmod

功能：更改文件和目录的权限。

用法：chomod 权限分配 文件

```
chmod u+rwx,g+rw,o+r aa.txt	#分别为属主，属组，其他分配权限
chmod 764 aa.txt
```

### 改变用户和组chown

功能：更改文件或者目录的属主属组

用法：`chown [OPTION]... [OWNER][:[GROUP]] FILE...`

```
#改变文件属主
chown weblogic nohup.log
#改变文件属组
chown :weblogic nohup.log
#改变文件属主属组
chown weblogic:weblogic nohup.log
```

只有文件主和超级用户才可以使用该命令。（基本都是超级管理员去修改）

### 压缩解压

平时遇到的基本都是tar.gzip包，用到的最多的命令就是下面两种。（我们采用在参数前不加'-'的旧风格，避免报错）

**压缩时：**-c

```
#打包，-c创建新包，-f制定新包的名称，结果会得到一个名为backup.tar的包
tar cvf backup.tar /etc
#压缩
gzip backup.tar	#压缩之后会得到backup.tar.gz压缩包
bzip2 backup.tar	#压缩之后得到backup.tar.bz2压缩包

#等价于
tar cvfz backup.tar.gz /etc		#-z：通过gzip指令处理打包文件
tar cvfj backup.tar.bz2 /etc	#-j：通过bzip2指令处理打包文件
```

**解压时：**-x

```
#解压缩
gunzip backup.tar.gz	#得到backup.tar，同时压缩包消失
bunzip2 backup.tar.bz2	#得到backup.tar，同时压缩包消失
#解包
tar xvf backup.tar	#得到打包之前的目录，并且backup.tar包不消失

等价于
tar xvfz backup.tar.gz	#-z按照gunzip解压，压缩包不消失
tar xvfj backup.tar.bz2	#-j按照bunzip2解压，压缩包不消失
```

### 杀掉进程ps

ps命令用来列出系统中当前运行的那些进程，为我们提供了进程的一次性的查看，它所提供的查看结果并不动态连续的；如果想对进程时间监控，应该用 top 工具。

```
ps -ef	#后面可以跟上grep命令
```

如果查到需要杀死的进程，则可以用kill命令处理。

用法：**kill 进程号**

```shell
[weblogic@localhost ~]$ ps -ef | grep vim
root     11588 10104  0 17:16 pts/5    00:00:00 vim aa.txt
weblogic 11600 11543  0 17:17 pts/0    00:00:00 grep --color vim

[root@localhost ~]# kill 11588
#或者
[root@localhost ~]# kill -9 11588	#kill发出第九种信号（SIGKILL），可以无条件杀死进程。
```

### Linux相关配置文件

| 配置文件                                    | 作用          |
| --------------------------------------- | ----------- |
| /etc/profile                            | 配置全局的环境变量   |
| ~/.bash_profile                         | 配置当前用户的环境变量 |
| /etc/xinetd.conf文件和.d/etc/xinetd.conf目录 | 配置常用的服务     |
| /etc/rc.d/rc.local                      | 开机启动脚本      |