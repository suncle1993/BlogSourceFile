---
categories:
  - linux
date: '2019-03-30T01:00:15'
description: ''
tags:
  - Linux
  - Bash
title: 常用Linux命令集锦
---




常用linux命令:

- 随机数
- 时间
- ip
- 日志分析
- 硬件信息
- 进程端口
- 调整分区大小

<!--more-->

## 随机数

生成10以内的随机数

```bash
echo $RANDOM%10 | bc
```

生成11-20之间的随机数

```
echo $(($(date +%s)%11+20-11))
```

## 时间

获取当前系统的时间

```bash
date|cut -c 1-26  # 2016年 07月 23日 星期六 19:12:40
```

查看硬件时间

```bash
hwclock
```

修改系统时间

```bash
date -s "2019-03-29 13:58:00" 
```

将系统时间同步到硬件时间

```
hwclock --systohc
```

## ip

```bash
[root@prcdo ~]# curl ipinfo.io/json
{
  "ip": "138.68.1.61",
  "city": "Santa Clara",
  "region": "California",
  "country": "US",
  "loc": "37.3501,-121.9850",
  "postal": "95051",
  "org": "AS14061 DigitalOcean, LLC"
}

[root@prcdo ~]# curl ip.cn
当前 IP：138.68.1.61 来自：美国 DigitalOcean

[root@prcdo ~]# curl ip.tl
IP: 138.68.1.61
DigitalOcean, LLC (AS14061) New York New York United States
```

## 日志分析

日志分析命令：

- 去重： uniq
- 排序： sort
- 统计： wc
- 筛选： awk
- 查找： grep

```bash
# 查看日志去除重复
cat catalina.out |grep "xxxxx"|awk -F ']' '{print $2}'|sort|uniq

# 统计去除重复的行数
cat catalina.out |grep "xxxxx"|awk -F ']' '{print $2}'|sort|uniq|wc -l 
 
# 查询日志大于某一个时间点的日志,并且去重复
cat catalina.out | grep "xxxxxxxx"|awk '($0>"05-19 18:20:00"){print $0}'|awk -F ']' '{print $2}'|sort|uniq

# 统计行数
cat catalina.out | grep "xxxxxxxx"|awk '($0>"05-19 18:20:00"){print $0}'|awk -F ']' '{print $2}'|sort|uniq|wc -l

# 获取disqus-proxy代理日志中的去重文章数量
grep "Get Comments with identifier" disqus-proxy.log | awk '{print $10}' | sort | uniq |wc -l
```

## 硬件信息

截取top命令中的时间

```bash
top -n 1|grep 'top -'|awk '{print $3}'  # 19:12:19
```

截取top命令中三次cpu利用率

```bash
top -n 3 b|grep 'Cpu(s)'|awk '{print $2}'|cut -d '%' -f 1
```

截取top命令中三次cpu利用率的平均值

```bash
top -n 3 b|grep 'Cpu(s)'|awk '{print $2}'|cut -d '%' -f 1|awk '{sum+= $1} END {printf "%.2f\n",sum/3}'  # 输出的0.13就是百分比，即cpu使用率是0.13%
```

截取free命令中的内存使用率

```bash
free|grep 'Mem:'|awk '{realused=$3-$6-$7} END {printf "%.2f\n",realused*100/$2}'
```

截取df命令中的/dev/sda3磁盘使用率

```bash
df|grep '/dev/sda3'|awk '{print $5}'|cut -d '%' -f 1  # 第5项就是磁盘使用率
```

截取df -i命令中的/dev/sda3的Inode使用率

```bash
df -i|grep '/dev/sda3'|awk '{print $5}'|cut -d '%' -f 1  # df -i显示的就是各个磁盘Inode的使用率，其中第5项就是Inode使用率 
```

截取df命令中的挂载在根目录下面的磁盘的使用率

```bash
df|awk '{if ($6=="\/") print $5}'|cut -d '%' -f 1  # awk: 警告: 转义序列“\/”被当作单纯的“/”
df|awk '{if(length($6)==1) print $5}'|cut -d '%' -f 1
```

截取df命令中的挂载在根目录下面的磁盘的Inode使用率

```bash
df -i|awk '{if ($6=="\/") print $5}'|cut -d '%' -f 1  # awk: 警告: 转义序列“\/”被当作单纯的“/”
df -i|awk '{if(length($6)==1) print $5}'|cut -d '%' -f 1
```

find查询结果文件的大小求和

```bash
find . -name "grsds.log.2016-08*" -exec ls -lh {} \;|awk '{print $5}'|cut -d 'M' -f 1|awk '{sum+=$1} END {printf "%.2f\n",sum/1024.0}'  # 已G为单位
```

查看物理CPU个数、核数、逻辑CPU个数

```bash
# 物理CPU：实际服务器中插槽上的CPU个数
# CPU核数：一块CPU上面能处理数据的芯片组的数量
# 逻辑CPU：一般来说，物理CPU个数×每颗核数就应该等于逻辑CPU的个数，如果不相等的话，则表示服务器的CPU支持超线程技术 
# 总核数 = 物理CPU个数 X 每颗物理CPU的核数 
# 总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数

# 查看物理CPU个数
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l

# 查看每个物理CPU中core的个数(即核数)
cat /proc/cpuinfo| grep "cpu cores"| uniq

# 查看逻辑CPU的个数
cat /proc/cpuinfo| grep "processor"| wc -l
```

## 进程端口

根据端口号查看占用端口的进程

```bash
[weblogic@localhost domains]$ lsof -i:8999
COMMAND   PID     USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
java    23256 weblogic  130u  IPv6 22077351      0t0  TCP 192.168.31.72:39609->192.168.31.72:bctp (ESTABLISHED)
java    23256 weblogic  131u  IPv6 22077353      0t0  TCP 192.168.31.72:39610->192.168.31.72:bctp (ESTABLISHED)
java    23256 weblogic  132u  IPv6 22077357      0t0  TCP 192.168.31.72:39611->192.168.31.72:bctp (ESTABLISHED)
java    23256 weblogic  133u  IPv6 22077359      0t0  TCP 192.168.31.72:39612->192.168.31.72:bctp (ESTABLISHED)
[weblogic@localhost domains]$ ps -ef | grep 23256
weblogic  5777  5745  0 17:04 pts/1    00:00:00 grep --color 23256
weblogic 23256     1  1  2016 ?        05:19:04 java -jar netbridge-client-0.0.1-SNAPSHOT.jar
```

根据进程名称查看进程号

```bash
[weblogic@localhost domains]$ ps -ef | grep netbridge
weblogic  5811  5745  0 17:16 pts/1    00:00:00 grep --color netbridge
weblogic 20109     1  2  2016 ?        10:31:37 java -jar netbridge-server-0.0.1-SNAPSHOT.jar
weblogic 20141     1  1  2016 ?        05:32:17 java -jar netbridge-client-0.0.1-SNAPSHOT.jar
weblogic 23234     1  1  2016 ?        05:19:17 java -jar netbridge-server-0.0.1-SNAPSHOT.jar
weblogic 23256     1  1  2016 ?        05:19:12 java -jar netbridge-client-0.0.1-SNAPSHOT.jar
```

根据进程号查看端口

```bash
[weblogic@localhost domains]$ netstat -anp | grep 23256
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)           
tcp        0      0 ::ffff:192.168.31.72:39539  ::ffff:192.168.31.72:8999   ESTABLISHED 23256/java          
tcp        0      0 ::ffff:192.168.31.72:39545  ::ffff:192.168.31.72:8999   ESTABLISHED 23256/java          
tcp        0      0 ::ffff:192.168.31.72:39559  ::ffff:192.168.31.72:8999   ESTABLISHED 23256/java          
unix  2      [ ]         STREAM     CONNECTED     22077363 23256/java  
```

## 调整分区大小

```bash
[root@cs02 /]# df -h
文件系统	      容量  已用  可用 已用%% 挂载点
/dev/mapper/vg_cs01-lv_root
                       50G  2.1G   45G   5% /
tmpfs                 3.8G     0  3.8G   0% /dev/shm
/dev/sda2             485M   37M  423M   9% /boot
/dev/sda1             200M  260K  200M   1% /boot/efi
/dev/mapper/vg_cs01-lv_home
                      399G  200M  378G   1% /home
[root@cs02 /]# umount /home
[root@cs02 /]# df -h
文件系统	      容量  已用  可用 已用%% 挂载点
/dev/mapper/vg_cs01-lv_root
                       50G  2.1G   45G   5% /
tmpfs                 3.8G     0  3.8G   0% /dev/shm
/dev/sda2             485M   37M  423M   9% /boot
/dev/sda1             200M  260K  200M   1% /boot/efi
[root@cs02 /]# mount /home
[root@cs02 /]# df -h
文件系统	      容量  已用  可用 已用%% 挂载点
/dev/mapper/vg_cs01-lv_root
                       50G  2.1G   45G   5% /
tmpfs                 3.8G     0  3.8G   0% /dev/shm
/dev/sda2             485M   37M  423M   9% /boot
/dev/sda1             200M  260K  200M   1% /boot/efi
/dev/mapper/vg_cs01-lv_home
                      399G  200M  378G   1% /home
[root@cs02 /]# umount /home
[root@cs02 /]# df -h
文件系统	      容量  已用  可用 已用%% 挂载点
/dev/mapper/vg_cs01-lv_root
                       50G  2.1G   45G   5% /
tmpfs                 3.8G     0  3.8G   0% /dev/shm
/dev/sda2             485M   37M  423M   9% /boot
/dev/sda1             200M  260K  200M   1% /boot/efi
[root@cs02 /]# resize2fs -p /dev/mapper/vg_cs01-lv_home 50G
resize2fs 1.41.12 (17-May-2010)
请先运行 'e2fsck -f /dev/mapper/vg_cs01-lv_home'.

[root@cs02 /]# e2fsck -f /dev/mapper/vg_cs01-lv_home
e2fsck 1.41.12 (17-May-2010)
第一步: 检查inode,块,和大小
第二步: 检查目录结构
第3步: 检查目录连接性
Pass 4: Checking reference counts
第5步: 检查簇概要信息
/dev/mapper/vg_cs01-lv_home: 36/26509312 files (0.0% non-contiguous), 1714751/106011648 blocks
[root@cs02 /]# resize2fs -p /dev/mapper/vg_cs01-lv_home 50G
resize2fs 1.41.12 (17-May-2010)
Resizing the filesystem on /dev/mapper/vg_cs01-lv_home to 13107200 (4k) blocks.
Begin pass 2 (max = 32991)
正在重定位块            XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Begin pass 3 (max = 3236)
正在扫描inode表          XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Begin pass 4 (max = 10)
正在更新inode引用       XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
The filesystem on /dev/mapper/vg_cs01-lv_home is now 13107200 blocks long.

[root@cs02 /]# mount /home
[root@cs02 /]# df -h
文件系统	      容量  已用  可用 已用%% 挂载点
/dev/mapper/vg_cs01-lv_root
                       50G  2.1G   45G   5% /
tmpfs                 3.8G     0  3.8G   0% /dev/shm
/dev/sda2             485M   37M  423M   9% /boot
/dev/sda1             200M  260K  200M   1% /boot/efi
/dev/mapper/vg_cs01-lv_home
                       50G  181M   47G   1% /home
[root@cs02 /]# lvreduce -L 50G /dev/mapper/vg_cs01-lv_home
  WARNING: Reducing active and open logical volume to 50.00 GiB
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce lv_home? [y/n]: y
  Reducing logical volume lv_home to 50.00 GiB
  Logical volume lv_home successfully resized
[root@cs02 /]# df -h
文件系统	      容量  已用  可用 已用%% 挂载点
/dev/mapper/vg_cs01-lv_root
                       50G  2.1G   45G   5% /
tmpfs                 3.8G     0  3.8G   0% /dev/shm
/dev/sda2             485M   37M  423M   9% /boot
/dev/sda1             200M  260K  200M   1% /boot/efi
/dev/mapper/vg_cs01-lv_home
                       50G  181M   47G   1% /home
[root@cs02 /]# vgdisplay
  --- Volume group ---
  VG Name               vg_cs01
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  5
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               3
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               462.11 GiB
  PE Size               4.00 MiB
  Total PE              118299
  Alloc PE / Size       27572 / 107.70 GiB
  Free  PE / Size       90727 / 354.40 GiB
  VG UUID               pNF2xf-FCVn-GRTw-841J-4kJN-V6kg-4ux3OD
   
[root@cs02 /]# lvextend -L +349G /dev/mapper/vg_cs01-lv_root
  Extending logical volume lv_root to 399.00 GiB
  Logical volume lv_root successfully resized
[root@cs02 /]# df -h
文件系统	      容量  已用  可用 已用%% 挂载点
/dev/mapper/vg_cs01-lv_root
                       50G  2.1G   45G   5% /
tmpfs                 3.8G     0  3.8G   0% /dev/shm
/dev/sda2             485M   37M  423M   9% /boot
/dev/sda1             200M  260K  200M   1% /boot/efi
/dev/mapper/vg_cs01-lv_home
                       50G  181M   47G   1% /home
[root@cs02 /]# resize2fs -p /dev/mapper/vg_cs01-lv_root
resize2fs 1.41.12 (17-May-2010)
Filesystem at /dev/mapper/vg_cs01-lv_root is mounted on /; on-line resizing required
old desc_blocks = 4, new_desc_blocks = 25
Performing an on-line resize of /dev/mapper/vg_cs01-lv_root to 104595456 (4k) blocks.
The filesystem on /dev/mapper/vg_cs01-lv_root is now 104595456 blocks long.

[root@cs02 /]# resize2fs -p /dev/mapper/vg_cs01-lv_root
resize2fs 1.41.12 (17-May-2010)
The filesystem is already 104595456 blocks long.  Nothing to do!
```

