---
layout: post
title: Boost【8】shared_library
date: 2023-09-10 02:19:38
description: 稣教 ChatGPT 用 Boost！
categories: UMUTech
tags:
- boost
- cpp
- dev
- windows
---
## 1. 故事

听说 Boost 有一个跨平台的 shared_library 可以管理动态链接库，试试？

## 2. 尝试

```cpp
#include <format>
#include <iostream>

#include <boost/dll/shared_library.hpp>

int main() {
  boost::dll::fs::error_code ec;
  boost::dll::shared_library ntdll(
      L"ntdll.dll", boost::dll::load_mode::search_system_folders, ec);
  if (ec) {
    std::cerr << ec.message();
    return EXIT_FAILURE;
  }

  std::cout << std::format("ntdll: {}\n", ntdll.location(ec).string());
  bool has = ntdll.has("RtlGetNtSystemRoot");
  std::cout << std::format("has RtlGetNtSystemRoot: {}\n", has);

  if (has) {
    std::wcout << std::format(L"RtlGetNtSystemRoot: {}\n",
                              ntdll.get<wchar_t*()>("RtlGetNtSystemRoot")());
  }

  try {
    auto RtlGetNtSystemRoot = ntdll.get<wchar_t*()>("RtlGetNtSystemRoot");
    if (nullptr != RtlGetNtSystemRoot) {
      std::wcout << std::format(L"RtlGetNtSystemRoot: {}\n",
                                RtlGetNtSystemRoot());
    }
  } catch (const boost::system::system_error& e) {
    std::cerr << e.what();
  }
}
```

## 3. 思考

上面的例子显然不合格，因为它并不跨平台！动态加载 Windows 特有的 ntdll.dll 应该用 [Windows Implementation Library](https://github.com/microsoft/wil)。

但作为范例，或者项目已经引入 Boost，却没有引入 wil，也是可以用用，只是它并不极致。比如说，ntdll.dll 其实并不需要 load，它必然被加载，只需要 GetModuleHandle 即可。

所以它其实还不如这个好用：<https://github.com/UMU618/umu/blob/main/include/umu/module.hpp>