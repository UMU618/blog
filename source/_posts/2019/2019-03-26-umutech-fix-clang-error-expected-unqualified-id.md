---
layout: post
title: "修复 Clang 编译错误：error: expected unqualified-id"
date: 2019-03-26 17:57:31
categories: UMUTech
tags:
- dev
- macos
- cpp
- debug
---
## 问题

今天编译 [EOSIO](https://github.com/EOSIO)/[eos](https://github.com/EOSIO/eos) 出现一些 `error: expected unqualified-id`。

## 环境

- 操作系统：macOS Mojave
- 编译器：AppleClang 10.0.1.10010046
- SDK：MacOSX10.14.sdk
- Boost：1.69.0（1.67.0 也有问题，干脆用这个版本）

## 验证问题

```cpp
#include <signal.h>

int main() {
  ::sigset_t sigset;

  ::sigemptyset(&sigset);
  ::sigaddset(&sigset, SIGCHLD);
  return 0;
}
```

编译输出：

```
Scanning dependencies of target signal
[ 50%] Building CXX object CMakeFiles/signal.dir/signal.cpp.o
/Users/umu/umutech/macos-cpp/source/study/posix/signal/signal.cpp:6:5: error: expected unqualified-id
  ::sigemptyset(&sigset);
    ^
/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.14.sdk/usr/include/signal.h:125:26: note: expanded from macro 'sigemptyset'
#define sigemptyset(set)        (*(set) = 0, 0)
                                ^
/Users/umu/umutech/macos-cpp/source/study/posix/signal/signal.cpp:7:5: error: expected unqualified-id
  ::sigaddset(&sigset, SIGCHLD);
    ^
/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.14.sdk/usr/include/signal.h:122:31: note: expanded from macro 'sigaddset'
#define sigaddset(set, signo)   (*(set) |= __sigbits(signo), 0)
                                ^
2 errors generated.
make[2]: *** [CMakeFiles/signal.dir/signal.cpp.o] Error 1
make[1]: *** [CMakeFiles/signal.dir/all] Error 2
make: *** [all] Error 2
```

## 解决

去掉 `sigemptyset` 和 `sigaddset` 前面的 `::` 即可。因为他们是宏，宏都是全局的，用 `::` 修饰反而错了，严格！

Boost 开发分支上已经修复：

<https://github.com/boostorg/process/blob/develop/include/boost/process/detail/posix/wait_for_exit.hpp#L60>

<https://github.com/boostorg/process/blob/develop/include/boost/process/detail/posix/wait_group.hpp#L65>