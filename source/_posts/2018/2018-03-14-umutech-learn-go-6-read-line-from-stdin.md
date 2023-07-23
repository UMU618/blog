---
layout: post
title: 学习 go 语言【6】从 stdin 读取一行汉字
date: 2018-03-14 23:39:35
categories: UMUTech
tags:
- dev
- go
---
## 需求

需要从 stdin 读取一行汉字！搜了几篇出来，居然都不支持输入中文……

## 代码

```go
func ScanLine() (line string) {
	var buffer []rune
	for {
		var c rune
		n, err := fmt.Scanf("%c", &c)
		if nil != err || 1 != n || '\r' == c || '\n' == c {
			break
		}
		buffer = append(buffer, c)
	}
	return string(buffer)
}
```
