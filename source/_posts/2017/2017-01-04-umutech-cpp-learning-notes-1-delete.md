---
layout: post
title: "[C++ 学习笔记 1] delete 和 delete [] 的本质区别"
date: 2017-01-04 23:25:53
categories: UMUTech
tags:
- cpp
- dev
---
本文宣告 UMU 正式开始学习 C++。~~之前只系统学过 C，自然地了解了一些 C++ 的皮毛（可以认为是 C+），然后就一直用着 C+ 开发，最近看了一些现代 C++ 代码，感觉是时候好好学习 C++ 了……后续会把学习中记的笔记发出来，尽量简短明了。~~

当 ptr 指向的是基础类型数组时，delete ptr 和 delete [] ptr 等价。这好比用 free 释放 malloc 分配的内存，malloc 了多少，不必关心，free 知道要释放多长，因为 malloc 会维护这个长度信息。

当 ptr 指向类对象数组时，两者的差别在于调用多少个析构函数，delete 只调用第一个元素的析构函数，delete [] 则调用所有元素的析构函数。

```cpp
#include <memory>
#include <iostream>

class Foo
{
public:
    Foo()
    {
        std::cout << __FUNCTION__ << std::endl;
    }

    ~Foo()
    {
        std::cout << __FUNCTION__ << std::endl;
    }
};

int main()
{
    // bug: 只会析构一个元素
    std::shared_ptr<Foo> p(new Foo[10]);

    return 0;
}
```
