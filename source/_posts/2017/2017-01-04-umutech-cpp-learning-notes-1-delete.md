---
layout: post
title: "[C++ 学习笔记 1] delete 和 delete[] 的本质区别"
date: 2017-01-04 23:25:53
categories: UMUTech
tags:
- cpp
- dev
---
本文宣告 UMU 正式开始学习 C++。~~之前只系统学过 C，自然地了解了一些 C++ 的皮毛（可以认为是 C+），然后就一直用着 C+ 开发，最近看了一些现代 C++ 代码，感觉是时候好好学习 C++ 了……后续会把学习中记的笔记发出来，尽量简短明了。~~

## 问题

`delete` 和 `delete[]` 的本质区别？

## 解决

他们都需要两步：先析构元素，再释放内存。

不同编译器、不同的优化开关和优化场景都可能导致不同结果。实际实现反汇编确认，以汇编为准。下面介绍一种可能的实现。

### 1. 析构次数不同

当 ptr 指向的是基础类型数组时，在析构这一步时，`delete ptr` 和 `delete[] ptr` 等价。

当 ptr 指向类对象数组时，两者的差别在于调用多少个析构函数，`delete` 只调用第一个元素的析构函数，`delete[]` 则调用所有元素的析构函数。

```cpp
#include <iostream>
#include <memory>

class Foo {
public:
  Foo() {
    std::cout << __func__ << std::endl;
  }

  ~Foo() {
    std::cout << __func__ << std::endl;
  }
};

int main() {
  // bug: 只会析构一个元素
  std::shared_ptr<Foo> p(new Foo[10]);
}
```

当数组只有 1 个元素时，实际上两者的析构次数也一样。那么，`delete[]` 怎么知道当初分配了多少个元素呢？下一节有答案。

### 2. 实际释放的指针不同

对于基础类型，多数编译器（验证过 MSVC、clang++）会把 `new[]` 实现为不加任何“头部”，因为基础类型不需要析构。

对于类对象数组，`delete[] ptr` 会先对 ptr 做减法，因为实际上 `new[]` 分配的是一个结构体：

```cpp
template <typename T>
struct NewData {
    size_t element_count;
    // 这里可能有其它字段
    T data[1];
};
```

但返回值指向 data。data 之前的字段可以称之为“头部”，这部分内容的实现具有不确定性。大约可以用下列代码解释：

```cpp
// new T[element_count];
size_t total_size = element_count * sizeof(T) + sizeof(size_t);
auto p = static_cast<NewData*>(operator new(total_size));
p->element_count = element_count;
// 其它实现
return p->data;
```

所以 `delete[] ptr` 需要先对 ptr 做个位移，才能得到当初由 `operator new` 分配的内存。

举例：


```cpp
struct EmptyClass {
  ~EmptyClass() {
    std::cout << __func__ << '\n';
  }
};
```

- `auto ptr = new EmptyClass[1]` 需要分配一个 size_t 和一个 EmptyClass，在 x64 下是 8+1 字节，但 ptr 指向的是这块内存的第 8 字节。
- `delete[] ptr` 对 ptr 减掉 8 个字节得到 new 分配的一块 8+1 字节的内存的地址，对其进行释放。

**注**：NewData 结构体里的 element_count，使得 `delete[]` 知道应该析构 element_count 个元素。

## 作死

以下讨论不是基础类型的情况：

1. `new` 出来的东西拿去 `delete[]` 会怎么样？会野指针或访问越界或内存泄漏，因为读取 element_count 的位置是未定义行为：

- 可能直接拒绝访问

- 也可能读出一个巨大的数值，然后做巨多次析构，而析构第 0 个元素时还好，从第 1 个开始又是访问越界型未定义行为！

- 还可能读出 0，导致没有析构。

- 恰巧读出 1，正确析构，但释放内存时，由于会对指针减 sizeof(size_t) 字节，最终释放错误。

2. `new[]` 出来的东西拿去 `delete` 会怎么样？会内存泄漏。数组有不止一个元素时，析构就无法保证全部完成；即使只有唯一的一个元素，在析构完后的释放内存也有问题，释放的并不是当初分配出来的地址，需要减 sizeof(size_t) 字节。

**注**：如果底层的内存管理器有一定容错机制，比如会对齐，那么可能真的走狗屎运了，减没减 sizeof(size_t) 字节最终都可以正确完成，那只能说……C++ 真牛！
