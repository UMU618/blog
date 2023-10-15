---
layout: post
title: 为什么应该使用 C++ 20？
date: 2023-10-15 19:23:01
description: 因为 Rust 太难了！
categories: UMUTech
tags:
- cpp
- dev
---
## 实战项目

1. [鎏光云游戏引擎](https://github.com/ksyun-kenc/liuguang)

2. [金山云 LiveNet](https://zhuanlan.zhihu.com/p/562199099)

3. [ClipboardSync](https://gitee.com/ksyun-kenc/ClipboardSync)

4. [PowerEconomizer](https://github.com/UMU618/PowerEconomizer)

其中，鎏光一开始是用 C++ 17 的，在 MSVC 的 std::format 可用时，第一时间切换到 C++ 20。而 LiveNet 主要运行于 Linux，开发时 gcc 还不支持 std::format，使用 fmt::format 代替，但一直用 cxxstd=20 编译。

ClipboardSync 和 PowerEconomizer 使用了 C++ 20 modules。

现在还在开发的云桌面产品也是使用 C++ 20，但没有用 modules。

## 理由

根据 [jetbrains](https://www.jetbrains.com/lp/devecosystem-2022/cpp/) 的统计，2022 年时，C++ 20 的使用率是 23%，已经超过经典 C++ 的 8%。在游戏开发领域，C++ 20 的使用率为 25%，甚至已经超过 C++ 14 的 24%。

以下列举能够很容易想到的一些好处：

1. 好用、安全的新类：std::format、std::span、std::jthread、原子(Atomic)智能指针

2. designated-initializers 安全初始化，防止因为调整结构体而顺序不对

3. modules 加速编译

4. 更多标签 [[likely]], [[unlikely]], [[nodiscard(reason)]]

5. 可以 using enum

6. 对模板形式的 Lambda 有更好支持

7. 范围 for 循环支持初始化

8. 三路比较运算符 <=>

9. Boost 的 awaitables 协程（BOOST_ASIO_HAS_CO_AWAIT）需要 C++ 20

10. ranges 库

## 阵痛

1. 曾经遇到 clang-format 对 modules 支持不好的问题，后来升级 clang-format 解决。但目前还不建议在大型项目里使用 modules。

2. 有些隐式转换无法编译，尤其在编译驱动代码时，容易遇到连 WDK 里的头文件都无法编译。这是因为 C++ 20 比 C++ 17 都严格。建议内核态驱动使用 C++ 17；用户态驱动可以 C++ 20。
