---
abbrlink: 1014846652
alias: 2021/07/27/redis-pfadd-count/index.html
categories:
- 工程实践
date: '2021-07-27T14:56:47'
tags:
- redis
- HyperLogLog
title: redis统计送礼人数
---







在Hago的营收活动中， 我们经常要记录的一个数据是送礼用户数，作为活动对于用户的吸引程度的一个关键指标。

本文将介绍3种使用 Redis 对用户数量进行记录的方案， 这些方案虽然都可以对送礼用户的数量进行统计， 但每个方案都有一些自己特有的操作， 并且各个方案的性能特征以及资源消耗也各有不同。

![](https://suncle-public.oss-cn-shenzhen.aliyuncs.com/uPic/T3LYCH-1627956415425.png)

<!--more-->

## 方案 1 ：使用集合

如果产品需求除了需要记录送礼用户数量，还需要记录送礼用户的名单， 那么可以使用集合来达成这个目标。

可以执行以下 [SADD](http://redisdoc.com/set/sadd.html) 命令， 将用户添加到送礼用户名单当中：

```
SADD "users" <uid>
```

通过使用 [SISMEMBER](http://redisdoc.com/set/sismember.html) 命令， 我们可以检查一个指定的用户是否送过礼：

```
SISMEMBER "users" <uid>
```

而统计送礼用户则可以通过执行 [SCARD](http://redisdoc.com/set/scard.html) 命令来完成：

```
SCARD "users"
```

通过集合运算操作， 我们可以像有序集合方案一样， 对不同时间段或者日期的送礼用户名单进行聚合计算。 比如说， 通过 [SINTER](http://redisdoc.com/set/sinter.html) 或者 [SINTERSTORE](http://redisdoc.com/set/sinterstore.html) 命令， 我们可以计算出一周之内连续送礼的用户：

```
SINTER "day_1_users" "day_2_users" ... "day_7_users"
```

此外， 通过 [SUNION](http://redisdoc.com/set/sunion.html) 命令或者 [SUNIONSTORE](http://redisdoc.com/set/sunionstore.html) 命令， 我们可以计算出一周内连续送礼的用户总数量：

```
SUNION "day_1_users" "day_2_users" ... "day_7_users"
```

而通过执行 [SDIFF](http://redisdoc.com/set/sdiff.html) 命令或者 [SDIFFSTORE](http://redisdoc.com/set/sdiffstore.html) 命令， 我们可以知道哪些用户今天送礼了， 但是昨天没有送礼：

```
SDIFF "today_users" "yesterday_users"
```

又或者工作日送礼了， 但是假日没有送礼：

```
# 计算工作日送礼名单
SINTERSTORE "weekday_users" "monday_users" "tuesday_users" ... "friday_users"
# 计算假日送礼名单
SINTERSTORE "holiday_users" "saturday_users" "sunday_users"
# 计算工作日送礼但是假日未送礼的名单
SDIFF "weekday_users" "holiday_users"
```

etc.

## 方案 2 ：使用 HyperLogLog

虽然集合能够很好的记录活动人数， 但以上这两个方案都有一个明显的缺点， 那就是， 这两个方案耗费的内存会随着被统计用户数量的增多而增多： 如果你的用户数量比较多， 又或者你需要记录多天/多个时段的送礼用户名单并进行聚合计算， 那么这两个方案可能会消耗你大量内存。

另一方面， 在有些情况下， 我们只想要知道送礼用户的人数， 而不需要知道具体的送礼用户名单， 这时集合储存的信息就会显得多余了。

在需要尽可能地节约内存并且只需要知道送礼用户数量的情况下， 我们可以使用 HyperLogLog 来对送礼用户进行统计： HyperLogLog 是一个概率算法， 它可以对元素的基数进行估算， 并且每个 HyperLogLog 只需要耗费 12 KB 内存， 对于用户数量非常多但是内存却非常紧张的系统， 这一方案无疑是最佳之选。

在这一方案下， 我们使用 [PFADD](http://redisdoc.com/hyperloglog/pfadd.html) 命令去记录在线的用户：

```
PFADD "users" <uid>
```

使用 [PFCOUNT](http://redisdoc.com/hyperloglog/pfcount.html) 命令获取在线人数：

```
PFCOUNT "users"
```

因为 HyperLogLog 也提供了计算交集的 [PFMERGE](http://redisdoc.com/hyperloglog/pfmerge.html) 命令， 所以我们也可以用这个命令计算出多个给定时间段或日期之内， 上线的总人数：

```
# 统计 7 天之内总共有多少人上线了
PFMERGE "7_days_both_users" "day_1_users" "day_2_users" ... "day_7_users"
PFCOUNT "7_days_both_users"
```

## 方案 3 ：使用位图（bitmap）

回顾上面介绍的2个方案， 我们可以得出以上结论：

- 使用集合能够储存具体的送礼用户名单， 但是却需要消耗大量的内存；
- 而使用 HyperLogLog 虽然能够有效地减少统计送礼用户所需的内存， 但是它却没办法准确地记录具体的送礼用户名单。

那么是否存在一种既能够获得送礼用户名单， 又可以尽量减少内存消耗的方法存在呢？ 这种方法的确存在 —— 使用 Redis 的位图就可以办到。

Redis 的位图就是一个由二进制位组成的数组， 通过将数组中的每个二进制位与用户 ID 进行一一对应， 我们可以使用位图去记录每个用户是否在线。

当一个用户上线时， 我们就使用 [SETBIT](http://redisdoc.com/string/setbit.html) 命令， 将这个用户对应的二进制位设置为 1 ：

```
# 此处的 uid 必须为数字，因为它会被用作索引
SETBIT "users" <uid> 1
```

通过使用 [GETBIT](http://redisdoc.com/string/getbit.html) 命令去检查一个二进制位的值是否为 1 ， 我们可以知道指定的用户是否送过礼：

```
GETBIT "users" <uid>
```

而通过 [BITCOUNT](http://redisdoc.com/string/bitcount.html) 命令， 我们可以统计出位图中有多少个二进制位被设置成了 1 ， 也即是有多少个用户送过礼：

```
BITCOUNT "users"
```

跟集合一样， 用户也能够对多个位图进行聚合计算 —— 通过 [BITOP](http://redisdoc.com/string/bitop.html) 命令， 用户可以对一个或多个位图执行逻辑并、逻辑或、逻辑异或或者逻辑非操作：

```
# 计算出 7 天都送礼的用户
BITOP "AND" "7_days_both_users" "day_1_users" "day_2_users" ... "day_7_users"

# 计算出 7 天的送礼用户总人数
BITOP "OR" "7_days_total_users" "day_1_users" "day_2_users" ... "day_7_users"

# 计算出两天当中只有其中一天送礼的用户
BITOP "XOR" "only_one_day_sent" "day_1_users" "day_2_users"
```

位图方案记录一个用户是否在线需要花费 1 个二进制位， 对于用户数为 100 万的网站来说， 使用这一方案只需要耗费 125 KB 内存， 而对于用户数为 1000 万的网站来说， 使用这一方案也只需要花费 1.25 MB 内存。

虽然位图节约内存的效果不及 HyperLogLog 那么显著， 但是使用位图可以准确地判断一个用户是否上线， 并且能够像集合和有序集合一样， 对送礼用户名单进行聚合计算。 因此对于想要尽量节约内存， 但又需要准确地知道用户是否在线， 又或者需要对用户的在线名单进行聚合计算的应用来说， 使用位图可以说是最佳之选。

## 总结

以下表格总结了以上3个方案的特点：

| 方案        | 特点                                                         |
| :---------- | :----------------------------------------------------------- |
| 集合        | 能够储存送礼用户的名单，也能够执行聚合计算，消耗的内存比有序集合少，但是跟有序集合一样，这个方案消耗的内存也会随着用户数量的增多而增多。 |
| HyperLogLog | 无论需要统计的用户有多少，只需要耗费 12 KB 内存，但由于概率算法的特性，只能给出在线人数的估算值，并且也无法获取准确的送礼用户名单。 |
| 位图        | 在尽可能节约内存的情况下，记录送礼用户的名单，并且能够对这些名单执行聚合操作。 |

