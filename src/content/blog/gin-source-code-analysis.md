---
author: yni9ht
pubDatetime: 2023-03-15T16:45:41Z
title: Gin Source Code Analysis - Part 1
postSlug: gin-source-code-analysis-part-1
featured: true
draft: false
tags:

- golang
- gin
- web
- framework
- source
- code
- analysis
- docs

ogImage: ""
description: ""

---

# Gin 源码分析 - Part 1

[Gin](https://gin-gonic.com/) 是一个使用 Golang 编写的 Web 框架，它的设计目标是快速、简单、可扩展。Gin
有一个非常活跃的社区，它的文档也非常完善。本文将对
Gin 的源码进行分析，希望能够帮助大家更好地理解 Gin 的设计思路。

## Gin 的设计

### 快速开始

我们先看下如何使用 Gin 来快速启动一个 Web 应用。

通过下面的代码我们便可以启动一个 Web 服务，监听 8080 端口，同时提供了一个访问路径为 `/ping` 的接口。

```go
package main

import (
	"github.com/gin-gonic/gin"
	"log"
)

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})

	// listen and serve on 0.0.0.0:8080
	if err := r.Run(); err != nil {
		log.Fatal(err)
	}
}

```

### 分析

我们先是通过 `gin.Default()` 创建了一个 Gin 的实例（该实例类型为 `*Engine`），然后通过 `r.GET()` 注册了一个 `GET`
请求的路由，最后通过 `r.Run()` 启动了 Web 服务。

#### Engine

这个对象是 Gin 的核心，我们来看下他的结构体定义。

```go
type Engine struct {
	RouterGroup
	// ...... 省略部分代码
	allNoRoute       HandlersChain
	allNoMethod      HandlersChain
	noRoute          HandlersChain
	noMethod         HandlersChain
	pool             sync.Pool
	trees            methodTrees
	// ...... 省略部分代码
}

var _ IRouter = (*Engine)(nil)
```

可以看到，`Engine` 嵌套了 `RouterGroup`。并且还包含一个 `trees` 字段，该字段存储的是所有的路由信息，是 Gin 的核心之一，
稍后我们会详细分析该路由树的具体细节。
