---
categories:
  - linux
date: '2017-09-02T00:53:01'
description: ''
tags:
  - samba
  - linux
  - ubuntu
  - svn
title: ubuntu16.04配置samba解决linux的svn使用舒适问题
---




个人感觉，svn的命令行使用起来没有git那么舒适，但是windows上的svn GUI客户端TortoiseSVN 使用非常方便。因此对于经常在虚拟机中做服务程序开发但是又不得不用svn的同学来说，结合linux开发环境和TortoiseSVN 来管理代码版本就显得尤其有用。

# 安装配置samba

ubuntu上使用apt-get安装

```shell
apt-get install samba samba-common
```

<!--more-->

关闭防火墙

```shell
systemctl stop ufw
```

使用`vim /etc/samba/smb.conf`命令编辑samba配置文件，在配置文件最后添加即可

```ini
[homes]
   comment = qk_python Directories
   browseable = no
   path = /root/qk_python
   valid users = root
   read only = no
```

添加用户，除了root用户外也可以输入其他的存在用户名。

```shell
smbpasswd -a root
```

重启samba服务生效

```shell
systemctl restart smbd
```

在Windows下运行窗口输入`\\`加上IP，例如：`\\192.168.1.177\root`。在弹出的窗口，输入刚刚添加的用户名和密码，就可以访问Linux的文件目录了。

# 配置svn

由于配置samba的时候配置成了非只读的，因此可以直接checkout相应的svn项目到Linux文件目录中。完成之后对svn做以下配置：

勾选svn的网络驱动类型

> TortoiseSVN->Settings->Icon Overlays  勾选Driver Types中的”Network drives”

显示svn项目绿色图标：

> TortoiseSVN->Settings->Icon Overlays 选择Shell

然后就可以显示绿色图标了，接下来就愉快的使用TortoiseSVN管理Linux代码吧。

