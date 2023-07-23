---
layout: post
title: 学习 go 语言【8】strings.Builder vs bytes.Buffer
date: 2018-04-02 18:47:56
categories: UMUTech
tags:
- dev
- go
- optimization
---
## 结论

- strings.Builder：省内存
- bytes.Buffer：快

## 代码

性能测试程序如下：

```go
package benchmark_test

import (
	"bytes"
	"strings"
	"testing"
)

var (
	ss = "1234567890abcdefghijklmnopqrstuvwxyz"
	bs = []byte(ss)
	rn = 'a'
	bt = byte('a')
)

func BenchmarkBuilderWrite(b *testing.B) {
	var builder strings.Builder
	for i := 0; i < b.N; i++ {
		builder.Write(bs)
	}
}

func BenchmarkBuiderWriteByte(b *testing.B) {
	var builder strings.Builder
	for i := 0; i < b.N; i++ {
		builder.WriteByte(bt)
	}
}

func BenchmarkBuilderWriteRune(b *testing.B) {
	var builder strings.Builder
	for i := 0; i < b.N; i++ {
		builder.WriteRune(rn)
	}
}

func BenchmarkBuilderWriteString(b *testing.B) {
	var builder strings.Builder
	for i := 0; i < b.N; i++ {
		builder.WriteString(ss)
	}
}

func BenchmarkBufferWrite(b *testing.B) {
	var buffer bytes.Buffer
	for i := 0; i < b.N; i++ {
		buffer.Write(bs)
	}
}

func BenchmarkBufferWriteByte(b *testing.B) {
	var buffer bytes.Buffer
	for i := 0; i < b.N; i++ {
		buffer.WriteByte(bt)
	}
}

func BenchmarkBufferWriteRune(b *testing.B) {
	var buffer bytes.Buffer
	for i := 0; i < b.N; i++ {
		buffer.WriteRune(rn)
	}
}

func BenchmarkBufferWriteString(b *testing.B) {
	var buffer bytes.Buffer
	for i := 0; i < b.N; i++ {
		buffer.WriteString(ss)
	}
}
```
