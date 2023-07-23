---
layout: post
title: 学习 go 语言【3】TCP Echo Server
date: 2015-09-22 15:41:56
categories: UMUTech
tags:
- dev
- go
---
## 问题

测试需要，以前用 C + libevent 写了一个 TCP Echo Server，返回服务器时间、客户端地址信息和客户端发送的原内容。为了水一篇，现在改为 go 语言实现。

## 参考

主要使用 go 语言自带的 net 库，学习资料：<https://golang.org/pkg/net/>

## 代码

```go
package main

import (
	"fmt"
	"io"
	"net"
	"os"
	"time"
)

const BUFFER_SIZE = 1024 * 4

var buffer = make([]byte, BUFFER_SIZE)

func handleConnect(tcpConn *net.TCPConn) {
	if tcpConn == nil {
		return
	}
	for {
		n, err := tcpConn.Read(buffer)
		if err == io.EOF {
			fmt.Printf("The RemoteAddr: %s is closed!\n", tcpConn.RemoteAddr().String())
			return
		}
		handleError(err)
		if n > 0 {
			//fmt.Printf("Read: %s", string(buffer[:n]))
			str := tcpConn.RemoteAddr().String() + " @ " +
				time.Now().Format("2006-01-02 15:04:05 Z07:00") + "\r\n" +
				string(buffer[:n])
			tcpConn.Write([]byte(str))
			fmt.Printf("Echo: %s", str)
		}
	}
}

// 错误处理
func handleError(err error) {
	if err == nil {
		return
	}
	panic(err)
	//fmt.Printf("error: %s\n", err.Error())
}

func main() {
	if len(os.Args) < 2 {
		fmt.Println("Usage:", os.Args[0], "<port>")
		return
	}
	port := os.Args[1]
	tcpAddr, err := net.ResolveTCPAddr("tcp4", "0.0.0.0:"+port)
	handleError(err)
	tcpListener, err := net.ListenTCP("tcp4", tcpAddr)
	handleError(err)
	defer tcpListener.Close()

	fmt.Println("Listening on", tcpAddr, "...")
	for {
		tcpConn, err := tcpListener.AcceptTCP()
		fmt.Printf("The client: %s has connected!\n", tcpConn.RemoteAddr().String())
		handleError(err)
		defer tcpConn.Close()
		go handleConnect(tcpConn)
	}
}
```
