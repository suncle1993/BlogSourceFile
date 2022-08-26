---
abbrlink: 3452107703
alias: 2016/03/16/Hexo博客换电脑步骤/index.html
categories:
- 随笔
date: '2016-03-16T20:23:24'
description: ''
tags:
- Hexo
- 多电脑
- github
title: Hexo博客换电脑步骤
---







![](http://upload.chinaz.com/2015/1016/1444964380585.png)

hexo博客固然好，但是在多个电脑上怎么同步写博客呢？

<!--more-->

**搭建hexo环境:**

要想在一个电脑上写博客，那肯定就得搭建node.js+hexo的环境。

**首先，安装node.js**

node.js的下载可以在node.js的[中文网站](https://nodejs.cn/)下载，也可以使用下列百度云地址：

链接: [https://pan.baidu.com/s/1kTVhcvd](https://pan.baidu.com/s/1kTVhcvd) 密码: mqsy

**然后安装git for windows**

git for windows可以从百度的软件中心[http://dlsw.baidu.com/sw-search-sp/soft/4e/30195/Git-2.7.2-32-bit_setup.1457942412.exe](http://dlsw.baidu.com/sw-search-sp/soft/4e/30195/Git-2.7.2-32-bit_setup.1457942412.exe)下载，也可以从github官网下载。还可以使用下面的百度云地址：

链接: [https://pan.baidu.com/s/1eRtDECE](https://pan.baidu.com/s/1eRtDECE) 密码: cxh5

安装好nodej.js之后，打开git bash安装hexo，安装步骤如下：

```
npm install -g hexo
```

将之前备份的博客站点文件全部复制到本地的一个目录下。就可以使用下列命令操作了

```
hexo clean
hexo g
hexo s
hexo d
```

至于博客源文件可以用git管理，pull下来即可。

关于git管理博客源文件的方法参加另一篇blog：[使用Github管理Hexo博客的源文件](https://suncle.me/2016/03/16/%E4%BD%BF%E7%94%A8github%E7%AE%A1%E7%90%86hexo%E5%8D%9A%E5%AE%A2%E7%9A%84%E6%BA%90%E6%96%87%E4%BB%B6/)

**附：**

其实这一篇的主要目的只是为了放置nodejs和git的下载链接，因为官网是在是下载不了，速度慢的要屎了。