---
abbrlink: 1653833773
alias: 2019/12/19/record-a-troubleshooting-process-of-ut-hanging-caused-by-mybatis-cache-and-transaction-propagation/index.html
categories:
- java
date: '2019-12-19T19:42:19'
description: ''
tags:
- mybatis
- 事务
- 缓存
title: 记录一次mybatis缓存和事务传播行为导致ut挂的排查过程
---









记录一次mybatis缓存和事务传播行为导致ut挂的排查过程

## 起因

rhea项目有两个ut一直都是挂的，之前也经过几个同事排查过，但是都没有找到解决办法，慢慢的这个问题就搁置了。因为之前负责rhea项目的同事离职，我临时接手了这个项目，刚好最近来了一个新同事在做新的功能开发的时候遇到了这个问题，于是我就接了一个锅，最终证明这个锅很好玩。

rhea是一个典型的使用mybatis orm的springboot项目，我们使用h2内存数据库做单元测试，每个单元测试都在一个事务内，都由Transactional进行注解。testGetBGWechatAccountByOpenid这个ut的核心调用链如下

![调用链](https://flowsnow.oss-cn-shanghai.aliyuncs.com/image/tech/troubleshoot-a-problem-caused-by-mybatis-cache/call-chain.jpg)

<!--more-->

调用深度较深，并且有多处使用到了事务，其中`BasePlatformUserService.insert`这个方法用到了`Propagation.REQUIRES_NEW`，也就是图中最右边的这个链路中最终插入了一个PlatformUser

ut代码如下：

```java
    @Test
		@Transactional
    public void testGetBGWechatAccountByOpenid() {
        OpenidRo openidRo = OpenidRo.builder()
            .openid(openidZmall)
            .appId(appIdZmall)
            .unionid(unionid)
            .openAppId(openAppId)
            .platformCategory(PlatformCategoriesEnum.Zmall.getValue())
            .service(ServicesEnum.Server.getValue())
            .serviceBusinessGroupId(serviceBusinessGroup2)
            .alived(false)
            .build();
        RheaAccount rheaAccount = platformUserService.getAccountByOpenId(openidRo);
        Assert.assertEquals(rheaAccount.getPhone(), phone2);

        RheaPlatformUser platformUser = platformUserMapper.getByOpenIdAndBG(
            openidZmall, appIdZmall, serviceBusinessGroup2, ServicesEnum.Server.getValue());
        Assert.assertEquals(rheaAccount.getId(), platformUser.getAccountId());

    }
```

但是在ut里面使用getByOpenIdAndBG查询platformUser却是null导致最终platformUser.getAccountId()这个方法抛出了NPE。

## 知识储备

排查这个问题会用到以下两个知识点

- 事务传播行为-Propagation
- mybatis缓存
- 事务和mybatis Session的关联

### 事务传播行为

Springboot的Transactional的实现包含两部分，一个部分是事务传播行为，一个部分是数据库隔离级别，代码如下：

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {
    @AliasFor("transactionManager")
    String value() default "";

    @AliasFor("value")
    String transactionManager() default "";

    Propagation propagation() default Propagation.REQUIRED;

    Isolation isolation() default Isolation.DEFAULT;
    
    ...
```

数据库隔离级别默认是Isolation.DEFAULT，也就是使用数据库自身的隔离级别，Mysql的默认隔离级别是REPEATABLE_READ可重复读，Oracle的默认事务隔离级别是读已提交READ_COMMITTED。具体的隔离级别不在此讨论。我们需要关注事务的传播行为，也就是Propagation。Propagation实现如下：

```java
public enum Propagation {
    REQUIRED(0),
    SUPPORTS(1),
    MANDATORY(2),
    REQUIRES_NEW(3),
    NOT_SUPPORTED(4),
    NEVER(5),
    NESTED(6);

    private final int value;

    private Propagation(int value) {
        this.value = value;
    }

    public int value() {
        return this.value;
    }
}
```

这里我们只是用到了REQUIRED和REQUIRED_NEW，REQUIRED也是默认的传播行为，这两个传播行为的区别在于：

- REQUIRED：默认的spring事务传播级别，使用该级别的特点是，如果上下文中已经存在事务，那么就加入到事务中执行，如果当前上下文中不存在事务，则新建事务执行。所以这个级别通常**能满足处理大多数的业务场景**。
- REQUIRED_NEW：从字面即可知道，new，每次都要一个新事务，该传播级别的特点是，每次都会新建一个事务，并且同时将上下文中的事务挂起，执行当前新建事务完成以后，上下文事务恢复再执行。
  - 只有在被调用方法中的数据库操作需要保存到数据库中，而不管覆盖事务的结果如何时，才应该使用 `REQUIRES_NEW` 事务属性
  - 举个栗子：假设尝试的所有股票交易都必须被记录在一个审计数据库中。出于验证错误、资金不足或其他原因，不管交易是否失败，这条信息都需要被持久化。如果没有对审计方法使用 `REQUIRES_NEW` 属性，审计记录就会连同尝试执行的交易一起回滚。使用 `REQUIRES_NEW` 属性可以确保不管初始事务的结果如何，审计数据都会被保存

### mybatis缓存

Mybatis-config.xml中可以配置mybatis的本地缓存范围localCacheScope。

mybatis官网解释：MyBatis 利用本地缓存机制（Local Cache）防止循环引用（circular references）和加速重复嵌套查询。 默认值为 SESSION，这种情况下会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地会话仅用在语句执行上，对相同 SqlSession 的不同调用将不会共享数据。

用白话解释：

- SESSION范围的缓存：在同一个SqlSession中多次查询会缓存的mapper中的方法，经过验证，key是单个查询方法
  - 连续查询则后续的查询会使用第一个查询的缓存结果——debug时无法找到查询的sql日志
  - 间断的查询则会实际执行每个查询操作——可以找到每个查询的sql日志
  - 连续的定义是：在当前Session中执行DML操作，或者开启了其他Session执行了DML操作，都认为是连续
- STATEMENT范围的缓存：本质是不使用缓存

在新版本的mysql中数据库自身有自己的缓存，我们并不需要Mybatis的缓存，而且Mybatis不是最底层的缓存，因为多个Session的存在，往往导致一些问题。

修改mybatis的默认缓存范围可以在Mybatis-config.xml中加入以下配置：

```xml
<!--设置缓存作用域，它决定是否使用mybatis的缓存。 系统默认值是SESSION，为了不使用mybatis缓存，设置为STATEMENT -->
<setting name="localCacheScope" value="STATEMENT"/>
```

使用以下配置可以打印出mybatis执行时的操作log和sql语句：

```xml
<setting name="logImpl" value="STDOUT_LOGGING" />
```

### 事务和mybatis Session的关联

开启一个新的事务并且在新的事务中首次执行mybatis操作时会开启新的mybatis Session，因此在`REQUIRES_NEW`中执行mybatis操作一定会开启新的Session

## 排查过程

1. 确保mapper方法对应的sql是对的
2. 将使用`REQUIRES_NEW`的方法改为默认的`REQUIRED`，发现能查询到platformUser
3. 在ut中使用其他方法查询插入的platformUser，发现能查询到
4. mybatis配置加上日志，debug发现ut中的查询platformUserMapper.getByOpenIdAndBG发现没有打印sql
5. 猜测可能是查询是用了mybatis缓存，取消缓存发现可以查询到真实记录
6. 分析：`REQUIRES_NEW`开启的新事务中开启的新Session插入的记录并没有打破老Session缓存的查询结果，因此在老Session中使用相同的查询语句是查询不到真实记录的

具体的debug日志如下：

![](https://flowsnow.oss-cn-shanghai.aliyuncs.com/image/tech/troubleshoot-a-problem-caused-by-mybatis-cache/mybatis-sql-session-log.jpg)

红框中的就是最外层的事务开启的老session，绿色框是中间`REQUIRES_NEW`新事务中开启的新Session。所以对于红框这个Session而言，它并不知道已经发生了DML操作，因此在后续继续查询时会使用最开始的查询结果，也就是null。

这种问题通常发生在getOrCreate操作中。

## 解决

去掉Mybatis层面的缓存

```xml
<!--设置缓存作用域，它决定是否使用mybatis的缓存。 系统默认值是SESSION，为了不使用mybatis缓存，设置为STATEMENT -->
<setting name="localCacheScope" value="STATEMENT"/>
```

解决这个问题对于`REQUIRES_NEW`这个传播行为的理解就更深刻了。

---

参考：

1. 了解事务陷阱：https://www.ibm.com/developerworks/cn/java/j-ts1.html
2. Spring五个事务隔离级别和七个事务传播行为：https://blog.csdn.net/caoxiaohong1005/article/details/79984912
3. Innodb中的事务隔离级别和锁的关系：https://tech.meituan.com/2014/08/20/innodb-lock.html
4. Mybatis XML配置：https://mybatis.org/mybatis-3/zh/configuration.html