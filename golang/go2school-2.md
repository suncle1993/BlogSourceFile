---
categories:
  - golang
date: '2018-11-15T15:36:59'
description: ''
tags:
  - Go
  - Redis
title: go2school-2
---





go2school 系列的第二道题目

- 用 Go 语言实现一个简单的 HTTP Server。

# 题目

简要描述：实现 GET(get), POST(set) 两个功能

```json
// 读取Redis的"path:path2"
@GET /path/path2
// 将 value(=3) 写入 redis 的 "path:path2"
@POST /path/path2
{
    "value": 3
}
```

现有可用的redis数据库（已脱敏）：

```json
{
    "Addr":"t.kezaihui.com:6580",
    "Password":"06ad0c72d5ce41ce9f04ad1237965a4d"
}
```

<!--more-->

# 交作业

1. 使用go语言原生的http库作为route处理url：暂时不适用框架
2. 使用go-redis作为redis client：使用人数多，速度快推荐

```go
package main

import (
	"encoding/json"
	"fmt"
	"github.com/go-redis/redis"
	"net/http"
	"os"
	"regexp"
)

type payloadStruct struct {
	Value string
}

var reg = regexp.MustCompile(`^/([(\w)]+)/([(\w)]+)/?$`)
var redisClient = redis.NewClient(&redis.Options{
	Addr:     "t.kezaihui.com:6580",
	Password: "06ad0c72d5ce41ce9f04ad1237965a4d",
})


func main() {
	http.HandleFunc("/", Route)
	err := http.ListenAndServe(":8080", nil)
	if err == nil {
		os.Exit(4)
	}
}


// web route
func Route(w http.ResponseWriter, r *http.Request) {
	path := r.URL.Path
	params := reg.FindStringSubmatch(path)
	key := params[1] + ":" + params[2]
	if r.Method == "GET" {
		val := getValueFromRedis(key)
		fmt.Fprintf(w, val)
	} else if r.Method == "POST" {
		decoder := json.NewDecoder(r.Body)
		var payload payloadStruct
		err := decoder.Decode(&payload)
		if err != nil {
			panic(err)
		}
		setValueToRedis(key, payload.Value)
		fmt.Fprintf(w, "ok")
	} else {
		fmt.Fprintf(w, "NOT IMPLEMENT")
	}
}


func setValueToRedis(key string, value string) {
	fmt.Println(redisClient, key, value)
	redisClient.Set(key, value, 0).Err()
}


func getValueFromRedis(key string) (val string) {
	fmt.Println(redisClient, key)
	val, err := redisClient.Get(key).Result()
	if err != nil {
		panic(err)
	}
	fmt.Println("key", val)
	return val
}
```

本次的 http server 也可以使用 web 框架，比如 iris 和 mux。（iris知名度高）

# 存在的问题

整体写下来还有两个问题不是很清楚

- Gopath
- 包依赖管理

因为需要依赖go-redis这个第三方包，因此需要使用`go get`这个命令下载第三方包，但是下载下来之后GoLand这个IDE会显示包不存在，但是实际运行并不会有问题，有代码洁癖的人实在无法忍受。本质原因还是Gopath没有设置正确。因为go的workspace和python还有c++实在是不同。因此习惯go的workspace，理解Gopath就至关重要了。但是好消息是go的2.0版本会使用类似于npm和pipenv的方式解决包管理的问题，期待...