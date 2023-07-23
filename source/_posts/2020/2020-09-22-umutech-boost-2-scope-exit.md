---
layout: post
title: Boost【2】ScopeExit
date: 2020-09-22 19:13:38
categories: UMUTech
tags:
- boost
- cpp
- dev
- windows
---
## 需求

- 资源有很多种，每种都封装一套，还是挺累的！对于比较少使用或者一个程序很可能只会用一次的资源，我们不想封装！

- Golang 的 defer 真香！

## 解决

利用 RAII 特性，封装个 ScopeGuard！或者直接用 Boost.ScopeExit。

```cpp
// umu/scope_exit.hpp
#pragma once

#include <functional>

// C++11

namespace umu {
class ScopeGuard {
 public:
  explicit ScopeGuard(std::function<void()> on_exit_scope)
      : on_exit_scope_(on_exit_scope), dismissed_(false) {}

  ~ScopeGuard() noexcept {
    if (!dismissed_) {
      on_exit_scope_();
    }
  }

  void Dismiss() { dismissed_ = true; }

 private:
  std::function<void()> on_exit_scope_;
  bool dismissed_;

  // noncopyable
  ScopeGuard(ScopeGuard const&) = delete;
  ScopeGuard& operator=(ScopeGuard const&) = delete;
};
}  // end of namespace umu

#define SCOPEGUARD_LINENAME_CAT(name, line) name##line
#define SCOPEGUARD_LINENAME(name, line) SCOPEGUARD_LINENAME_CAT(name, line)
#define ON_SCOPE_EXIT(callback) \
  umu::ScopeGuard SCOPEGUARD_LINENAME(EXIT, __LINE__)(callback)
```

## 范例

Windows 上使用 socket 必须先调用 WSAStartup 初始化 WinSock 环境。

### 常规封装版

```cpp
#include <winsock2.h>

#pragma comment(lib, "ws2_32.lib")

class WinSock {
 public:
  WinSock() : error_code_(WSAEFAULT) {}

  int Initialize(WORD version_requested = WINSOCK_VERSION) {
    assert(WSAEFAULT == error_code_);
    return error_code_ = ::WSAStartup(version_requested, &wsa_data_);
  }

  bool GetWsaData(WSADATA& wsa_data) {
    if (NO_ERROR == error_code_) {
      wsa_data = wsa_data_;
      return true;
    }
    return false;
  }

  int GetErrorCode() { return error_code_; }

  ~WinSock() {
    if (NO_ERROR == error_code_) {
      ::WSACleanup();
    }
  }

 private:
  int error_code_;
  WSADATA wsa_data_;
}

int main(int argc, char* argv[]) {
  WinSock winsock;
  int error_code = winsock.Initialize();
  if (NO_ERROR != error_code) {
    std::cerr << "Initialize() failed with " << error_code << '\n';
    return error_code;
  }

  // test codes
  SOCKET s = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);
  if (INVALID_SOCKET != s) {
    closesocket(s);
    std::cout << "OK\n";
  } else {
    error_code = WSAGetLastError();
    if (WSANOTINITIALISED == error_code) {
      std::cerr << "WSANOTINITIALISED\n";
    } else {
      std::cerr << "socket() failed with " << error_code << '\n';
    }
  }

  return error_code;
}
```

### Boost.ScopeExit 版

现在是 2020 年 9 月，建议使用 cpp17，所以抛弃 BOOST_SCOPE_EXIT + BOOST_SCOPE_EXIT_END，使用 cpp11 的 BOOST_SCOPE_EXIT_ALL。

```cpp
#include <winsock2.h>

#include <boost/scope_exit.hpp>

#pragma comment(lib, "ws2_32.lib")

int main(int argc, char* argv[]) {
  WSADATA wsa_data = {};
  int error_code = ::WSAStartup(WINSOCK_VERSION, &wsa_data);
  if (NO_ERROR != error_code) {
    std::cerr << "WSAStartup() failed with " << error_code << '\n';
    return error_code;
  }
  // 类似 Golang 的 defer
  BOOST_SCOPE_EXIT_ALL(&) { ::WSACleanup(); };

  // test codes
  SOCKET s = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);
  if (INVALID_SOCKET != s) {
    closesocket(s);
    std::cout << "OK\n";
  } else {
    error_code = WSAGetLastError();
    if (WSANOTINITIALISED == error_code) {
      std::cerr << "WSANOTINITIALISED\n";
    } else {
      std::cerr << "socket() failed with " << error_code << '\n';
    }
  }

  return error_code;
}
```

Dismiss 演示：

```cpp
boost::scope_exit::aux::guard<void> defer;
defer = [] { std::cout << "On scope exit!\n"; };
// dismiss
defer = {};
// Won't print "On scope exit!"
```

## 参考

<https://www.boost.org/doc/libs/1_74_0/libs/scope_exit/doc/html/BOOST_SCOPE_EXIT_ALL.html>
