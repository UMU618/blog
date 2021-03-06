---
layout: post
title: 优化思维【4】Bash for 循环
date: 2019-07-30 23:25:57
categories: UMUTech
tags:
- dev
- optimization
---
## 常规教材

Bash 循环有三种写法。

### 1. 类 C 语言语法

```

for ((i = 1; i <= 10000000; ++i))
do
  echo $i
done
```

### 2. for in 语法

```
for i in {1..10000000}
do
  echo $i
done
```

### 3. 使用外部命令 seq

```
for i in `seq 1 10000000`
do
  echo $i
done
```

## 分析

这三种方法中，性能最好的是第一种，最差的是第三种。

方法 1 是语言层面的循环语法，循环会**立刻**开始，而后两种，作为对比，并不会**立刻**开始！

方法 2 中的 `{1..10000000}` 会产生一个 1 到 10000000 的序列，然后再开始循环。

方法 3 中的 `seq 1 10000000` 是调用 `seq` 产生一个 1 到 10000000 的序列，然后再开始循环。涉及到外部进程调用和管道传递，所以比方法 2 更慢。

### 总结

- 方法 1 写起来最麻烦，性能却是最好的。**“做一件事，有很多种方式”，有时候不是好事，有对比，就有伤害……优化往往是和人性作对！**

- 语言原生的方法一般比调用外部命令好。

## 同类经验

Java 的 for 和 while 本质上一样（其实 while 可以去掉），而同是 JVM 语言的 Scala 的 for 却不同于 while。Scala 的 `for <-` 也会先产生序列，再循环，所以超大规模循环时，for 性能不如 while。
