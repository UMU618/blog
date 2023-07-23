---
layout: post
title: 【GSL 系列 1】为什么有智能指针还要 gsl::final_action？
date: 2022-09-04 16:36:46
categories: UMUTech
tags:
- cpp
- dev
- gsl
- optimization
---
## 故事

稣看到一些代码使用手动方式管理资源，便打算安利《[Boost【2】ScopeExit](/2020/09/22/umutech-boost-2-scope-exit/)》减少心智负担，然而并非所有团队都能立刻接受 [Boost](/tags/boost/) 这么大的开发库，于是先推荐 [GSL](https://github.com/Microsoft/GSL)。

结果被问了这么一个问题：

- 用智能指针不行吗？

## 案例分析

### 1. 手动管理

假设有一种资源由 C 代码管理，还有一个可能抛出异常的函数，如下：

```cpp
extern "C" {

#include <stdio.h>

    void init() {
        printf("%s\n", __func__);
    }

    void uninit() {
        printf("%s\n", __func__);
    }

}

void something_may_throw() {
    std::cout << __func__ << '\n';
    throw std::exception("Bad news!");
}
```

那么手动管理的代码可能类似这样：

```cpp
void scope1() {
    std::cout << __func__ << '\n';

    try {
        init();
        something_may_throw();
        uninit();// may leak
    }
    catch (std::exception& ex) {
        std::cout << ex.what() << '\n';
    }

    std::cout << '\n';
}
```

它的实际运行结果将是：

```
scope1
init
something_may_throw
Bad news!
```

八哥在于 uninit 漏调用了！结论：**手动管理是有心智负担的！**

### 2. 使用智能指针

利用自定义智能指针的 deleter 来实现自动调用 uninit：

```cpp
void scope2() {
    std::cout << __func__ << '\n';

    try {
        init();
        std::shared_ptr<void> _(nullptr, [](void* p) -> void { uninit(); });
        something_may_throw();
    }
    catch (std::exception& ex) {
        std::cout << ex.what() << '\n';
    }

    std::cout << '\n';
}
```

运行结果：没有资源泄漏！

```
scope2
init
something_may_throw
uninit
Bad news!
```

但它有两个问题：

- 丑！

- 抽象代价高！

使用 [Compiler Explorer](https://godbolt.org/) 查看以上智能指针编译出来的汇编行数，就知道有多污染眼睛！

另外提醒，以下 unique_ptr 版本无法达到效果：

```cpp
void scope2u() {
    std::cout << __func__ << '\n';

    try {
        init();
        // nullptr 使 deleter 不被调用
        std::unique_ptr<void, void(*)(void* p)> _(nullptr, [](void* p) -> void { uninit(); });
        something_may_throw();
    }
    catch (std::exception& ex) {
        std::cout << ex.what() << '\n';
    }

    std::cout << '\n';
}
```

### 3. 使用 gsl::final_action

```cpp
void scope3() {
    std::cout << __func__ << '\n';

    try {
        init();
        gsl::final_action _{ [] { uninit(); } };
        something_may_throw();
    }
    catch (std::exception& ex) {
        std::cout << ex.what() << '\n';
    }

    std::cout << '\n';
}
```

运行结果同样完美无泄漏：

```
scope3
init
something_may_throw
uninit
Bad news!
```

并且可读性更好，其对应的汇编也更为简洁。

## 结论

C++ 是追求尽量降低抽象成本的，显然在这种场景下使用智能指针不如 [Boost.ScopeExit](/2020/09/22/umutech-boost-2-scope-exit/) 或 gsl::final_action 合适。
