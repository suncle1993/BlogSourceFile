---
abbrlink: 1939519946
alias: 2021/07/27/go-genstruct/index.html
categories:
- golang
date: '2021-07-27T15:36:52'
tags:
- genstruct
- generator
title: 仿MybatisGenerator：根据sql生成go struct
---







# genstruct

项目地址：https://github.com/suncle1993/genstruct

根据mysql schema生成go struct，适用于习惯先写sql后写struct的同学

根据 https://github.com/fifsky/genstruct 项目做了一些修改，更适用于目前的hago项目。在原版的基础上添加了以下功能：

1. schema的生成（信奉sql和model放在一起的人喜欢这种方式），便于自动建表之内的操作
2. 暴露的变量和方法注释的添加
3. 通过表注释添加为struct的注释
4. 对于生成的struct字段进行首字母缩写词的转换，总共38种，完全符合golint的检查规则

<!--more-->

## 命令行版本

安装：

```
go get github.com/suncle1993/genstruct
```

使用方法：

```
genstruct -h 127.0.0.1 -u root -P 123456 -p 3306
```

* `-h` default `localhost`
* `-u` default `root`
* `-p` default `3306`

演示见下面的 aciinema svg：

[![asciicast](https://asciinema.org/a/X5sk7TqrTTjF8AhN764K0Fc6m.svg)](https://asciinema.org/a/X5sk7TqrTTjF8AhN764K0Fc6m)

## 线上版本

页面地址：https://genstruct.suncle.me/

![](https://suncle-public.oss-cn-shenzhen.aliyuncs.com/uPic/DV5XD3-1627372059061.png)


## 接口版本

```bash
curl --location --request GET 'https://genstructapi.herokuapp.com/api/struct/generate' \
--header 'Content-Type: application/json' \
--data-raw '{
    "tags": ["db", "json"],
    "table": "create table user_mine_info( id bigint(20) NOT NULL AUTO_INCREMENT, uid bigint(20) NOT NULL DEFAULT '\''0'\'' COMMENT '\''用户uid'\'', mined_cnt bigint(20) NOT NULL COMMENT '\''剩余挖矿次数'\'', un_exchange_diamond bigint(20) NOT NULL COMMENT '\''未兑换为挖矿次数的钻石'\'', created_at bigint(20) NOT NULL COMMENT '\''创建时间'\'', updated_at bigint(20) NOT NULL COMMENT '\''更新时间'\'', PRIMARY KEY (id), UNIQUE KEY uk_uid (uid) USING BTREE) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 COMMENT = '\''用户挖矿剩余次数记录'\'';"
}'
```

## 示例模型

建表数据

```mysql
CREATE TABLE user_mine_info(
  id bigint(20) NOT NULL AUTO_INCREMENT, 
  UID bigint(20) NOT NULL DEFAULT '0' COMMENT '用户uid', 
  mined_cnt bigint(20) NOT NULL COMMENT '剩余挖矿次数', 
  un_exchange_diamond bigint(20) NOT NULL COMMENT '未兑换为挖矿次数的钻石', 
  created_at bigint(20) NOT NULL COMMENT '创建时间', 
  updated_at bigint(20) NOT NULL COMMENT '更新时间', 
  PRIMARY KEY (id), 
  UNIQUE KEY uk_uid (UID) USING BTREE
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 COMMENT = '用户挖矿剩余次数记录';

```

生成的模型：

```go
package user_mine_info

// UserMineInfo 用户挖矿剩余次数记录
type UserMineInfo struct {
	ID                int64 `db:"id" json:"id" `
	UID               int64 `db:"uid" json:"uid" `                                 // 用户uid
	MinedCnt          int64 `db:"mined_cnt" json:"mined_cnt" `                     // 剩余挖矿次数
	UnExchangeDiamond int64 `db:"un_exchange_diamond" json:"un_exchange_diamond" ` // 未兑换为挖矿次数的钻石
	CreatedAt         int64 `db:"created_at" json:"created_at" `                   // 创建时间
	UpdatedAt         int64 `db:"updated_at" json:"updated_at" `                   // 更新时间
}

// TableName ...
func (u *UserMineInfo) TableName() string {
	return "user_mine_info" // TODO: 如果分表需要修改
}

// PK ...
func (u *UserMineInfo) PK() string {
	return "id"
}

// Schema ...
func (u *UserMineInfo) Schema() string {
	return `(
  id bigint NOT NULL AUTO_INCREMENT,
  uid bigint NOT NULL DEFAULT '0' COMMENT '用户uid',
  mined_cnt bigint NOT NULL COMMENT '剩余挖矿次数',
  un_exchange_diamond bigint NOT NULL COMMENT '未兑换为挖矿次数的钻石',
  created_at bigint NOT NULL COMMENT '创建时间',
  updated_at bigint NOT NULL COMMENT '更新时间',
  PRIMARY KEY (id),
  UNIQUE KEY uk_uid (uid) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='用户挖矿剩余次数记录'`
}
```
