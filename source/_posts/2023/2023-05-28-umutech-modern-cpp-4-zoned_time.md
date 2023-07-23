---
layout: post
title: 现代 C++【4】std::chrono::zoned_time
date: 2023-05-28 15:12:06
description: ChatGPT 叫稣教它 C++ 20！
categories: UMUTech
tags:
- cpp
- dev
---
## 问题

最近经历了一次半夜提交代码，却发现单元测试无法通过，而无法合并到主线的小事故。经过检查，是一个日志清理模块的实现有问题，一会儿使用 UTC，一会儿使用本地时间（东八区），导致只要在 \[0:00, 8:00\) 提交代码就无法通过单元测试！而平时都是 10 点上班，所以没长期发现。

在纠正实现的时候，首先想到可以用 _get_timezone 来修正时间，但它是个 CRT 函数，显得不够现代，所以打算用 C++ 20 来实现。

## 解决

先来看 C 和 C++ 混合的解决方式：

```cpp
long tz{};
_get_timezone(&tz);
auto local_now = std::chrono::system_clock::now() - std::chrono::seconds(tz);
```

这个代码除了不够现代，它还是 MS 特有的（Microsoft Specific），文档都埋坑（见文末）……C++ 20 里有跨平台的封装：std::chrono::zoned_time，下面用它来实现：

```cpp
#include <chrono>
#include <iostream>

int main() {
  // QPC 时间，非人类历法时间
  auto now = std::chrono::steady_clock::now();
  std::cout << "System boot time: " << now.time_since_epoch() << '\n';

  // 以下是人类历法时间
  auto utc_now = std::chrono::system_clock::now();
  std::cout << "UTC time: " << utc_now
            << ", timestamp: " << utc_now.time_since_epoch() << '\n';

  auto current_zone = std::chrono::current_zone();
  std::cout << current_zone->name()
            << " time: " << current_zone->to_local(utc_now) << ", timestamp: "
            << current_zone->to_local(utc_now).time_since_epoch() << '\n';

  std::chrono::zoned_time<std::chrono::system_clock::duration> local_time{
      std::chrono::current_zone(), utc_now};
  std::cout << "Local time: " << local_time.get_local_time()
            << ", timestamp: " << local_time.get_local_time().time_since_epoch()
            << '\n';

  std::cout << "Timezone: "
            << (utc_now.time_since_epoch() -
                local_time.get_local_time().time_since_epoch())
            << '\n';
}
```

可能的输出：

```
System boot time: 963185693626400ns
UTC time: 2023-05-28 07:59:21.9329686, timestamp: 16852607619329686[1/10000000]s
Asia/Shanghai time: 2023-05-28 15:59:21.9329686, timestamp: 16852895619329686[1/10000000]s
Local time: 2023-05-28 15:59:21.9329686, timestamp: 16852895619329686[1/10000000]s
Timezone: -288000000000[1/10000000]s
```

PS: 目前为止，g++ 对 C++ 20 支持不好，请用 MSVC 测试。

**注意事项**：`std::chrono::zoned_time` may throw if `location` is not in the time zone database. 需要 catch 类型为 std::chrono::nonexistent_local_time 的异常。

## _get_timezone 的坑

`_get_timezone` 的返回值的含义是 UTC 和 localtime 的差值，单位为秒，比如东八区是 -28800。它的实现是这样的：

```cpp
extern "C" errno_t __cdecl _get_timezone(long* result)
{
    _VALIDATE_RETURN_ERRCODE(result != nullptr, EINVAL);

    // This variable is correctly inited at startup, so no need to check if
    // CRT init finished.
    *result = _timezone.value();
    return 0;
}
```

目前它的[文档][_get_timezone]里并没有提到需要“前置调用”……如果直接使用，可能得到一个错误的默认值 28800，这是“西八区”的意思！正确的做法是调用 `_tzset`、`gmtime` 或 `localtime` 等函数后，再调用 _get_timezone。

## 参考

[_get_timezone]: https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/get-timezone

1. [_get_timezone]
2. `std::chrono::zoned_time`: <https://en.cppreference.com/w/cpp/chrono/zoned_time>
