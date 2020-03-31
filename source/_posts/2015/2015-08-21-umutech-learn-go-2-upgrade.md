---
layout: post
title: 学习 go 语言【2】升级 1.5 + 优化
date: 2015-08-21 19:18:39
categories: UMUTech
tags:
- dev
- go
---
## 问题

安装 1.5 时，直接覆盖 1.4.2，结果不能用了，报错：

> imports runtime: C source files not allowed when not using cgo or SWIG

## 解决

删掉 go 1.5……然后修复安装一遍。

## 优化前例代码

1. 加了计时功能，纯属蛋疼。

2. 学到一个不占空间的 struct{}，map[string]bool 改为 map[string]struct{}。


```go
// UMU @ 2015-08-17 11:30
// Last update: 2015-08-21 17:40
package main

import (
	"bufio"
	"fmt"
	"os"
	"path"
	"path/filepath"
	"time"
)

func isSourceFile(ext string) bool {
	var kSourceFileExts = []string{".c", ".cc", ".cpp", ".h", ".hpp",
		".go",
		".java"}
	for _, r := range kSourceFileExts {
		if r == ext {
			return true
		}
	}
	return false
}

func main() {
	if len(os.Args) < 2 {
		fmt.Println("Usage: ", os.Args[0], "[file or folder]...")
		return
	}

	start := time.Now()
	files := make(map[string]struct{})
	lines := 0

	for _, r := range os.Args {
		fi, err := os.Stat(r)
		if err == nil || os.IsExist(err) {
			if fi.IsDir() {
				AddDirectory(r, files)
			} else {
				AddFile(r, files)
			}
		}
	}

	for file, _ := range files {
		l := CountLine(file)
		lines += l
		fmt.Println(file, l)
	}
	fmt.Printf("Total lines: %d, cost %fs\n", lines, timeElapsed(start))
}

func AddDirectory(name string, files map[string]struct{}) {
	filepath.Walk(name, func(path string, fi os.FileInfo, err error) error {
		if nil == fi {
			return err
		}
		if fi.IsDir() {
			return nil
		}
		AddFile(path, files)
		return nil
	})
}

func AddFile(name string, files map[string]struct{}) {
	ext := path.Ext(name)
	if isSourceFile(ext) {
		path, err := filepath.Abs(name)
		if err == nil {
			_, exists := files[path]
			if exists {
				fmt.Println("Duplicated", path)
			} else {
				files[path] = struct{}{}
				fmt.Println("Add", path)
			}
		}
	}
}

func CountLine(path string) (num int) {
	f, err := os.Open(path)
	if nil != err {
		return
	}
	defer f.Close()

	s := bufio.NewScanner(f)
	for s.Scan() {
		num += 1
	}
	return
}

func timeElapsed(start time.Time) float64 {
	dis := time.Since(start).Seconds()
	return dis
}
```
