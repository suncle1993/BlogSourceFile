---
categories:
  - 大数据
date: '2018-05-03T16:54:05'
description: ''
tags:
  - 集群
  - 搭建
  - YARN
  - Hadoop
title: 搭建Hadoop3集群
---



强烈建议再搭建hadoop集群之前体验一下单机模式和伪分布式模式的搭建过程，可以参考以下链接：

- https://suncle.me/2018/04/16/Hadoop3-basic-installation-and-configuration/

# 开始之前

本次集群搭建所依赖的软件环境如下：

1. Linux系统：以运行在阿里云ECS上的Ubuntu 16.04 LTS版本为例
2. jdk-8u162-linux-x64.tar.gz
3. hadoop 3.1.0

先了解一个概念：

> **Hadoop YARN**： YARN是一个在所有节点上执行数据处理任务的作业调度框架。

然后执行以下初始步骤：

1. 创建三台阿里云ECS，也可以在本地创建3台配置较好的Vmware虚拟机。分别作为hadoop集群的node-master，node1和node2（名称可以自取）。 ~~建议将每个主机名设置为节点名~~ ，一定要修改hostname。
2. 为每台机器创建hadoop用户，后续如没有特殊说明，所有命令均在hadoop用户下执行。
3. 在三台机器上都安装jdk，统一使用hadoop用户安装在`/usr/local/src`目录下（其他目录也可，放在用户目录下会更好，省掉权限问题），更改`/usr/local/src`目录的属主和属组为hadoop，可以使用`chown hadoop:hadoop /usr/local/src`命令更改。
4. 需要在各个节点的/bin目录下增加java可执行文件的软连接，以node2为例

```shell
hadoop@node2:~$ cd /bin
hadoop@node2:/bin$ sudo ln -s /usr/local/src/jdk1.8.0_162/bin/java java
```

如果没有添加，在执行MR程序时会报错：`/bin/bash: /bin/java: No such file or directory`

创建hadoop用户和安装jdk的步骤参见文章开头的单机和伪分布式搭建过程。

下面是本次集群安装的三台ECS机器的ip情况：

- **node-master**: 120.77.239.67
- **node1**: 119.23.145.73
- **node2**: 119.23.141.223

<!--more-->

# Hadoop集群架构

在配置主从节点之前，了解Hadoop集群的不同组件是非常重要的。

主节点保存有关分布式文件系统的信息，例如ext3文件系统上的inode表，并调度资源分配。 此次搭建过程中node-master即为主节点，并运行两个守护进程：

- **NameNode**：管理分布式文件系统并知道集群内存储的数据块的位置。
- **ResourceManager**：管理YARN作业，监管从节点上的调度进程和执行进程。

从节点存储实际数据并提供处理能力来运行作业。分别是node1和node2，并运行两个守护进程：

- **DataNode**：管理物理存储在节点上的实际数据。
- **NodeManager**：管理节点上任务的执行。

# 配置集群

## 在每个节点上创建主机文件

 要想使用节点名称通信，需要编辑`/etc/hosts`文件以添加三台服务器的IP地址。

```
120.77.239.67     node-master
119.23.145.73     node1
119.23.141.223    node2
```

相当于给ip取名称。

## 修改所有节点hostname文件

**这一步骤一定要操作**：以管理节点为例进行操作

```
sudo vim /etc/hostname
```

替换掉其中已有的`hostname`，写入`node-master`，和上述hosts文件中保持一致即可。

如果这个步骤不修改则会在后续集群中执行MapReduce程序过程中出现以下错误：

```
2018-05-08 19:50:46,481 ERROR org.apache.hadoop.yarn.server.resourcemanager.scheduler.SchedulerApplicationAttempt: Error trying to assign container token and NM token to an updated container container_1525778560515_0005_01_000001
java.lang.IllegalArgumentException: java.net.UnknownHostException: iZwz99xn3877js1s191xp9Z
        at org.apache.hadoop.security.SecurityUtil.buildTokenService(SecurityUtil.java:445)
        at org.apache.hadoop.yarn.event.AsyncDispatcher.dispatch(AsyncDispatcher.java:197)
        at org.apache.hadoop.yarn.event.AsyncDispatcher$1.run(AsyncDispatcher.java:126)
        at java.lang.Thread.run(Thread.java:748)
Caused by: java.net.UnknownHostException: iZwz99xn3877js1s191xp9Z
```

意思是管理节点无法识别从节点的hostname，因为在管理节点的hosts文件中对应的是node2，而不是node2的真是hosname，也就是iZwz99xn3877js1s191xp9Z。因此一定要修改hostname。

参考：http://www.voidcn.com/article/p-dsepxqfl-pz.html

## 为Hadoop用户分配认证密钥对

主节点将使用ssh协议通过密钥对认证连接到其他节点，以管理群集。

以hadoop用户身份登录到node-master，并生成一个ssh-key（如果执行已生成过ssh-key则会提示重复，是否需要重写，此时忽略即可）：

```shell
ssh-keygen -b 4096
```

将密钥复制到其他节点。 将密钥复制到节点主机本身也是一种很好的做法，这样您可以根据需要将它用作DataNode。 输入以下命令，并在询问时输入hadoop用户的密码。 如果提示是否将密钥添加到已知主机，请输入yes：

```shell
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@node-master
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@node1
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@node2
```

## 下载hadoop安装包并上传

以hadoop用户身份登录到node-master，将下载好的安装包上传并解压：

```shell
cd /usr/local/src
tar -xzvf jdk-8u162-linux-x64.tar.gz
```

## 设置hadoop环境变量

编辑`~/.profile`文件并写入以下内容：

```shell
# hadoop install env settings
HADOOP_INSTALL=/usr/local/src/hadoop-3.1.0
PATH=$HADOOP_INSTALL/bin:$HADOOP_INSTALL/sbin:$PATH
export HADOOP_INSTALL PATH
```

# 配置管理节点

配置将在node-master上完成并复制到其他节点。

## 设置hadoop依赖的java环境变量

修改`/usr/local/src/hadoop-3.1.0/etc/hadoop/hadoop-env.sh`文件中的`JAVA_HOME`变量，改为实际的即可：

```shell
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

## 配置core-site.xml

在master主机上配置hdfs地址，注意和伪分布式的略微不同，需要直接指定master节点所在的地址。在`/usr/local/src/hadoop-3.1.0/etc/hadoop/core-site.xml`文件中写入以下内容：

```xml
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/usr/local/src/hadoop-3.1.0/tmp</value>
        <description>Abase for other temporary directories.</description>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://node-master:9000</value>
    </property>
</configuration>
```

## 配置hdfs-site.xml

配置副本的个数及数据的存放路径，在`/usr/local/src/hadoop-3.1.0/etc/hadoop/hdfs-site.xml`文件中写入：

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
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

其中：

- `dfs.replication` 表示数据块的副本数量，指示数据在集群中的复制次数。 您可以设置2以将所有数据复制到两个节点上。 不要设置高于实际节点数量的值。
- `dfs.namenode.name.dir` 元数据存放路径
- `dfs.datanode.data.dir` 数据节点存放路径

## 配置mapred-site.xml

设置YARN为作业调度器，也就是默认的MapReduce框架，在`/usr/local/src/hadoop-3.1.0/etc/hadoop/mapred-site.xml`文件中写入：

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

## 配置yarn-site.xml文件

在`/usr/local/src/hadoop-3.1.0/etc/hadoop/yarn-site.xml`文件中写入：

```xml
<configuration>
<!-- Site specific YARN configuration properties -->
  <property>
    <name>yarn.resourcemanager.address</name>
    <value>node-master:18040</value>
  </property>
  <property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>node-master:18030</value>
  </property>
  <property>
    <name>yarn.resourcemanager.webapp.address</name>
    <value>node-master:18088</value>
  </property>
  <property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>node-master:18025</value>
  </property>
  <property>
    <name>yarn.resourcemanager.admin.address</name>
    <value>node-master:18141</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>
  <property>
    <name>yarn.app.mapreduce.am.env</name>
    <value>HADOOP_MAPRED_HOME=/usr/local/src/hadoop-3.1.0</value>
  </property>
  <property>
    <name>mapreduce.map.env</name>
    <value>HADOOP_MAPRED_HOME=/usr/local/src/hadoop-3.1.0</value>
  </property>
  <property>
    <name>mapreduce.reduce.env</name>
    <value>HADOOP_MAPRED_HOME=/usr/local/src/hadoop-3.1.0</value>
  </property
</configuration>
```

注意修改的各个`value`需要和`/etc/hosts`中的名称保持一致。

这三项配置一定要有：`yarn.app.mapreduce.am.env` `mapreduce.map.env` `mapreduce.reduce.env`否则在执行MR程序时会直接报错（hadoop3.1中已验证）。

具体错误参考：https://stackoverflow.com/questions/47599789/hadoop-pagerank-error-when-running

## 配置workers文件

列出所有workers的主机名。在`/usr/local/src/hadoop-3.1.0/etc/hadoop/workers`文件中写入：

```
node-master
node1
node2
```

注意：

1. hadoop2.x配置的是slaves文件，这里有所改变。
2. 此处的worker中写入了管理节点，因此启动HDFS之后也会在管理节点所在机器创建一个DataNode。如果不想在管理节点机器中开启DataNode，则删除workers文件中的node-master配置。

此外，如果想在Hadoop集群中动态增加和删除节点，则更改此文件即可。

# 配置内存分配

内存分配在低RAM节点上可能会很棘手，因为默认值不适用于RAM少于8GB的节点，因此在使用sqoop等命令时调用的MapReduce程序会有如下类似的报错：

```
Application is added to the scheduler and is not yet activated. Queue's AM resource limit exceeded. Details : AM Partition = <DEFAULT_PARTITION>; AM Resource Request = <memory:2048, vCores:1>; Queue Resource Limit for AM = <memory:3072, vCores:1>; User AM Resource Limit of the queue = <memory:3072, vCores:1>; Queue AM Resource Usage = <memory:2048, vCores:1>;
```

这里将重点介绍如何为MapReduce作业分配内存，因为此次使用的ECS机器是4GB内存，因此为4GB RAM节点提供示例配置。

## 内存分配属性

YARN作业执行需要使用以下两种资源：

- Application Master (AM) ：负责监视应用程序并协调集群中的分布式执行程序。
- Executors：一些由AM创建的Executors，用于真正的运行该作业。 对于MapReduce作业，executors会并行的执行map和reduce操作。

两者都在从节点的容器中运行。 每个从节点都运行一个NodeManager守护进程，负责在节点上创建容器。 整个集群由一个ResourceManager管理，它根据容量要求和当前使用情况调度所有所有从节点上的容器分配。

需要正确配置四种类型的资源分配才能使群集正常工作。分别是：

1. 可以为单个节点上的YARN容器分配的内存大小。 这个限制应该高于其他所有的限制; 否则，容器分配会被拒绝，应用程序失败。 但是，它不应该是节点上的全部RAM。

> 这个值在`yarn-site.xml`中配置`yarn.nodemanager.resource.memory-mb`属性

2. 单个容器可以消耗的内存大小以及允许的最小内存分配量。 一个容器永远不会超过最大容量，否则分配将失败，并且总是以最小分配量的倍数进行RAM分配。

> 这些值在`yarn-site.xml`中配置`yarn.scheduler.maximum-allocation-mb`和`yarn.scheduler.minimum-allocation-mb`属性。

3. 分配给ApplicationMaster的内存大小。 是一个适合容器最大尺寸的常数值。

> 这个值在`mapred-site.xml`中配置`yarn.app.mapreduce.am.resource.mb`属性。

4. 分配给map和reduce操作的内存大小。应该小于最大尺寸。

> 这是在`mapred-site.xml`中配置的，其属性为`mapreduce.map.memory.mb`和`mapreduce.reduce.memory.mb`。

具体配置参数可以参见：https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html

## 各内存大小计算方式

下载内存计算脚本：

```shell
wget https://raw.githubusercontent.com/mahadevkonar/ambari-yarn-utils/master/yarn-utils/yarn-utils.py
```

使用方法：

```
python yarn-utils.py -c 16 -m 64 -d 4 -k True
```

- -c选项：cpu核数
- -m选项：内存大小
- -d选项：机器上的磁盘数量
- -k选项：如果安装了HBase则设置为True，否则为False

> 其中：Core的数量可以通过`nproc`命令计算；内存大小可以通过`free -m`命令来计算需要换算为G;磁盘的数量可以通过`lsblk -s`或`sudo fdisk -l`命令来查看。

计算完成之后，最后的脚本执行命令为：

```shell
hadoop@node-master:~$ python yarn-utils.py -c 2 -m 8 -d 1 -k False
 Using cores=2 memory=8GB disks=1 hbase=False
 Profile: cores=2 memory=6144MB reserved=2GB usableMem=6GB disks=1
 Num Container=3
 Container Ram=2048MB
 Used Ram=6GB
 Unused Ram=2GB
 yarn.scheduler.minimum-allocation-mb=2048
 yarn.scheduler.maximum-allocation-mb=6144
 yarn.nodemanager.resource.memory-mb=6144
 mapreduce.map.memory.mb=1024
 mapreduce.map.java.opts=-Xmx819m
 mapreduce.reduce.memory.mb=2048
 mapreduce.reduce.java.opts=-Xmx1638m
 yarn.app.mapreduce.am.resource.mb=1024
 yarn.app.mapreduce.am.command-opts=-Xmx819m
 mapreduce.task.io.sort.mb=409
```

## 8GB节点的示例配置

| 属性                                 | 值   |
| ------------------------------------ | ---- |
| yarn.nodemanager.resource.memory-mb  | 6144 |
| yarn.scheduler.maximum-allocation-mb | 6144 |
| yarn.scheduler.minimum-allocation-mb | 2048 |
| yarn.app.mapreduce.am.resource.mb    | 1024 |
| mapreduce.map.memory.mb              | 1024 |
| mapreduce.reduce.memory.mb           | 2048 |

编辑 `/usr/local/src/hadoop-3.1.0/etc/hadoop/yarn-site.xml` 文件，并增加以下行：

```xml
  <property>
    <name>yarn.nodemanager.resource.memory-mb</name>
    <value>6144</value>
  </property>
  <property>
    <name>yarn.scheduler.maximum-allocation-mb</name>
    <value>6144</value>
  </property>
  <property>
    <name>yarn.scheduler.minimum-allocation-mb</name>
    <value>2048</value>
  </property>
  <property>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
  </property>
```

编辑`/usr/local/src/hadoop-3.1.0/etc/hadoop/mapred-site.xml`文件，并增加以下行：

```xml
  <property>
    <name>yarn.app.mapreduce.am.resource.mb</name>
    <value>1024</value>
  </property>
  <property>
    <name>mapreduce.map.memory.mb</name>
    <value>1024</value>
  </property>
  <property>
    <name>mapreduce.reduce.memory.mb</name>
    <value>2048</value>
  </property>
```

# 配置从节点

复制hadoop的压缩包到所有从节点（也可以使用ftp手动上传）：

```shell
scp hadoop-3.1.0.tar.gz hadoop@node1:/usr/local/src/
scp hadoop-3.1.0.tar.gz hadoop@node2:/usr/local/src/
```

使用hadoop用户连接到所有的从节点，解压安装包：

```shell
cd /usr/local/src
tar -xzvf jdk-8u162-linux-x64.tar.gz
```

复制主节点的所有hadoop配置文件到各从节点中：

```shell
scp /usr/local/src/hadoop-3.1.0/etc/hadoop/* hadoop@node1:/usr/local/src/hadoop-3.1.0/etc/hadoop/
scp /usr/local/src/hadoop-3.1.0/etc/hadoop/* hadoop@node2:/usr/local/src/hadoop-3.1.0/etc/hadoop/
```

# 格式化HDFS

HDFS需要像任何传统文件系统一样格式化。 在node-master上，运行以下命令：

```shell
hdfs namenode -format
```

# 启动停止HDFS

通过从node-master运行以下脚本启动HDFS：

```
start-dfs.sh
```

这个命令会启动node-master上的NameNode和SecondaryNameNode，并且根据node1和node2上的配置文件分别启动node1和node2的DataNode。

使用jps命令检查每个节点上的进程是否启动：

```
24053 SecondaryNameNode
23721 NameNode
23850 DataNode
24205 Jps
```

（如果node-master上也启动了一个DataNode那么在node-master上也能看到NodeManager）

在node1上jps结果如下：

```
27387 Jps
27311 DataNode
```

在node2上jps结果如下：

```
1314 Jps
1227 DataNode
```

要停止主节点和从节点上的HDFS，请从node-master运行以下命令：

```shell
stop-dfs.sh
```

在hdfs启动之后，各种hdfs命令就都可以直接在集群上使用。

关于hdfs安全模式的解除：重启机器等操作时会导致hdfs处于安全模式，因此需要用命令解除：

```shell
hdfs dfsadmin -safemode leave
```

# 运行YARN

HDFS是一个分布式存储系统，它不提供任何服务来运行和调度集群中的任务。 这是YARN框架的作用。 以下部分是关于启动，监控和向YARN提交作业。

## 启动停止YARN

运行以下脚本启动：

```shell
start-yarn.sh
```

使用jps命令检查各节点上正在运行的进程。除了前面的HDFS守护进程之外，还应该在node-master上看到ResourceManager，并在node1和node2上看到NodeManager。

要停止YARN，请在node-master上运行以下命令：

```shell
stop-yarn.sh
```

## 监控YARN

yarn命令提供实用的命令套件程序来管理YARN集群。 还可以使用以下命令打印正在运行的节点的报告：

```shell
yarn node -list
```

如果运行错误，需要检查YARN的配置文件`hadoop/yarn-site.xml`是否配置错误。

可以使用以下命令获取正在运行的应用程序的列表：

```shell
yarn application -list
```

要获得yarn命令的所有可用参数，请参阅[Apache YARN文档](https://community.hortonworks.com/content/supportkb/49544/hdfs-client-fails-with-unknownhostexception-when-h.html)

与HDFS一样，YARN提供了一个友好的Web UI，默认端口为8088。 具体端口可通过yarn-site.xml文件里面的yarn.resourcemanager.webapp.address配置。示例地址如下：

```
http://120.77.239.67:18088/cluster
```

## 提交MapReduce作业至YARN

YARN作业被打包成jar文件，并提交给YARN用命令`yarn jar`执行。

---

参考：

- https://suncle.me/2018/04/16/Hadoop3-basic-installation-and-configuration/
- https://www.cnblogs.com/guoyuanwei/p/8583380.html
- https://linode.com/docs/databases/hadoop/how-to-install-and-set-up-hadoop-cluster
- http://www.dajiangtai.com/community/18389.do
- https://www.2cto.com/net/201610/557536.html
