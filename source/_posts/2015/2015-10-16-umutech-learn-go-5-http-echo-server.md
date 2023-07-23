---
layout: post
title: 学习 go 语言【5】HTTP Echo Server
date: 2015-10-16 15:19:11
categories: UMUTech
tags:
- dev
- go
---
## 需求

做加速时经常需要用到 HTTP Echo Server 来测试加速有没有成功，如果成功了，是显示请求来自加速代理服务器。原来用 node.js 写了一个，代码如下：

```nodejs
var http = require('http');
http.createServer(
    function (req, res) {
        res.writeHead(200, {'Content-Type': 'text/plain'});
        var ip = req.headers['x-forwarded-for'] || req.connection.remoteAddress || req.socket.remoteAddress || req.connection.socket.remoteAddress;
        var ua = req.headers['user-agent'];
        res.end(ip + '\n' + ua);
    }
).listen(80);
console.log('Server is running...');
```

既然在学 Go 语言，就用它写个新的吧！功能要更强一些。

## 代码

稍微改进一下：

```go
package main

import (
	"fmt"
	"net/http"
	"os"
	"time"
)

func main() {
	var addr string
	if len(os.Args) > 1 {
		addr = ":" + os.Args[1]
	} else {
		addr = ":80"
	}
	server := http.Server{
		Addr:        addr,
		Handler:     &MyHandler{},
		ReadTimeout: 5 * time.Second,
	}
	err := server.ListenAndServe()
	fmt.Println(err)
}

type MyHandler struct{}

func (*MyHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	info := r.RemoteAddr + " @ " + time.Now().String() + "\r\n" +
		r.Method + " " + r.RequestURI + "\r\n" +
		"Host: " + r.Host + "\r\n" +
		"UserAgent: " + r.UserAgent() + "\r\n"
	w.Write([]byte(info))
	fmt.Println(info)
}
```
