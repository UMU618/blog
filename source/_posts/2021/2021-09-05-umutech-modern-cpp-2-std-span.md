---
layout: post
title: 现代 C++【2】std::span
date: 2021-09-05 18:20:06
categories: UMUTech
tags:
- cpp
- dev
---
## 前提

多现代？C++ 20。

C++ 17 才有 `std::string_view`，而相似的 `std::span` 居然到 C++ 20 才有。

## 问题

如何解决 C-Style 数组（包含动态分配的连续内存）的退化（array decay）和越界访问（range errors）两大问题？

## 解决

C 语言解决这两个问题，主要是增加一个长度参数。很多 Win32 API 这样做，例如：

```c
PCSTR WSAAPI inet_ntop(
  INT        Family,
  const VOID *pAddr,
  PSTR       pStringBuf,
  size_t     StringBufSize
);

int GetWindowTextA(
  HWND  hWnd,
  LPSTR lpString,
  int   nMaxCount
);
```

但它会带来新问题：不小心传错！另外也有一些地方并没有提供长度参数，比如下面 Linux 内核代码里的函数：

```c
static inline int ip_decrease_ttl(struct iphdr *iph);
```

当我们打算把 uint8_t 数组转成 `struct iphdr *` 时，必须在调用前保证数组长度大于等于最小 IP 头长度。

C++ 的解决方案是：`std::span`，它是一个连续对象存储的观察者。类似 `std::string_view` 是 `std::string` 的观察者。它可以同时管理数组的地址和大小，并且它没有数据所有权，仅占用最多两个指针的空间，可以像 `std::string_view` 一样在绝大多数时候直接按值传递。

## 例子

以下函数用于获取 IP 头的长度：

```cpp
std::uint8_t GetHeaderLength(const void* ip_header, size_t size) noexcept;

std::uint8_t ip[] = {0x45, 0x00, 0x00, 0x54, 0xfa, 0xa6, 0x40, 0x00,
                     0x40, 0x01, 0xb3, 0x9a, 0xc0, 0xa8, 0x0b, 0x02,
                     0xc0, 0xa8, 0x00, 0x15};
std::cout << "HeaderLength: " << (int)GetHeaderLength(ip, sizeof(ip)) << '\n';
```

它可以用 `std::span` 包装成：

```cpp
template <typename T, size_t N>
inline std::uint8_t GetHeaderLength(std::span<T, N> ip_header) noexcept {
  return GetHeaderLength(ip_header.data(), sizeof(T) * ip_header.size());
}

std::cout << "HeaderLength: " << (int)GetHeaderLength(std::span{ip}) << '\n';
```

另一个便利是，使用 subspan 成员函数可以对其内部指针和长度成对操作，以避免单独处理时可能不小心少处理一个的问题。

## 避坑

`std::span` 和 `std::string_view` 一样，没有数据所有权，所以要担心数据失效问题，不要在数据被释放后使用。

下面是个错误示范，来自：[std::string_view encourages use-after-free; the Core Guidelines Checker doesn't complain #1038](https://github.com/isocpp/CppCoreGuidelines/issues/1038)

```cpp
#include <iostream>
#include <string>
#include <string_view>

int main() {
  std::string s = "Hellooooooooooooooo ";
  std::string_view sv = s + "World\n";
  std::cout << sv;
}
```
