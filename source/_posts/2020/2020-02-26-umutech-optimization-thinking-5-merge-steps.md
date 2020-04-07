---
layout: post
title: 优化思维【5】合并步骤
date: 2020-02-26 23:38:07
categories: UMUTech
tags:
- dev
- optimization
---
## 前情

在《[优化思维【3】消除没必要步骤](/2019/07/16/umutech-optimization-thinking-3-remove-unnecessary-steps/)》提到一个对象转化的例子：A 对象要转为 B 对象，实现时先把 A 对象转为中间对象 T，再将 T 转为 B 对象，由于两步都很容易实现，一个现有函数即可，所以很多人可能会采用这个思路。

下面要介绍的“合并步骤”，类似于优等生解应用题时“跳步”（一行合并多个步骤），可以作为前文的补充。

## 例子

### 1. 合并 cat 和 grep

以下命令 UMU 经常看到，其实它可以用 `grep UMU test` 来优化，减少一次管道交互。

```sh
cat test | grep UMU
```

### 2. 合并多个 sed

再看下面例子是从一个命令行里移除 A 和 C 两个选项：

```sh
cmd="EXE A=1 B=2 C=3 D=4"
removed=$(echo $cmd | sed -e 's/ A=[^ ]*//' | sed -e 's/ C=[^ ]*//')
echo $removed
```

其中两次 `sed` 可以合并为一次：

- `sed -e 's/ A=[^ ]*//;s/ C=[^ ]*//'`

- 或 `sed -e 's/ [\(A\)\(C\)]=[^ ]*//g'`

- 或 `sed -e 's/ \(A\|C\)=[^ ]*//g'`，这个 macOS 上不行。

- 或 `sed -E -e 's/ (A|C)=[^ ]*//g'`，这个适合 macOS。

### 3. TFO

先查一下 `sysctl net.ipv4.tcp_fastopen`，一般应该是 1，说明客户端支持 TFO；如果是 2 则说明服务端支持；3 是同时支持。

TFO (TCP Fast Open) 是一种能够在 TCP 连接建立阶段传输数据的机制。使用这种机制可以将数据交互提前，降低应用层事务的延迟。

这其实也是一种合并步骤的思想，把传输数据合并到三次握手期间。

参考：

- <https://www.ietf.org/proceedings/80/slides/tcpm-3.pdf>

- <https://datatracker.ietf.org/doc/rfc7413/>
