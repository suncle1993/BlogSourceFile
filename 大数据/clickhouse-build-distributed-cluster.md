---
abbrlink: 847397348
alias: 2019/07/27/clickhouse-build-distributed-cluster/index.html
categories:
- 大数据
date: '2019-07-27T17:13:16'
description: ''
tags:
- clickhouse
- 分布式
- 搭建
title: Clickhouse分布式集群搭建
---









# 目标：1分片2副本的集群

所以需要两台机器，分别是：172.31.59.118|172.31.40.79


安装参考dockerfile:

* https://hub.docker.com/r/yandex/clickhouse-client/dockerfile
* https://hub.docker.com/r/yandex/clickhouse-server/dockerfile

<!--more-->

# 安装步骤

Amazon linux 2是centos系的，使用的yum系的安装方式

* [GitHub - Altinity/clickhouse-rpm-install: How to install clickhouse RPM packages](https://github.com/Altinity/clickhouse-rpm-install)

修改机器时区(不需要重启)
```bash
sudo cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

开始安装

```bash
sudo yum install -y curl
sudo yum install -y epel-release
curl -s https://packagecloud.io/install/repositories/altinity/clickhouse/script.rpm.sh | sudo os=centos dist=7 bash  # for Amazon Linux
sudo yum list 'clickhouse*'
sudo yum install -y clickhouse-server clickhouse-client
sudo yum list installed 'clickhouse*'
sudo /etc/init.d/clickhouse-server restart
clickhouse-client
```
安装zookeeper集群，也可以使用现成的，本次使用现成的，配置如下，后续加入配置文件中

```xml
    <zookeeper-servers>
        <node index="1">
            <host>172.31.3.79</host>
            <port>2181</port>
        </node>
        <node index="2">
            <host>172.31.47.229</host>
            <port>2181</port>
        </node>
        <node index="3">
            <host>172.31.53.227</host>
            <port>2181</port>
        </node>
    </zookeeper-servers>
```

设置每台机器clickhouse用户和密码，先生成sha256的密码

```bash
PASSWORD=$(base64 < /dev/urandom | head -c16); echo "$PASSWORD"; echo -n "$PASSWORD" | sha256sum | tr -d '-'
rKbfrrze4PO5xWeN
93626c6535b2817d55eca365d9a00bbe88c63e24fa4941ce8cdaf4c07f4ab4a6
```
添加用户`sudo vim /etc/clickhouse-server/users.xml`
```xml
    <users>
	      <zaihui>
            <password_sha256_hex>93626c6535b2817d55eca365d9a00bbe88c63e24fa4941ce8cdaf4c07f4ab4a6</password_sha256_hex>
            <networks incl="networks" replace="replace">
                <ip>::/0</ip>
            </networks>
            <profile>default</profile>
            <quota>default</quota>
        </zaihui>
    </users>
```
验证
```bash
clickhouse-client -u zaihui --password rKbfrrze4PO5xWeN
clickhouse-client -u zaihui --password rKbfrrze4PO5xWeN
```
修改clickhouse时区配置
```bash
<timezone>Asia/Shanghai</timezone>
```

取消访问来源ip的限制`sudo vim /etc/clickhouse-server/config.xml`

```xml
    <!-- <listen_host>::</listen_host> -->
    <!-- Same for hosts with disabled ipv6: -->
    <listen_host>0.0.0.0</listen_host>

    <!-- Default values - try listen localhost on ipv4 and ipv6: -->
    <!--
    <listen_host>::1</listen_host>
    <listen_host>127.0.0.1</listen_host>
    -->
```
开始配置集群
配置`sudo vim /etc/clickhouse-server/config.xml`

```xml
    <!-- If element has 'incl' attribute, then for it's value will be used corresponding substitution from another file.
         By default, path to file with substitutions is /etc/metrika.xml. It could be changed in config in 'include_from' element.
         Values for substitutions are specified in /yandex/name_of_substitution elements in that file.
      -->
    <include_from>/etc/clickhouse-server/metrika.xml</include_from>
    <!-- ZooKeeper is used to store metadata about replicas, when using Replicated tables.
         Optional. If you don't use replicated tables, you could omit that.

         See https://clickhouse.yandex/docs/en/table_engines/replication/
      -->
```
配置`/etc/clickhouse-server/metrika.xml`，所有机器都一样
```xml
<yandex>
    <clickhouse_remote_servers>
        <ck_cluster>
            <shard>
                <weight>1</weight>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>172.31.59.118</host>
                    <port>9000</port>
                    <user>zaihui</user>
                    <password>rKbfrrze4PO5xWeN</password>
                </replica>
                <replica>
                    <host>172.31.40.79</host>
                    <port>9000</port>
                    <user>zaihui</user>
                    <password>rKbfrrze4PO5xWeN</password>
                </replica>
            </shard>
        </ck_cluster>
    </clickhouse_remote_servers>

    <zookeeper-servers>
        <node index="1">
            <host>172.31.3.79</host>
            <port>2181</port>
        </node>
        <node index="2">
            <host>172.31.47.229</host>
            <port>2181</port>
        </node>
        <node index="3">
            <host>172.31.53.227</host>
            <port>2181</port>
        </node>
    </zookeeper-servers>

    <networks>
        <ip>::/0</ip>
    </networks>

    <clickhouse_compression>
        <case>
            <min_part_size>10000000000</min_part_size>
            <min_part_size_ratio>0.01</min_part_size_ratio>
            <method>lz4</method>
        </case>
    </clickhouse_compression>
</yandex>
```
配置`/etc/clickhouse-server/config.d/macros.xml`，所有机器都不一样，虽然也可以把配置放在metrika.xml中，但是把不同的独立出来更合适
```xml
<yandex>
    <macros>
        <replica>172.31.59.118</replica>
        <shard>01</shard>
        <layer>01</layer>
    </macros>
</yandex>
```
```xml
<yandex>
    <macros>
        <replica>172.31.40.79</replica>
        <shard>01</shard>
        <layer>01</layer>
    </macros>
</yandex>
```

集群配置完成之后重启一下，确保每个机器都能连接成功

>  使用datagrip连接各个机器，全部成功

# 验证集群功能

创建以replica结尾的本地表delphi_membership_properties_replica和分布式表delphi_membership_properties

```sql
create table dm.delphi_membership_properties_replica
(
  membership_id  int,  -- comment '会员id',
  membership_uid String, -- comment '会员uid',
  business_group_id int, -- comment '商户id',
  business_group_uid String , --comment '商户uid',
  business_group_name String, -- comment '商户名',
  business_id Nullable(int), -- comment '门店id',
  business_uid Nullable(String), -- comment '门店uid',
  business_name Nullable(String), -- comment '门店name',
  membership_source String, -- comment '会员入会来源',
  created_at DateTime,
  calendar_date Date,
  last_visited_date Date, -- comment '最近一次访问时间',
  membership_level int, -- comment '会员等级',
  customer_type String, -- comment '会员类型:新会员/忠诚会员/常来会员/淡忘会员/流失会员，根据最后一次访问时间和商户配置计算而来',
  visit_count int, -- comment '到访次数',
  consumptions_count Nullable(int), -- comment '消费次数',
  consumptions_original_amount Nullable(Decimal128(2)), -- comment '消费总金额：原始金额',
  consumptions_amount Nullable(Decimal128(2)), -- comment '消费总金额：实付金额',
  average_consume Nullable(Decimal128(2)), -- comment '平均消费金额：原始金额/消费次数',
  account_id int, -- comment '用户id',
  account_uid String, -- comment '用户uid',
  account_phone String, -- comment '用户手机',
  age Nullable(int), -- comment '年龄',
  birthday Nullable(String), -- comment '生日',
  birthday_month Nullable(int), -- comment '生日月份',
  birthday_day Nullable(int), -- comment '生日天',
  birthday_year Nullable(int), -- comment '生日年',
  zodiac String, -- comment '星座',
  name Nullable(String), -- comment '姓名',
  gender int, -- comment '性别',
  profession Nullable(String), -- comment '职业',
  country Nullable(String), -- comment '国家',
  province Nullable(String), -- comment '省份',
  city Nullable(String), -- comment '城市',
  region Nullable(String), -- comment '商圈',
  head_img_url Nullable(String), -- comment '头像',
  wechat_name Nullable(String), -- comment '微信名',
  wechat_city Nullable(String), -- comment '微信城市',
  wechat_country Nullable(String), -- comment '微信国家',
  wechat_province Nullable(String), -- comment '微信省份',
  wechat_head_img_url Nullable(String), -- comment '微信头像',
  wechat_groupid int, -- comment '微信组',
  wechat_remark Nullable(String), -- comment '微信备注'
  insert_time DateTime DEFAULT now(), -- 数据插入时间
  insert_date Date DEFAULT toDate(now()) -- 数据插入日期
)
ENGINE = ReplicatedMergeTree('/clickhouse/tables/{layer}-{shard}/delphi_membership_properties_replica', '{replica}')
order by (business_group_uid, calendar_date, created_at, membership_uid);

create table dm.delphi_membership_properties as dm.delphi_membership_properties_replica
ENGINE = Distributed(ck_cluster, dm, delphi_membership_properties_replica, rand())
```
插入数据：在本地表和分布式表插入时在每个replica中都有数据生成
```sql
INSERT INTO dm.delphi_membership_properties_replica (membership_id, membership_uid, business_group_id, business_group_uid, business_group_name, business_id, business_uid, business_name, membership_source, created_at, calendar_date, last_visited_date, membership_level, customer_type, visit_count, consumptions_count, consumptions_original_amount, consumptions_amount, average_consume, account_id, account_uid, account_phone, age, birthday, birthday_month, birthday_day, birthday_year, zodiac, name, gender, profession, country, province, city, region, head_img_url, wechat_name, wechat_city, wechat_country, wechat_province, wechat_head_img_url, wechat_groupid, wechat_remark, insert_time, insert_date) VALUES (3209903, '6735462d5ce444dd8d80763dbcaee746', 2524, '00067f26104445ff89f89820b898af37', '沐沐茶旅', null, null, null, 'third_party', '2017-12-24 16:17:34', '2017-12-24', '2017-12-24', 1, 'Forget', 0, 0, 0.00, 0.00, 0.00, 2754132, 'e3c8b0925460435586f05741eaae548f', '18616566494', null, null, null, null, null, 'Unknown', null, 1, null, '中国', '上海', '黄浦', null, null, 'fengyi', '黄浦', '中国', '上海', 'https://thirdwx.qlogo.cn/mmopen/Ria7DkYdO91HKPibgeJm3Inq3lbbFXlwHJAJMFREYOVibwNCriab41qpVvicm6zd3kZqByBQFC9t9pfMuORQoUIroyicBibicSQIIn0Z/132', 0, '', '2019-07-10 07:50:00', '2019-07-10');

INSERT INTO dm.delphi_membership_properties (membership_id, membership_uid, business_group_id, business_group_uid, business_group_name, business_id, business_uid, business_name, membership_source, created_at, calendar_date, last_visited_date, membership_level, customer_type, visit_count, consumptions_count, consumptions_original_amount, consumptions_amount, average_consume, account_id, account_uid, account_phone, age, birthday, birthday_month, birthday_day, birthday_year, zodiac, name, gender, profession, country, province, city, region, head_img_url, wechat_name, wechat_city, wechat_country, wechat_province, wechat_head_img_url, wechat_groupid, wechat_remark, insert_time, insert_date) VALUES (3226176, '50778ff3ca434edcb113f93af43d646a', 2524, '00067f26104445ff89f89820b898af37', '沐沐茶旅', null, null, null, 'third_party', '2017-12-25 15:44:41', '2017-12-25', '2017-12-25', 1, 'Forget', 0, 0, 0.00, 0.00, 0.00, 2780924, 'c01c3877410144a1965b3ede6e18905c', '13564809560', null, null, null, null, null, 'Unknown', null, 1, null, '中国', '', '', null, null, '骨头™', '', '中国', '', 'https://thirdwx.qlogo.cn/mmopen/VNMic85jx3X5tq6iaBbVY7spB1dWWsWiae5Dz1p2LCsq0mCxps6Zt9sxPjdb7RkribVVElytAmichfx8ibayvC4QmW0g/132', 0, '', '2019-07-10 08:50:11', '2019-07-10');
```
查询数据：停掉一个replica之后仍然能查询出数据

# JDBC连接clickhouse cluster

两种方式，一种是使用clickhouse-jdbc连接集群中的每一个节点，另外一种是使用SLB提供一个对外的统一地址

## 使用BalancedClickhouseDataSource

参考以下clickhouse-jdbc中的代码中的注释：`jdbc:clickhouse://localhost:8123,localhost:8123/database?compress=1&decompress=2`

* [clickhouse-jdbc/BalancedClickhouseDataSource.java at master · yandex/clickhouse-jdbc · GitHub](https://github.com/yandex/clickhouse-jdbc/blob/master/src/main/java/ru/yandex/clickhouse/BalancedClickhouseDataSource.java)
```java
    /**
     * create Datasource for clickhouse JDBC connections
     *
     * @param url address for connection to the database
     *            must have the next format {@code jdbc:clickhouse://<first-host>:<port>,<second-host>:<port>/<database>?param1=value1&param2=value2 }
     *            for example, {@code jdbc:clickhouse://localhost:8123,localhost:8123/database?compress=1&decompress=2 }
     * @throws IllegalArgumentException if param have not correct format, or error happens when checking host availability
     */
    public BalancedClickhouseDataSource(final String url) {
        this(splitUrl(url), getFromUrl(url));
    }

    /**
     * create Datasource for clickhouse JDBC connections
     *
     * @param url        address for connection to the database
     * @param properties database properties
     * @see #BalancedClickhouseDataSource(String)
     */
    public BalancedClickhouseDataSource(final String url, Properties properties) {
        this(splitUrl(url), new ClickHouseProperties(properties));
    }

    /**
     * create Datasource for clickhouse JDBC connections
     *
     * @param url        address for connection to the database
     * @param properties database properties
     * @see #BalancedClickhouseDataSource(String)
     */
    public BalancedClickhouseDataSource(final String url, ClickHouseProperties properties) {
        this(splitUrl(url), properties.merge(getFromUrlWithoutDefault(url)));
    }
```
## 使用SLB

使用LB均衡到各个副本，保证应用方查询单host，本次不使用BalancedClickhouseDataSource，从github issue上看BalancedClickhouseDataSource在之前版本出现副本故障时没能故障转移，不知道是否有修复。

配置LB：**使用标准JDBC连接时需要映射http协议到clickhouse的8123端口(http监听端口)**

验证LB配置是否生效

```bash
echo 'SELECT * from dm.delphi_membership_properties FORMAT Pretty' | curl 'internal-clickhouse-prod-621097858.
cn-north-1.elb.amazonaws.com.cn:80/?' --data-binary @-
```

贴一下在Springboot中使用标准JDBC数据源HikariDataSource÷连接clickhouse的配置：

```java
package com.kezaihui.delphi.core.config;

import com.baomidou.mybatisplus.entity.GlobalConfiguration;
import com.baomidou.mybatisplus.enums.DBType;
import com.baomidou.mybatisplus.enums.IdType;
import com.baomidou.mybatisplus.spring.MybatisSqlSessionFactoryBean;
import com.zaxxer.hikari.HikariDataSource;
import lombok.extern.slf4j.Slf4j;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.annotation.MapperScan;
import org.mybatis.spring.boot.autoconfigure.MybatisProperties;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.core.io.support.ResourcePatternResolver;

import javax.sql.DataSource;

/**
 * clickhouse 数据源配置
 *
 * @author Suncle
 * @date 2019-07-05
 */
@Slf4j
@Configuration
@MapperScan(basePackages = {
    "com.kezaihui.delphi.core.membership.**.mapper"
}, sqlSessionFactoryRef = "ckSqlSessionFactory")
public class CkDataSourceConfig {

    @Autowired
    private MybatisProperties mybatisProperties;

    /**
     * 读取数据源
     *
     * @return javax.sql.DataSource 数据源
     */
    @Bean(name = "ckDataSource")
    @ConfigurationProperties(prefix = "spring.clickhouse.datasource")
    public DataSource dataSource() {
        return new HikariDataSource();
    }

    /**
     * sql 会话工厂配置
     *
     * @param ckDataSource javax.sql.DataSource 数据源
     * @return SqlSessionFactory
     */
    @Bean(name = "ckSqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory(@Qualifier("ckDataSource") DataSource ckDataSource) {
        MybatisSqlSessionFactoryBean bean = new MybatisSqlSessionFactoryBean();
        bean.setDataSource(ckDataSource);

        try {
            GlobalConfiguration configuration = new GlobalConfiguration();
            configuration.setDbType(DBType.OTHER.name());
            configuration.setIdType(IdType.AUTO.getKey());
            configuration.setDbColumnUnderline(true);
            bean.setGlobalConfig(configuration);
            ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
            bean.setMapperLocations(mybatisProperties.resolveMapperLocations());
            bean.setConfigLocation(resolver.getResource(mybatisProperties.getConfigLocation()));
            return bean.getObject();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

配置好之后可以使用mybatis查询数据

# 原生TCP协议连接clickhouse cluster

同样是采用SLB映射到不同的节点上，但是映射的时候需要注意不同之处：

- **映射TCP协议到clickhouse的9000端口**

连接方式可以参考clickhouse-driver的连接，也可以直接使用python语言clickhouse-driver库

# clickhouse python client的选择

官方没有维护各语言的driver，全部由第三方维护，主要有以下两个，对比参见后面。

结论：选择clickhouse-driver，数仓项目使用orm的意义不大。因为应用层不是python项目，是java项目

## clickhouse-driver

 [GitHub - mymarilyn/clickhouse-driver: ClickHouse Python Driver with native interface support](https://github.com/mymarilyn/clickhouse-driver)
活跃度高，star数最高。语法主要是执行原生sql

## infi.clickhouse_orm

[GitHub - Infinidat/infi.clickhouse_orm: A Python library for working with the ClickHouse database (https://clickhouse.yandex/)](https://github.com/Infinidat/infi.clickhouse_orm)
活跃度高，star数第二高。是一个为clickhouse封装的orm框架，写起来有django的感觉

---

参考：

- https://github.com/jneo8/clickhouse-setup

- https://clickhouse.yandex/docs/zh/operations/table_engines/replication/

- https://clickhouse.yandex/docs/zh/operations/table_engines/distributed/

- https://clickhouse.yandex/docs/en/operations/table_engines/replication/

- https://clickhouse.yandex/tutorial.html

- https://hzkeung.com/2018/06/30/clickhouse-cluster-test
- https://hzkeung.com/2018/06/21/clickhouse-cluster-install
- https://www.jianshu.com/p/383cae967a64

