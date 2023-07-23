---
layout: post
title: 学习 go 语言【7】设置进程退出码
date: 2018-03-31 20:45:49
categories: UMUTech
tags:
- debug
- dev
- go
---
## 常规方案

直接用 os.Exit (exit_code)，但这个太暴力了，我们需要高雅一点的，于是找到了这个：<https://stackoverflow.com/questions/24601516/correct-way-to-set-exit-code-of-process>

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    code := 0
    defer func() {
        os.Exit(code)
    }()
    defer func() {
        fmt.Println("Another deferred func")
    }()
    fmt.Println("Hello, 世界")
    code = 1
}
```

## 问题

调用 panic 的时候就知道以上的方法存在不足！panic 之后会导致 main 退出，本来紧接着应该打印 Trace Log，然而 main 退出时调用了 os.Exit ()，然后没有然后了……

本来 panic 时，退出码应该是 2 的，结果由于以上装 X 代码的作用，退出码变成了 0！如果 panic 是自己主动调用的，那还可以改改，使用别的方式；如果是其它库函数的就难办了……
