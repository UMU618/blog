---
layout: post
title: 【GSL 系列 4】为什么需要 gsl::not_null？
date: 2023-04-29 16:59:47
description: UMU 叫 ChatGPT 教您 C++
categories: UMUTech
tags:
- cpp
- dev
- gsl
- optimization
---
[gsl]: https://github.com/microsoft/GSL
[ccg]: https://github.com/UMU618/CppCoreGuidelines-zh-CN

## 问题

- 为啥要用 gsl::not_null？什么时候用？

## 分析

gsl::not_null 修饰指针，主要有两个目的：

- 提高可读性。它相等于给代码写了一份注释，说明它修饰的变量不能为 nullptr。
- 提高执行效率。对一定不会是 nullptr 的变量反复判空，怎么说都是白费力气。

它主要用于函数的：

- 参数。说明这个参数不接受 nullptr。
- 返回值。说明这个函数不会返回 nullptr。比如说，它恒返回一个 static 值/引用。

## 感想

本文大概率是 [GSL 系列](/tags/gsl/)的最后一篇。[GSL][gsl] 有部分已经被 C++ 20 的 STL 覆盖，比如 gsl::byte。众所周知，现在 C++ 20 是主流……所以本系列只划了 4 个重点，其它类可以选修。

- gsl::final_action

- gsl::narrow_cast

- gsl::owner

- gsl::not_null

挑选的这 4 个类，主要贯彻 [C++ 核心指南][ccg] 提到的以下原则：

- P.1: 在代码中直接表达你的想法

- P.3: 表达你的设计意图

- P.5: 编译期检查优先于运行时检查

- P.8: 不要泄漏任何资源

最后，[GSL][gsl] 的定位是基础，也就是说它本身应该是一看就懂，不需要特地去分析的，用起来就对了。但实际上，很多人会嫌引入一个库麻烦，干脆不用。如果您是初学者，这种心态是要不得的，因为一开始不认真对待，写代码时很容易一多就乱，一乱就弃疗。
