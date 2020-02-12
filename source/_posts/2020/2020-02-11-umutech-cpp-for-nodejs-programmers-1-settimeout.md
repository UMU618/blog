layout: post
title: Node.js 程序员的 C++ 进修指南【1】：SetTimeout
date: 2020-02-11 21:49:15
description:
categories: UMUTech
tags:
- cpp
- nodejs
---
# 宗旨

- 如果您看得懂，那么，这是 Node.js 程序员的 C++ 进修指南。

- 如果您没看懂，那么，这是学 C++ 的劝退书！

# 第一个例子

在《[学习 Rust【2】减少代码嵌套](/2020/01/08/umutech-learn-rust-2-reduce-nesting/)》中，UMU 提到一个使代码平坦化的例子，咱们把其中最基本的功能提炼出来，成为最简单的例子：

```js
setTimeout(() => {
  console.log('step1')
}, 1000)
```

这段代码实现的功能是：一秒后打印 `step1`，退出。

# 翻译为 C++

首先明确一点：JavaScript 是 JIT 语言，不用编译，语言宿主直接解释运行。C++ 是 AOT 语言，需要编译。所以我们需要编译器（比如 g++、clang++）和编译脚本（比如 make、cmake）。下面我们会选择在 macOS 上使用 clang++ 和 cmake 来编译 C++ 代码。其中，cmake 其实是用来产生 Makefile 的，如果您学过 Makefile，可以直接用它。

## 安装依赖软件

- 安装 Xcode，以获取 MacOSX.sdk。

- 安装 clang++ 和 cmake：

```bash
brew install llvm
brew install cmake
```

## STL 实现

- C++ 代码：

```cpp
// set_timeout.cc
// UMU: 这是一个不太好的实现
#include <chrono>
#include <future>
#include <iostream>
#include <vector>

class Timer {
 public:
  template <typename T>
  void setTimeout(T function, int delay);
  void stop();
  void wait();

 private:
  bool clear = false;
  std::vector<std::thread> pool;
};

template <typename T>
void Timer::setTimeout(T function, int delay) {
  this->clear = false;
  pool.emplace_back(std::thread([=]() {
    if (this->clear) {
      return;
    }
    std::this_thread::sleep_for(std::chrono::milliseconds(delay));
    if (this->clear) {
      return;
    }
    function();
  }));
}

void Timer::stop() {
  this->clear = true;
}

void Timer::wait() {
  for (std::thread& thread : pool) {
    if (thread.joinable()) {
      thread.join();
    }
  }
}

int main() {
  Timer timer;
  timer.setTimeout([]() { std::cout << "step1\n"; }, 1000);
  timer.wait();
  return 0;
}
```

- cmake 脚本，CMakeLists.txt：

```cmake
cmake_minimum_required (VERSION 3.5)
project (set_timeout)

add_executable(set_timeout set_timeout.cc)
set_property(TARGET set_timeout PROPERTY CXX_STANDARD 17)
target_compile_features(set_timeout PRIVATE cxx_auto_type)
```

**小结**：以上代码，可用，但它是用多线程模拟的定时器，当设置 N 个定时器时，将创建 N 个线程，这不够优雅。

## Boost Asio 实现

我们知道，Nodejs 内部使用 libuv 作为异步 IO 库，它是 C 实现的，用 C++ 调用 libuv 就显得不那么 C++，所以我们决定用和 libuv 同类且更强大的 Boost Asio 来代替。

- 安装 boost，目前是 1.72.0 版：

```bash
brew install boost
```

- C++ 代码：

```cpp
#include <chrono>
#include <iostream>
#include <vector>

#include <boost/asio.hpp>
#include <boost/asio/steady_timer.hpp>

class Timer {
 public:
  template <typename T>
  void setTimeout(T function, int delay);
  void stop();
  void wait();

 private:
  boost::asio::io_service io;
  std::vector<boost::asio::steady_timer> timers;
  size_t count;
};

template <typename T>
void Timer::setTimeout(T function, int delay) {
  auto& timer = timers.emplace_back(boost::asio::steady_timer(io));
  ++count;
  timer.expires_from_now(std::chrono::milliseconds(delay));
  timer.async_wait([=](const boost::system::error_code& error) {
    // boost::system::error::operation_canceled
    // boost::asio::error::operation_aborted
    if (!error) {
      function();
    }
  });
}

void Timer::stop() {
  for (auto& timer : timers) {
    timer.cancel();
  }
}

void Timer::wait() {
  io.run();
}

int main() {
  Timer timer;
  timer.setTimeout([&]() { std::cout << "step1\n"; timer.stop(); }, 1000);
  //timer.setTimeout([]() { std::cout << "step1\n"; }, 1000);
  timer.setTimeout([]() { std::cout << "step2\n"; }, 2000);
  timer.wait();
  return 0;
}
```

- cmake 脚本，CMakeLists.txt：

```cmake
cmake_minimum_required (VERSION 3.5)
project (set_timeout)

add_executable(set_timeout set_timeout.cc)
set_property(TARGET set_timeout PROPERTY CXX_STANDARD 17)
target_compile_features(set_timeout PRIVATE cxx_auto_type)

set(Boost_USE_STATIC_LIBS ON)
set(Boost_USE_MULTITHREADED ON)
set(Boost_USE_STATIC_RUNTIME ON)
find_package(Boost 1.71.0)
if(Boost_FOUND)
  include_directories(${Boost_INCLUDE_DIRS})
  target_link_libraries(set_timeout ${Boost_LIBRARIES})
endif()
```

**小结**：太难了……这真的是劝退书！
