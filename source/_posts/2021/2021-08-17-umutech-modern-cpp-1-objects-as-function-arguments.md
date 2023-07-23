---
layout: post
title: 现代 C++【1】类对象作为函数参数
date: 2021-08-17 22:49:46
categories: UMUTech
tags:
- cpp
- dev
---
## 前提

多现代？C++ 17，因为本文内含 `std::string_view`。

目前 C++ 20 还未普及，CLang 和 GCC 对 C++ 20 不是很上心，【直到今天 2020-08-18】连 std::format 都没有，被 MSVC 甩开。

## 问题

类对象作为参数究竟应该怎么传？

《[Effective C++](/2018/11/26/umutech-effective-cpp/)》的条款 20 说：

> 20. 宁以 pass-by-reference-to-const 替换 pass-by-value

为什么新规范又建议构造函数 pass-by-value？

## 原则

1. 只读访问并且不复制时，使用 pass-by-reference-to-const。

2. 需要保存对象副本时，并且对象可移动，使用 pass-by-value。

3. 对象很小时，使用 pass-by-value。

## 例子

```cpp
// 只读访问，不需要将 text 保存起来。
void Print(const std::string& text) {
  std::cout << text;
}

class Foo {
 public:
  // 需要保存 message 到类成员变量
  Foo(std::string message) : message_(std::move(message)) {}

 private:
  std::string message_;
};

// std::string_view 很小，连 const 都没必要加，就好像 int 型参数不会加 const
void CallCStyleApi(std::string_view dir) {
  if (0 < dir.size()) {
    chdir(dir.data());
  }
}
```

## 说明

Foo 的构造函数使用 pass-by-value，这使得它变成“两用”的，相当于针对这个类对象参数同时实现复制构造函数（ctor）、 移动构造函数（mtor）。

- 当传一个左值给它时，参数 message 是复制的，但它立刻移动给了成员变量 message_，整个过程发生一次复制和一次移动；

- 当传一个右值给它时，参数 message 是这个右值移动而来的，然后又立刻移动给了成员变量 message_，整个过程发生两次移动；

```cpp
std::string name("UMU618");
Foo f1(name);  // copy + move
Foo f2(std::move(name)); // move 2 times
```

如果类实现得妥当，移动两次对象实际上最多可以被编译器优化成零次。
