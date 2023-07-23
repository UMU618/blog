---
layout: post
title: 【GSL 系列 3】为什么需要 gsl::owner？
date: 2023-04-22 22:54:53
description: UMU 叫 ChatGPT 教您 C++
categories: UMUTech
tags:
- cpp
- dev
- gsl
- optimization
---
## 问题

- 为啥要用 gsl::owner？什么时候用？

## 分析

当我们需要指针时，对于新写的代码，更应该使用智能指针（Smart Pointers）。但由于历史原因，一些旧代码里的裸指针（Raw Pointers）难以短时间重构为智能指针。而很多时候，裸指针的所有权难以一眼看出。良心代码可能通过注释指明，屎山代码就只能通过阅读大片相关语句块或函数来判断。

如果能用一个比较标准或通用的方式来指明，即可提高代码可读性，程序员能够更好滴理解代码，也就相应地能够提高健壮性。没错，这就是 gsl::owner 的应用场景。它就是**提高【含有大量裸指针的】旧代码的可读性和健壮性的简单方法！**

举个最简单的场景：一个类里有一个成员变量叫 A* ptr，那么当这个类析构时，需要 delete ptr 吗？

- 如果类对 ptr 有所有权，而析构时，没 delete ptr，则资源泄漏，危害整个程序。

- 如果类对 ptr 无所有权，而析构时，delete ptr，则造成悬空指针（Dangling Pointer），危害其它类。

这个类的作者当然知道需不需要了，但即使他知道，也可能忘记写，或可能因为套用现有代码（Copy & Paste）而多写了 delete！其他接盘侠（代码阅读者）想弄清楚这个问题，就更难了，一般需要认真阅读并调试。

但如果一开始，作者就用 gsl::owner<A*> ptr 来指明所有权，那么很自然地，大家都很容易知道：类析构时，需要 delete ptr。

当代码应用 gsl::owner 时，也能得使一些静态代码分析工具（static code analysis tools）更容易找出资源泄露问题。