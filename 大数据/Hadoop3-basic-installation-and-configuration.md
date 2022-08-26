---
abbrlink: 4062227706
alias: 2018/04/16/Hadoop3-basic-installation-and-configuration/index.html
categories:
- 大数据
date: '2018-04-16T19:08:28'
description: ''
tags:
- Hadoop
- 安装
- 配置
title: Hadoop3单机和伪分布式模式安装配置
---









> 搭建hadoop

为了体验HDFS和MapReduce框架，以及在HDFS上运行示例程序或简单作业，我们首先需要完成单机上的Hadoop安装。所依赖的软件环境如下：

1. Linux系统：以运行在阿里云ECS上的Ubuntu 16.04 LTS版本为例
2. jdk-8u162-linux-x64.tar.gz
3. hadoop 3.1.0

本次演示统一将软件放置在`/usr/local/src`目录中

<!--more-->

# 创建hadoop用户

首先需要建立一个hadoop用户，用来启动Hadoop的进程，这样避免使用root用户启动进程，这也是比较规范的服务器用户管理，使用以下命令创建hadoop用户：

```bash
useradd -m hadoop -s /bin/bash
passwd hadoop  # 为hadoop用户设置密码，直接设置为hadoop
adduser hadoop sudo  # 为 hadoop 用户增加管理员权限，方便部署
```

后续均在hadoop用户中操作。

**免密码ssh设置**

Hadoop中namenode需要启动集群中的所有机器的Hadoop守护进程，而这个过程需要通过SSH登录来实现。而Hadoop并没有提供SSH输入密码的登录形式，因此为了保证可以顺利登录每台机器，需要将所有机器配置为namenode可以无密码登录它们。所以我们需要配置SSH的无密码访问（注意无密码访问是为hadoop用户配置的，故以下操作需要在hadoop用户下完成）：

```bash
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```

此时再用 `ssh localhost` 命令，无需输入密码就可以直接登陆了（第一次需要先输入一个yes）

# Java安装配置

下载地址：`https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html`

执行以下命令解压

```bash
cd /usr/local/src
tar -xzvf jdk-8u162-linux-x64.tar.gz
```

然后编辑`~/.profile`文件，在文件结尾处增加以下内容：

```bash
# java jdk env settings
JAVA_HOME=/usr/local/src/jdk1.8.0_162
PATH=${JAVA_HOME}/bin:$PATH
export JAVA_HOME PATH
```

修改完成之后使用source命令使配置生效：

```bash
source ~/.profile
```

若输出JAVA_HOME环境变量有结果则说明修改成功：

```bash
echo $JAVA_HOME
```

此时可以使用验证java环境变量是否配置成功：

```bash
java -version
```

# Hadoop安装配置

下载地址：`https://hadoop.apache.org/releases.html`

Hadoop的运行有三种形式：

- 单实例运行
- 伪分布式
- 完全分布式

本次主要介绍单实例和伪分布式Hadoop的安装以及使用简介。首先需要先配置hadoop的环境变量。

编辑`~/.profile`文件，在文件结尾处增加以下内容：

```bash
# hadoop install env settings
HADOOP_INSTALL=/usr/local/src/hadoop-3.1.0
PATH=$HADOOP_INSTALL/bin:$HADOOP_INSTALL/sbin:$PATH
export HADOOP_INSTALL PATH
```

## 单机模式

单机模式（**standalone**）是Hadoop的默认模式。当首次解压Hadoop的源码包时，Hadoop无法了解硬件安装环境，便保守地选择了最小配置。在这种默认模式下所有3个XML文件均为空。当配置文件为空时，Hadoop会完全运行在本地。因为不需要与其他节点交互，单机模式就不使用HDFS，也不加载任何Hadoop的守护进程。该模式主要用于开发调试MapReduce程序的应用逻辑。

> 此程序一般不建议安装，网络上很少这方面资料

```bash
cd /usr/local/src
tar -xzvf hadoop-3.1.0.tar.gz
```

解压完成之后测试安装是否正常：执行命令`hadoop`，正常情况应该显示hadoop的命令使用文档。

```bash
hadoop
hadoop version
```

也可以运行MapReduce任务，执行如下命令：

```bash
cd hadoop-3.1.0/
mkdir input
cp etc/hadoop/*.xml input
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.0.jar grep input output 'dfs[a-z.]+'
```

执行完成之后可以发现output文件夹中生成了两个文件`part-r-00000`和`_SUCCESS`，其中`part-r-00000`文件中记录着在input目录中的所有xml文件中上述正则表达式匹配成功的单词的数量。可以使用以下命令检查结果是否正确。

```
grep -5 --color "dfs" *.xml
```

在hadoop3.1版本中的所有xml文件中只有`dfsadmin`这个单词出现了一次。

## 伪分布模式

伪分布模式（**Pseudo-Distributed Mode**）在“单节点集群”上运行Hadoop，其中所有的守护进程都运行在同一台机器上。该模式在单机模式之上增加了代码调试功能，允许你检查内存使用情况，HDFS输入输出，以及其他的守护进程交互。

> namenode，datanode，secondarynamenode，jobtracer，tasktracer这5个进程，都能在集群上看到。

伪分布式模式只需要在单机模式的基础上改两个配置文件并且格式化namenode即可。

编辑文件`etc/hadoop/core-site.xml`：

```xml
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/usr/local/src/hadoop-3.1.0/tmp</value>
        <description>Abase for other temporary directories.</description>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

编辑文件`etc/hadoop/hdfs-site.xml`：

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/src/hadoop-3.1.0/tmp/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/local/src/hadoop-3.1.0/tmp/dfs/data</value>
    </property>
</configuration>
```

> Hadoop配置文件说明：
>
> Hadoop 的运行方式是由配置文件决定的（运行 Hadoop 时会读取配置文件），因此如果需要从伪分布式模式切换回单机模式，需要删除 core-site.xml 中的配置项。
>
> 此外，伪分布式虽然只需要配置 fs.defaultFS 和 dfs.replication 就可以运行（官方教程如此），不过若没有配置 hadoop.tmp.dir 参数，则默认使用的临时目录为 /tmp/hadoo-hadoop，而这个目录在重启时有可能被系统清理掉，导致必须重新执行 format 才行。所以我们进行了设置，同时也指定 dfs.namenode.name.dir 和 dfs.datanode.data.dir，否则在接下来的步骤中可能会出错。

配置完成后，执行 namenode  的格式化：

```bash
hdfs namenode -format
```

然后使用`start-dfs.sh`命令启动NameNode daemon进程和DataNode daemon进程：

在启动前需要修改`etc/hadoop/hadoop-env.sh`文件中的`JAVA_HOME`变量，改为实际的即可：

```bash
# Many of the options here are built from the perspective that users
# may want to provide OVERWRITING values on the command line.
# For example:
#
#  JAVA_HOME=/usr/java/testing hdfs dfs -ls
JAVA_HOME=/usr/local/src/jdk1.8.0_162
#
# Therefore, the vast majority (BUT NOT ALL!) of these defaults
# are configured for substitution and not append.  If append
# is preferable, modify this file accordingly.
```

启动完成后，可以通过命令 `jps` 来判断是否成功启动，若成功启动则会列出如下进程: “NameNode”、”DataNode” 和 “SecondaryNameNode”（如果 SecondaryNameNode 没有启动，请运行 sbin/stop-dfs.sh 关闭进程，然后再次尝试启动尝试）。如果没有 NameNode 或 DataNode ，那就是配置不成功，请仔细检查之前步骤，或通过查看启动日志排查原因。

启动成功之后，浏览NameNode的web接口，Web界面示例地址如下（hadoop2.x端口默认为50070，hadoop3.x端口默认为9870）：

```
http://120.77.239.67:9870
```

上面的单机模式，grep 例子读取的是本地数据，伪分布式读取的则是 HDFS 上的数据。要使用 HDFS，首先需要在 HDFS 中创建用户目录：

```bash
hdfs dfs -mkdir -p /user/hadoop
```

接着将 `etc/hadoop` 中的 xml 文件作为输入文件复制到分布式文件系统中，即将 `/usr/local/src/hadoop-3.1.0/etc/hadoop` 目录下的xml文件复制到分布式文件系统中的 /user/hadoop/input 中。我们使用的是 hadoop 用户，并且已创建相应的用户目录 /user/hadoop ，因此在命令中就可以使用相对路径如 input，其对应的绝对路径就是 /user/hadoop/input：

```bash
hdfs dfs -mkdir input
hdfs dfs -put ./etc/hadoop/*.xml input
```

复制完成后，可以通过如下命令查看文件列表：

```
hdfs dfs -ls input
```

伪分布式运行 MapReduce 作业的方式跟单机模式相同，区别在于伪分布式读取的是HDFS中的文件（可以将单机步骤中创建的本地 input 文件夹，输出结果 output 文件夹都删掉来验证这一点）。

```bash
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.0.jar grep input output 'dfs[a-z.]+'
```

查看运行结果的命令（查看的是位于 HDFS 中的输出结果）：

```bash
hdfs dfs -cat output/*
```

结果如下，注意到刚才我们已经更改了配置文件，所以运行结果不同。

```
hadoop@iZwz9367lkujh8ulgxc2cwZ:/usr/local/src/hadoop-3.1.0$  hdfs dfs -cat output/*
1	dfsadmin
1	dfs.replication
1	dfs.namenode.name.dir
1	dfs.datanode.data.dir
```

也可以将运行结果取回到本地：

```bash
./bin/hdfs dfs -get output ./output     # 将 HDFS 上的 output 文件夹取回到本机
cat ./output/*
```

Hadoop 运行程序时，输出目录不能存在，否则会提示错误org.apache.hadoop.mapred.FileAlreadyExistsException: Output directory hdfs://localhost:9000/user/hadoop/output already exists，因此若要再次执行，需要执行如下命令删除 output 文件夹：

```bash
hdfs dfs -rm -r output
```

若要关闭 Hadoop，则运行

```
stop-dfs.sh
```

下次启动 hadoop 时，无需进行 NameNode 的初始化，只需要运行 `start-dfs.sh` 就可以！

---

参考：

- https://blog.csdn.net/muyi_amen/article/details/62423649
- https://hadoop.apache.org/docs/r1.0.4/cn/quickstart.html
- http://www.aboutyun.com/thread-6839-1-1.html
- https://www.jianshu.com/p/e450fe10d003
- http://dblab.xmu.edu.cn/blog/install-hadoop/