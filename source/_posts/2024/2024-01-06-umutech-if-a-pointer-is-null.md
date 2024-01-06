---
layout: post
title: 指针判空
date: 2024-01-06 14:48:08
description: ChatGPT 也认为少敲几个键比较养生！
categories: UMUTech
tags:
- dev
- coding-style
- cpp
---
[gcpp]: https://google.github.io/styleguide/cppguide.html
[cppcore]: https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#c-core-guidelines

## 引子

C 和 C++ 都有一些很基本的语句出现不同派系的写法，比如 * 靠左还是靠右，抑或居中？这种还是排版问题，并不影响有效字符数，但下面这个就直接影响有效字符数了！

《对于选择恐惧症患者来说，指针判空究竟要怎么写才不纠结？》

## 问题

指针判空有两大类写法：

- if \(p\) {}

- if (nullptr != p) {}

后者被挺多人推荐的，比如林锐的《[高质量C++/C编程指南](https://vrlab.org.cn/~zhuq/download/%E9%AB%98%E8%B4%A8%E9%87%8F%E7%BC%96%E7%A8%8B%E6%8C%87%E5%8D%97.pdf)》。

![高质量C++/C编程指南](/images/2024/20240106-linrui.png)

搜一下，能发现不少网友都挺纠结。~~这么基础的问题，如果不交代清楚，就是给 C++ 黑很好的攻击理由。~~

## 故事

稣刚大学毕业时，是使用 C，并不屑 C++ 的，当时看了不少代码，就是直接 `if (p)` 和 `if (!p)`，所以也坚持这种写法，并认为这样写比较短，对手指好，比较养生。

后来 C++17 出现，稣就坚定地改用 C++，于是也更多遵守 C++ 类型安全的原则，开始认为 `if (p)` 和 `if (!p)` 这种写法隐含类型转换，不太好。记得 2017 年以前，其实就看过 [Google C++ Style Guide][gcpp]，里面也曾经建议写成 `if (nullptr != p)` 和 `if (nullptr == p)`。

现在，再去看 [Google C++ Style Guide][gcpp]，已经没有这样的建议。

## 解决

[C++ Core Guidelines][cppcore] 中有一条：[ES.87: Don’t add redundant == or != to conditions](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-if)，它的理由是：

> **Reason** Doing so avoids verbosity and eliminates some opportunities for mistakes. Helps make style consistent and conventional.

if、while、for 的条件语句本身就是在选择 true 或 false，对于指针来说，会自动与 nullptr 比较。通常来说，`if (p)` 可以读作 “if p is valid”。

接着，还有一个例子：

```cpp
if (auto pc = dynamic_cast<Circle>(ps)) { ... } // execute if ps points to a kind of Circle, good

if (auto pc = dynamic_cast<Circle>(ps); pc != nullptr) { ... } // not recommended
```

同样是推荐更简短的写法。但这个例子却有人提出疑问：推荐的写法不会被怀疑是 == 少写了一个 = 吗？

还真不会！这个语句其实不是经典 C++ 的语法，而是 C++17 的 init-statement 语法，这里的 auto 是类型，说明 = 不可能是 == 错误地少写成 =。

对于部分 C 程序员和“经典 C++”程序员会把赋值语句写在 if 条件里的做法，建议是改为分开写。

```cpp
// p is defined before

if (p = func()) { ... } // bad

p = func();
if (p) { ... } // good

// p is used after
```

稣会在新工程里坚持使用 [C++ Core Guidelines][cppcore] 的建议，但也不反对另一种写法，只要不混合使用。~~允许有不同派系，但最好别精神分裂。~~
