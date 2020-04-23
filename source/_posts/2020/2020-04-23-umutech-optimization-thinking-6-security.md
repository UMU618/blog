---
layout: post
title: 优化思维【6】安全性
date: 2020-04-23 14:56:05
categories: UMUTech
tags:
- dev
- optimization
- 
---
## 前情

前五篇，主要考虑性能优化，只有第二篇与安全性相关。

- [优化思维【1】字符串去空格](/2019/04/10/umutech-optimization-thinking-1-trim-strings/)

- [优化思维【2】有符号和无符号的本质区别](/2019/05/16/umutech-optimization-thinking-2-signed-and-unsigned/)

- [优化思维【3】消除没必要步骤](/2019/07/16/umutech-optimization-thinking-3-remove-unnecessary-steps/)

- [优化思维【4】Bash for 循环](/2019/07/30/umutech-optimization-thinking-4-bash-for-loop/)

- [优化思维【5】合并步骤](/2020/02/26/umutech-optimization-thinking-5-merge-steps/)

其实区块链业界一直不缺乏黑客，最近看过不少安全事故导致惨重代价，所以想总结点安全性方面的优化思路。总的来说，为了安全是必须付出实现或者性能代价的。实现代码是开发、测试阶段就要投入更多精力，性能代码是因为考虑更多，有可能消耗更大运行资源。但从长期来看，这些代价都是必须的。

## 知识深度

一般黑客都是上层、底层皆通，尤其擅长底层。很少听说只做增删查改业务的人能够黑掉什么东西、偷到数字货币，因为同样只做增删查改业务的人就具备防止这种级别的攻击手段。

比如古老的 SQL 注入漏洞，即便是入门级的 Web 开发也能理解并防护，用预编译语句、存储过程、改用 ORM 就天然免疫。他们无法防护的往往来自更底层的 Web Server 的漏洞，比如 Apache、Nginx 某个版本有 bug，刚好中枪。

再举个例子，用 C/C++ 写 UDP 服务程序，“先把它实现，能用就行”，“不就 socket 嘛？很容易！”于是没有考虑 socket 等资源的生存周期，没料到**黑客可以伪造 UDP 包源地址**，实现出来的就可能有**拒绝服务攻击 (Denial of Service，DoS) 漏洞**。

总之，为了性能或安全的优化，开发者往往需要往底层钻。为性能，主要是研究底层模块与之配合，达到消除瓶颈目的；为安全，则是不让对底层设计的不了解，实现不严谨周密而产生漏洞。
