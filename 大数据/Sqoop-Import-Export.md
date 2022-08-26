---
abbrlink: 2633130510
alias: 2018/05/09/Sqoop-Import-Export/index.html
categories:
- 大数据
date: '2018-05-09T15:17:50'
description: ''
tags:
- Sqoop
- HDFS
- Hadoop
title: Sqoop导入导出
---









Apache Sqoop是一种用于在Apache Hadoop和结构化数据存储（如关系数据库）之间高效传输批量数据的工具。实质就是将导入导出命令转换成mapreduce程序来实现。

# Sqoop版本选择

根据官网介绍，当前（文档编写时间：2018-05-07）最新的稳定版本是1.4.7。 Sqoop2的最新版本是1.99.7（下载，文档）。 请注意，1.99.7与1.4.7不兼容，且未完成功能，具体信息可以参见Apache Sqoop官网。因此不适用于生产部署。所以我们选择1.4.7版本。

- 1.4.7版本下载地址：https://www.apache.org/dyn/closer.lua/sqoop/1.4.7

> 可以选择华中科技大学的镜像站进行下载：http://mirrors.hust.edu.cn/apache/sqoop/1.4.7

- 1.4.7版本文档地址：https://sqoop.apache.org/docs/1.4.7/index.html

<!--more-->

# Sqoop安装配置

上传sqoop安装包sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz到/usr/local/src目录中，解压并改名：

```shell
cd /usr/local/src/
tar xvfz sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz
mv sqoop-1.4.7.bin__hadoop-2.6.0 sqoop
```

配置SQOOP_HOME到环境变量中，`vim ~/.profile`，然后写入以下内容（根据实际情况修改）：

```shell
# sqoop install env settings
export SQOOP_HOME=/usr/local/src/sqoop
export PATH=$PATH:$SQOOP_HOME:$SQOOP_HOME/bin
```

配置sqoop-env.sh

```shell
cd /usr/local/src/sqoop/conf
mv sqoop-env-template.sh sqoop-env.sh
```

然后使用 `vim sqoop-env.sh` 命令，打开文件添加如下内容：

```shell
#Set path to where bin/hadoop is available
export HADOOP_COMMON_HOME=/usr/local/src/hadoop-3.1.0

#Set path to where hadoop-*-core.jar is available
export HADOOP_MAPRED_HOME=/usr/local/src/hadoop-3.1.0

#set the path to where bin/hbase is available
#export HBASE_HOME=

#Set the path to where bin/hive is available
#export HIVE_HOME=

#Set the path for where zookeper config dir is
#export ZOOCFGDIR=
```

如果数据读取不涉及hbase和hive，那么相关hbase和hive的配置可以不加；如果集群有独立的zookeeper集群，那么配置zookeeper，反之，不用配置。因为本次主要是使用Sqoop从Mysql导入数据到HDFS和使用Sqoop导出HDFS数据到Mysql，所以不需要配置这三项，但是会出现harmless warnning，不过没影响。

将mysql-connector-java.jar文件复制到sqoop/lib文件夹下：

```shell
cd /usr/local/src/
wget https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-5.1.46.tar.gz
tar xvfz mysql-connector-java-5.1.46.tar.gz
cp mysql-connector-java-5.1.46/mysql-connector-java-5.1.46-bin.jar /usr/local/src/sqoop/lib/
```

**测试运行**

使用vps中的数据库测试，数据库url为ip

```shell
# 列出mysql server中所有的数据库
hadoop@iZwz9367lkujh8ulgxc2cwZ:/usr/local/src/sqoop/lib$ sqoop list-databases --connect jdbc:mysql://138.68.1.61:3306/ --username root -P
2018-05-07 17:05:13,057 INFO sqoop.Sqoop: Running Sqoop version: 1.4.7
Enter password: 
2018-05-07 17:05:15,646 INFO manager.MySQLManager: Preparing to use a MySQL streaming resultset.
information_schema
mysql
performance_schema
wordpress

# 列出数据库中的所有表
hadoop@iZwz9367lkujh8ulgxc2cwZ:/usr/local/src/sqoop/lib$ sqoop list-tables --connect jdbc:mysql://138.68.1.61:3306/wordpress --username root -P
2018-05-07 17:07:37,570 INFO sqoop.Sqoop: Running Sqoop version: 1.4.7
Enter password: 
2018-05-07 17:07:39,948 INFO manager.MySQLManager: Preparing to use a MySQL streaming resultset.
wp_commentmeta
wp_comments
wp_pic_postmeta
wp_pic_posts
...
```

sqoop 命令执行成功，代表安装成功，数据库连接成功。

如果使用阿里云RDS进行连接测试，需要配置RDS和本地的DNS，以便支持阿里云RDS的连接。如果不做配置会有如下报错：

```
2018-05-07 16:54:23,169 ERROR sqoop.Sqoop: Got exception running Sqoop: java.lang.RuntimeException: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure
```

# Sqoop从Mysql导入数据到HDFS

新建Mysql测试表tb_roommate：

```mysql
CREATE TABLE `tb_roommate` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `name_1` VARCHAR(30) NOT NULL COMMENT '姓名',
  `age` TINYINT(3) UNSIGNED NOT NULL COMMENT '年龄',
  `height` TINYINT(3) UNSIGNED NOT NULL COMMENT '身高',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_name` (`name_1`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COMMENT='室友表'
```

插入测试数据：

```mysql
INSERT INTO tb_roommate(name_1, age, height) VALUES('ChenLiang', 24, 182);
INSERT INTO tb_roommate(name_1, age, height) VALUES('NieMing', 23, 173);
INSERT INTO tb_roommate(name_1, age, height) VALUES('LvShaohe', 23, 172);
INSERT INTO tb_roommate(name_1, age, height) VALUES('LiXuyun', 22, 173);
COMMIT;
```

查询待处理结果集：

```mysql
SELECT * FROM tb_roommate WHERE age > 22;
```

数据导入：

```shell
hadoop@node-master:~/workspace$ sqoop import --connect jdbc:mysql://138.68.1.61:3306/wordpress --username root --password XXXXXX --table tb_roommate
Warning: /usr/local/src/sqoop/../hbase does not exist! HBase imports will fail.
Please set $HBASE_HOME to the root of your HBase installation.
Warning: /usr/local/src/sqoop/../hcatalog does not exist! HCatalog jobs will fail.
Please set $HCAT_HOME to the root of your HCatalog installation.
Warning: /usr/local/src/sqoop/../accumulo does not exist! Accumulo imports will fail.
Please set $ACCUMULO_HOME to the root of your Accumulo installation.
Warning: /usr/local/src/sqoop/../zookeeper does not exist! Accumulo imports will fail.
Please set $ZOOKEEPER_HOME to the root of your Zookeeper installation.
2018-05-09 09:47:44,962 INFO sqoop.Sqoop: Running Sqoop version: 1.4.7
2018-05-09 09:47:45,011 WARN tool.BaseSqoopTool: Setting your password on the command-line is insecure. Consider using -P instead.
2018-05-09 09:47:45,202 INFO manager.MySQLManager: Preparing to use a MySQL streaming resultset.
2018-05-09 09:47:45,203 INFO tool.CodeGenTool: Beginning code generation
2018-05-09 09:47:47,328 INFO manager.SqlManager: Executing SQL statement: SELECT t.* FROM `tb_roommate` AS t LIMIT 1
2018-05-09 09:47:48,371 INFO manager.SqlManager: Executing SQL statement: SELECT t.* FROM `tb_roommate` AS t LIMIT 1
2018-05-09 09:47:49,053 INFO orm.CompilationManager: HADOOP_MAPRED_HOME is /usr/local/src/hadoop-3.1.0
Note: /tmp/sqoop-hadoop/compile/6b2ce87c6baaca5f524499832b6b1bdd/tb_roommate.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
2018-05-09 09:47:51,122 INFO orm.CompilationManager: Writing jar file: /tmp/sqoop-hadoop/compile/6b2ce87c6baaca5f524499832b6b1bdd/tb_roommate.jar
2018-05-09 09:47:51,134 WARN manager.MySQLManager: It looks like you are importing from mysql.
2018-05-09 09:47:51,134 WARN manager.MySQLManager: This transfer can be faster! Use the --direct
2018-05-09 09:47:51,134 WARN manager.MySQLManager: option to exercise a MySQL-specific fast path.
2018-05-09 09:47:51,134 INFO manager.MySQLManager: Setting zero DATETIME behavior to convertToNull (mysql)
2018-05-09 09:47:51,813 INFO mapreduce.ImportJobBase: Beginning import of tb_roommate
2018-05-09 09:47:51,814 INFO Configuration.deprecation: mapred.job.tracker is deprecated. Instead, use mapreduce.jobtracker.address
2018-05-09 09:47:52,013 INFO Configuration.deprecation: mapred.jar is deprecated. Instead, use mapreduce.job.jar
2018-05-09 09:47:53,671 INFO Configuration.deprecation: mapred.map.tasks is deprecated. Instead, use mapreduce.job.maps
2018-05-09 09:47:53,816 INFO client.RMProxy: Connecting to ResourceManager at node-master/120.77.239.67:18040
2018-05-09 09:47:54,816 INFO mapreduce.JobResourceUploader: Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/hadoop/.staging/job_1525781821036_0013
2018-05-09 09:50:15,455 INFO db.DBInputFormat: Using read commited transaction isolation
2018-05-09 09:50:15,629 INFO db.DataDrivenDBInputFormat: BoundingValsQuery: SELECT MIN(`id`), MAX(`id`) FROM `tb_roommate`
2018-05-09 09:50:15,804 INFO db.IntegerSplitter: Split size: 0; Num splits: 4 from: 1 to: 4
2018-05-09 09:50:16,198 INFO mapreduce.JobSubmitter: number of splits:4
2018-05-09 09:50:16,237 INFO Configuration.deprecation: yarn.resourcemanager.system-metrics-publisher.enabled is deprecated. Instead, use yarn.system-metrics-publisher.enabled
2018-05-09 09:50:16,817 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1525781821036_0013
2018-05-09 09:50:16,819 INFO mapreduce.JobSubmitter: Executing with tokens: []
2018-05-09 09:50:17,053 INFO conf.Configuration: resource-types.xml not found
2018-05-09 09:50:17,054 INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
2018-05-09 09:50:17,169 INFO impl.YarnClientImpl: Submitted application application_1525781821036_0013
2018-05-09 09:50:17,217 INFO mapreduce.Job: The url to track the job: http://node-master:18088/proxy/application_1525781821036_0013/
2018-05-09 09:50:17,218 INFO mapreduce.Job: Running job: job_1525781821036_0013
2018-05-09 09:50:23,347 INFO mapreduce.Job: Job job_1525781821036_0013 running in uber mode : false
2018-05-09 09:50:23,348 INFO mapreduce.Job:  map 0% reduce 0%
2018-05-09 09:50:32,417 INFO mapreduce.Job:  map 25% reduce 0%
2018-05-09 09:50:41,462 INFO mapreduce.Job:  map 50% reduce 0%
2018-05-09 09:50:50,508 INFO mapreduce.Job:  map 75% reduce 0%
2018-05-09 09:50:59,550 INFO mapreduce.Job:  map 100% reduce 0%
2018-05-09 09:51:00,562 INFO mapreduce.Job: Job job_1525781821036_0013 completed successfully
2018-05-09 09:51:00,647 INFO mapreduce.Job: Counters: 32
	File System Counters
		FILE: Number of bytes read=0
		FILE: Number of bytes written=888404
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=393
		HDFS: Number of bytes written=71
		HDFS: Number of read operations=24
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=8
	Job Counters 
		Launched map tasks=4
		Other local map tasks=4
		Total time spent by all maps in occupied slots (ms)=23644
		Total time spent by all reduces in occupied slots (ms)=0
		Total time spent by all map tasks (ms)=23644
		Total vcore-milliseconds taken by all map tasks=23644
		Total megabyte-milliseconds taken by all map tasks=48422912
	Map-Reduce Framework
		Map input records=4
		Map output records=4
		Input split bytes=393
		Spilled Records=0
		Failed Shuffles=0
		Merged Map outputs=0
		GC time elapsed (ms)=279
		CPU time spent (ms)=3990
		Physical memory (bytes) snapshot=953724928
		Virtual memory (bytes) snapshot=10424242176
		Total committed heap usage (bytes)=560463872
		Peak Map Physical memory (bytes)=262942720
		Peak Map Virtual memory (bytes)=2613026816
	File Input Format Counters 
		Bytes Read=0
	File Output Format Counters 
		Bytes Written=71
2018-05-09 09:51:00,654 INFO mapreduce.ImportJobBase: Transferred 71 bytes in 186.9643 seconds (0.3798 bytes/sec)
2018-05-09 09:51:00,657 INFO mapreduce.ImportJobBase: Retrieved 4 records.
hadoop@node-master:~/workspace$ hdfs dfs -ls tb_roommate
Found 5 items
-rw-r--r--   2 hadoop supergroup          0 2018-05-09 09:50 tb_roommate/_SUCCESS
-rw-r--r--   2 hadoop supergroup         19 2018-05-09 09:50 tb_roommate/part-m-00000
-rw-r--r--   2 hadoop supergroup         17 2018-05-09 09:50 tb_roommate/part-m-00001
-rw-r--r--   2 hadoop supergroup         18 2018-05-09 09:50 tb_roommate/part-m-00002
-rw-r--r--   2 hadoop supergroup         17 2018-05-09 09:50 tb_roommate/part-m-00003
hadoop@node-master:~/workspace$ hdfs dfs -cat tb_roommate/part-m-00000
1,ChenLiang,24,182
hadoop@node-master:~/workspace$ hdfs dfs -cat tb_roommate/part-m-00001
2,NieMing,23,173
hadoop@node-master:~/workspace$ hdfs dfs -cat tb_roommate/part-m-00002
3,LvShaohe,23,172
hadoop@node-master:~/workspace$ hdfs dfs -cat tb_roommate/part-m-00003
4,LiXuyun,22,173
```

将msyql数据库wordpress中的表tb_roommate，导入到hdfs目录，默认会导入到`/user/hadoop/tb_roommate`下，其中tb_roommate为导入的表名。

如果想要数据导入速度更快，可以使用`--direct`模式，sqoop为特定的RDBMS提供直接连接器，因此传输更快

```
sqoop import --connect jdbc:mysql://138.68.1.61:3306/wordpress --username root --password XXXXXX --table tb_roommate --direct
```

**但是需要在每台机器上有一份mysqldump可执行文件，解决办法是复制一份mysqldump文件或者直接在每台机器上安装一个mysql数据库**，如果没有mysqldump，会报如下错误：

```
Error: java.io.IOException: Cannot run program "mysqldump": error=2, No such file or directory
```

如果要想导入到指定的目录，添加一个选项--target-dir：

```shell
sqoop import --connect jdbc:mysql://138.68.1.61:3306/wordpress --username root --password XXXXXX --table tb_roommate --target-dir /output/sqoop/tb_roommate
```

因为默认执行sqoop会有4个maptasks任务，为了满足业务的需要，可以进行修改，只需要在命令后面加一个选项-m：

```shell
sqoop import --connect jdbc:mysql://138.68.1.61:3306/wordpress --username root --password XXXXXX --table tb_roommate --target-dir /output/sqoop/tb_roommate -m 1
```

执行的过程中，如果输出目录已经存在，报错，要想输出到该目录 使用选项--delete-target-dir：

```shell
sqoop import --connect jdbc:mysql://138.68.1.61:3306/wordpress --username root --password XXXXXX --table tb_roommate --target-dir /output/sqoop/tb_roommate -m 1 --delete-target-dir
```

如果想在原来的基础之上追加新的数据，只需要添加一个选项--append,但是注意，--append和--delete-target-dir不能同时存在：

```shell
sqoop import --connect jdbc:mysql://138.68.1.61:3306/wordpress --username root --password XXXXXX --table tb_roommate --target-dir /output/sqoop/tb_roommate -m 1 --append
```

条件导入：

```shell
sqoop import --connect jdbc:mysql://138.68.1.61:3306/wordpress --username root --password XXXXXX --table tb_roommate --target-dir /output/sqoop/tb_roommate -m 1 --append --where "age > 22"
```

通过sql导入：

```shell
sqoop import --connect jdbc:mysql://138.68.1.61:3306/wordpress --username root --password XXXXXX --table tb_roommate --target-dir /output/sqoop/tb_roommate -m 1 --append --query "SELECT id, name_1, age, height FROM tb_roommate WHERE age > 22"
```

# Sqoop导出HDFS数据到Mysql

数据导出到mysql，默认以逗号作为分隔符。导出数据到Mysql之前，表需要已经存在，否则报错

```shell
sqoop export --connect jdbc:mysql://138.68.1.61:3306/wordpress --username root --password XXXXXX --table tb_roommate1 --export-dir /user/hadoop/tb_roommate
```

类似于Mysql duplicate操作，如果存在就更新，不存在就插入：

```shell
sqoop export --connect jdbc:mysql://138.68.1.61:3306/wordpress --username root --password XXXXXX --table tb_roommate1 --export-dir /user/hadoop/tb_roommate -m 2 --update-key id --update-mode allowinsert
```

# 常见错误整理

1. 阿里云RDS连接不上时，先用一个本地的url中只有ip的数据库或者是腾讯Mysql数据库进行测试，确认是否有问题
2. 命令使用过程中的warning信息，需要判断是否是harmless的


---

参考：

- https://sqoop.apache.org/
- https://sqoop.apache.org/docs/1.4.6/SqoopUserGuide.html
- https://www.jianshu.com/p/19ff7effcaf2
- https://www.cnblogs.com/zhangs1986/p/7063592.html
- https://acadgild.com/blog/exporting-files-hdfs-mysql-using-sqoop/
- https://my.oschina.net/sniperLi/blog/687942
- https://www.cnblogs.com/zhangs1986/p/7052621.html
- http://blog.51cto.com/xpleaf/2090584
- https://www.alibabacloud.com/help/zh/doc-detail/28133.htm
- https://yq.aliyun.com/articles/43799