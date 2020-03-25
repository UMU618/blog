---
layout: post
title: MongoDB db.stats() 的各种 Size
date: 2018-11-18 19:16:25
categories: UMUTech
tags:
- ops
- mongodb
---
## 现象

db.stats() 的各种 Size 需要理理，先看例子：

```
> db.action_traces.dataSize()
12489840963

> db.action_traces.totalSize()
5391249408

> db.action_traces.storageSize()
3684032512

> db.action_traces.totalIndexSize()
1707216896
```

## 概念解释

db.action_traces.stats() 里的 size 就是 db.action_traces.dataSize()，也就是数据本身的逻辑大小。

由于数据库引擎有压缩概念，所以存储到介质时，可能占用的空间并没有逻辑大小那么多，比如 WiredTiger Storage Engine 的压缩率就挺不错的，dataSize = 12,489,840,963 字节的数据，存到硬盘只有 storageSize = 5,391,249,408 字节。

其中 totalIndexSize 是索引占存储器的大小，所以 totalSize = storageSize + totalIndexSize。

注意：索引有时会比数据本身还大……

## 参考

[db.collection.stats() — MongoDB Manual](https://docs.mongodb.com/manual/reference/method/db.collection.stats/index.html)

[db.collection.totalIndexSize() — MongoDB Manual](https://docs.mongodb.com/manual/reference/method/db.collection.totalIndexSize/index.html)

[db.collection.dataSize() — MongoDB Manual](https://docs.mongodb.com/manual/reference/method/db.collection.dataSize/index.html)

[db.collection.storageSize() — MongoDB Manual](https://docs.mongodb.com/manual/reference/method/db.collection.storageSize/index.html)

[db.collection.totalSize() — MongoDB Manual](https://docs.mongodb.com/manual/reference/method/db.collection.totalSize/)