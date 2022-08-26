---
abbrlink: 1407336675
alias: 2016/01/25/win7x64安装配置Oracle 11g R2数据库/index.html
categories:
- 数据库
date: '2016-01-25T10:41:46'
description: ''
tags:
- Oracle数据库
- 安装
title: win7x64安装配置Oracle 11g R2数据库
---








## 安装前的准备

Oracle数据库安装文件，下载地址见Oracle官网：[https://www.oracle.com/](https://www.oracle.com/)

## 开始安装

1、Oracle11gR2的安装文件分成了两个，需要我们在解压的时候，解压到相同的目录下边，里面文件的目录名是默认的database，如图所示：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-oracle%E6%95%B0%E6%8D%AE%E5%BA%93%E5%AE%89%E8%A3%85%E5%8C%85.jpg)



2、解压到相同目录之后，双击文件夹中的setup.exe文件，双击之后，出现启动安装界面的命令窗口，会检查监视器配置，并且启动安装界面进入正式安装界面，如图所示，取消下图所示的选中，即不接受安全更新，然后单击"下一步"继续，会提示尚未提供电子邮件，不管，直接点是，Oracle默认是选择基本安装。

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-oracle%E6%AD%A3%E5%BC%8F%E5%AE%89%E8%A3%85%E7%95%8C%E9%9D%A2.jpg)



<!--more-->



3、在“选择安装选项”界面中，按照默认的设置选择“创建和配置数据库”，单击“下一步”按钮，如图所示：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-Oracle%E9%80%89%E6%8B%A9%E5%AE%89%E8%A3%85%E9%80%89%E9%A1%B9.jpg)



4、在“系统类”界面中，选择“桌面类”即可，单击“下一步”按钮，如图所示：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-Oracle%E7%B3%BB%E7%BB%9F%E7%B1%BB.jpg)



5、在“典型安装配置”界面中，输入oralce的根目录，最好不要包含空格和中文。全局数据库名就是本地数据库的实例名，管理口令适用于sys，system等系统用户。如下图所示：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-Oracle%E5%85%B8%E5%9E%8B%E5%AE%89%E8%A3%85.jpg)



6、然后出现“执行先决条件检查”界面，所出现错误就逐个解决即可，或者直接全部忽略。然后出现“概要”界面，可以看到一些安装的基本信息，单击“完成”按钮，开始安装，进入“安装产品”界面，如下图所示：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-Oracle%E4%BA%A7%E5%93%81%E5%AE%89%E8%A3%85%E8%BF%87%E7%A8%8B.jpg)



7、在安装的过程中，会出现数据库配置助手，帮助我们默认的建立一个数据库，默认创建的数据库是orcl，数据库创建完成之后显示如下的界面如图所示：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-Oracle%E6%95%B0%E6%8D%AE%E5%BA%93%E9%85%8D%E7%BD%AE%E5%8A%A9%E6%89%8B%E7%95%8C%E9%9D%A2.jpg)

8、点击口令管理，进入在弹出的“口令管理”界面中，我们可以看到SYS和SYSTEM用户已经解锁，然后我们可以解锁并且设置scott账户的密码，并激活账户，设置完成后，单击“确定”按钮，如图所示：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/Oracle-Oracle%E5%8F%A3%E4%BB%A4%E7%AE%A1%E7%90%86%E7%95%8C%E9%9D%A2.jpg)



9、数据库安装完成。

数据库控制网址（Enterprise Manager Database Control URL - (orcl) ）：

[https://localhost:1158/em](https://localhost:1158/em)

数据库配置文件已经安装到 D:\Oracle11g\Administrator,同时其他选定的安装组件也已经安装到 D:\Oracle11g\Administrator\product\11.2.0\dbhome_1。