---
layout: post
title: 优化思维【3】消除没必要步骤
date: 2019-07-16 20:14:44
categories: UMUTech
tags:
- dev
- cpp
- optimization
- blockchain
---
## 故事

四月底给 [EOSIO](https://github.com/EOSIO) / [eos](https://github.com/EOSIO/eos) 提了一个[优化 MongoDB 插件性能的 PR](https://github.com/EOSIO/eos/pull/7207)，被连续感谢好几个 [Release](https://github.com/EOSIO/eos/releases/tag/v1.8.0)。

## 分析

原先的流程：fc::variant -> JSON string -> BSON，实现起来很简单，因为 JSON 是很常见的，fc::variant 和 BSON 都有到 JSON 的转化，所以实现代码很简单，一行两个函数。

但数据大时，性能问题就暴露了，这个过程先把 fc::variant 对象序列化为 JSON 字符串，然后反序列化到 BSON 对象。两步都是 CPU 密集型操作，由于 nodeos 及其插件暂时对多核支持不好，导致单核跑爆。

两个过程都要用递归实现，调用栈可能很深。调用函数可能有入栈出栈的消耗，有一种优化思路正是**用 inline 减少函数的频繁调用**。

回归到本质，fc::variant 和 BSON 都是对象，应该直接转化才对。只是实现起来就不是一行能搞定的。先挑简单的方式实现，后期再优化，这是一种挺常规的做法。
