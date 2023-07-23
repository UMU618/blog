---
layout: post
title: 学习 MongoDB 选举机制
date: 2018-05-31 18:25:32
categories: UMUTech
tags:
- dev
- mongodb
---
为了快速了解 MongoDB 选举机制，在网上找了一些文章来学习，后来发现里面提到的一些机制都过时了，尝试看代码了解，发现协议有 PV0 和 PV1 两种。

代码：[https://github.com/mongodb/mongo/blob/r3.6.5/src/mongo/db/repl/topology_coordinator.cpp](https://github.com/mongodb/mongo/blob/r3.6.5/src/mongo/db/repl/topology_coordinator.cpp)

一篇比较新的参考文章：[https://blog.csdn.net/wentyoon/article/details/78986174](https://blog.csdn.net/wentyoon/article/details/78986174)

如果新选举出的主节点立马挂掉，至少需要 30s 重新选主，这个是由 leaseTime 常量决定的：

const Seconds TopologyCoordinator::VoteLease::leaseTime = Seconds(30);

PV0 时，一个反对会将最终票数减 10000，即在绝大多数情况下,只要有节点反对，请求的节点就不能成为主节点，由 prepareElectResponse 函数实现，里面有不少 vote = -10000;，PV1 版本取消了否决票。
