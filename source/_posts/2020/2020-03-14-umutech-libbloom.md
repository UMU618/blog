---
layout: post
title: libbloom
date: 2020-03-14 13:51:46
categories: UMUTech
tags:
- c
- dev
- lib
---
## 概念

*Bloom filter* 是由 Howard Bloom 在 1970 年提出的二进制向量数据结构，具有很好的空间和时间效率，被用来检测一个元素是不是集合的成员。

Bloom filter 采用的是哈希函数的方法，将一个元素映射到一个 m 长度的阵列上的一个点，当这个点是 1 时，那么这个元素可能在集合内，反之则一定不在集合内。

[libbloom](https://github.com/jvirkki/libbloom) 是 *Bloom filter* 的 C 语言实现库，其中哈希函数是 MurmurHash2。

## 特征

如果检测结果为是，该元素不一定在集合中；但如果检测结果为否，该元素一定不在集合中。

## 优缺点

- 优点：插入和查询时间都是常数。它查询元素却不保存元素本身，节省大量的存储空间。当元素是密码时，不保存元素的特征使其具有良好的安全性。

- 缺点：存在误报（false positive）。当插入的元素越多，错判“在集合内”的概率就越大。另外 Bloom filter 也不能删除一个元素，因为多个元素哈希的结果可能在 Bloom filter 结构中占用的是同一个位，如果删除了一个比特位，可能会影响多个元素的检测。

## 算法分析

以官方 test-basic 为例，简化的代码如下：

```c
#include <assert.h>

#include "bloom.h"

int main()
{
  struct bloom bloom;
  assert(bloom_init(&bloom, 1002, 0.1) == 0);
  assert(bloom.ready == 1);
  bloom_print(&bloom);
  bloom_free(&bloom);
}
```

输出为：

```
 ->entries = 1002
 ->error = 0.100000
 ->bits = 4802
 ->bits per elem = 4.792529
 ->bytes = 601
 ->hash functions = 4
```

数学原理参考：《[Bloom Filter概念和原理](https://blog.csdn.net/jiaomeng/article/details/1495500)》

- bytes 是最容易理解的，4802 位需要 601 字节存储。

- bits = bits per elem \* entries，每个元素需要多少位 \* 元素个数。

- hash functions = ceil(-ln(error) / ln(2))

- bits per elem = -ln(error) / ln(2)^2

## 参考

<https://baike.baidu.com/item/bloom%20filter>
