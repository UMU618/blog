---
layout: post
title: Boost【6】函数式编程
date: 2021-07-23 20:54:22
categories: UMUTech
tags:
- boost
- cpp
- dev
---
本文可简单了……结论先行：尽量别用 Boost 进行函数式编程。

## 原因

1. 它们大部分已经被加入标准库，应该直接使用 STL。

| Boost | STL | Header |
| :- | -: | -: |
| Boost.Function | std::function | \<functional\> |
| Boost.Bind | std::bind | \<functional\> |
| Boost.Ref | std::ref, std::cref | \<functional\> |
| Boost.Lambda | lambda | part of C++11 |

2. Boost.Phoenix 是个例外，目前并没有 STL 可以完全替代，但它可读性不好，尽量不要用。

## 使用时机

有时候用 Boost.Phoenix 省事，因为它就像 lambda 表达式的模板，比如：

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

#include <boost/phoenix/phoenix.hpp>

bool is_odd(int i) {
  return i % 2 == 1;
}

int main() {
  std::vector<int> v{1, 2, 3, 4, 5};

  std::cout << std::count_if(v.begin(), v.end(), is_odd) << '\n';

  auto lambda = [](int i) { return i % 2 == 1; };
  std::cout << std::count_if(v.begin(), v.end(), lambda) << '\n';

  using namespace boost::phoenix::placeholders;
  auto phoenix = arg1 % 2 == 1;
  std::cout << std::count_if(v.begin(), v.end(), phoenix) << '\n';

  std::vector<long long> v2;
  v2.insert(v2.begin(), v.begin(), v.end());
  // warning
  // std::cout << std::count_if(v2.begin(), v2.end(), lambda) << '\n';

  std::cout << std::count_if(v.begin(), v.end(), phoenix) << '\n';
  std::cout << std::count_if(v2.begin(), v2.end(), phoenix) << '\n';

  std::cout << "arg1(): " << arg1(1, 2, 3, 4, 5) << '\n';

  auto value = boost::phoenix::val(2);
  std::cout << value() << '\n';
}
```

本例中采用 Boost.Phoenix 可以自动适配 int 和 long long，而 lambda 表达式是确定的 int 参数，传入 long long 会 warning。

C++14 支持基于类型推断的泛型 lambda 表达式，将上面代码改进一下，说明没必要使用 Boost.Phoenix：

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

bool is_odd(int i) {
  return i % 2 == 1;
}

int main() {
  std::vector<int> v{1, 2, 3, 4, 5};

  std::cout << std::count_if(v.begin(), v.end(), is_odd) << '\n';

  // C++14
  auto lambda = [](auto i) { return i % 2 == 1; };
  std::cout << std::count_if(v.begin(), v.end(), lambda) << '\n';

  std::vector<long long> v2;
  v2.insert(v2.begin(), v.begin(), v.end());
  std::cout << std::count_if(v2.begin(), v2.end(), lambda) << '\n';
}
```

## 延申讨论

**STL 和 Boost 都有的类应该用哪个？**

- 如果是标准库原本没有，Boost 先有，然后 Boost 的实现被加入标准库，那么应该使用标准库。

- 如果 Boost 加强了标准库的实现，那么就看标准库能不能满足您的需求，如果不能再采用 Boost 的。

- 因为依赖而必须采用 Boost，那就别费力去改用标准库。比如有些 Boost 库（比如 Boost.Log）使用了 boost::shared_ptr，这时候是不能简单地改用 std::shared_ptr 的。
