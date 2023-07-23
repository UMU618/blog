---
layout: post
title: 现代 C++【3】返回类对象
date: 2021-09-05 20:17:30
categories: UMUTech
tags:
- cpp
- dev
---
## 前提

多现代？C++ 11 就有了。

## 问题

我想返回一个对象，但我受到惊吓……

是不是应该从指针型参数返回对象？

## 结论

已经 C++20 了，请放心，直接，返回对象！

## 概念

- RVO：Return Value Optimization，返回值优化。

- NRVO：Named RVO，具名的返回值优化。

返回的对象会 move 给接收的变量，并且，最多可能优化成直接对接收变量进行构造（NRVO）。

如果明确没有 move 构造函数，则会调用 copy 构造函数，当对象构造代价高时，应该尽量保证有 move 构造函数。

## 例子

```cpp
// 传统，不建议，可读性差，使用也不方便
void GetName(std::string& name) noexcept {
  name = "UMU";
}

// RVO，优化，但不够
std::string GetName() noexcept {
  return "UMU618";
}

// NRVO, 最优化，推荐这样写！
std::string GetName() noexcept {
  std::string name("UMU618");
  return name;
}
```

## 避坑

没有必要对返回值再加一次 std::move，因为返回本身就已经是 move，再加一次就是多一次没必要的 move。
