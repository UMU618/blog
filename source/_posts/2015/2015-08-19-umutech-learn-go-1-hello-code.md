---
layout: post
title: 学习 go 语言【1】Hello code!
date: 2015-08-19 16:44:00
categories: UMUTech
tags:
- dev
- go
---
## 需求

go 即将发布的 1.5 版解决了 GC stop-the-world 问题，所以 UMU 打算以后用它来开发工具。

最近想统计代码行数，以前用 VBS 写的一时间居然找不到，直接用 go 写一个。

## 心得

基本从零开始用了大约 4 小时完成，可见 go 对初学者相当友好。具体经验和心得：

1. go 的 runtime 居然没有 set，只能用 map 代替了，一开始觉得不优雅，不过想来也差不多，不计较那么多了。

2. 语法还确实挺简洁，第一次练手就感觉学这个语言，其实是在学它的规范，语言本身很容易。

3. defer 挺好用的，简洁、省心，比如这个核心函数：

```go
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
```

4. LiteIDE X 还比较好用，写 import 保存时，会用 `gofmt` 自动按字母顺序排列库名，这样省得纠结顺序……

## 代码

刚刚接触，就说到这里，最后附代码。

```go
// UMU @ 2015-08-17 11:30
// Last update: 2015-08-17 17:01
package main

import (
	"bufio"
	"fmt"
	"os"
	"path"
	"path/filepath"
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

	files := make(map[string]bool)
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
	fmt.Println("Total lines:", lines)
}

func AddDirectory(name string, files map[string]bool) {
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
	return
}

func AddFile(name string, files map[string]bool) {
	ext := path.Ext(name)
	if isSourceFile(ext) {
		path, err := filepath.Abs(name)
		if err == nil {
			_, exists := files[path]
			if exists {
				fmt.Println("Duplicated", path)
			} else {
				files[path] = true
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
```
