---
layout: post
title: Mongo Shell 下批量更新集合
date: 2018-02-02 17:57:47
categories: UMUTech
tags:
- ops
- mongodb
---
## 需求

延长 MongoDB 某集合里的“过期时间”字段。

## 风险分析

update 一下是很简单，主要怕在 Shell 下操作可能改变数字类型。
先做了实验，发现 3.2 的版本下，并没有这个问题，之前看书，说数字可能被改为双精度，看来是旧版本的不足。

```
db.UMU.find().forEach(function (doc) {
    doc.expireDate = NumberLong(doc.updateTime + 180*24*60*60*1000);
    db.UMU.save(doc);
})
```

其中 NumberLong 是必要的，不然更新后，expireDate 的类型并不是和 updateTime 一样的 NumberLong。
