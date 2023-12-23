---
layout: post
title: BoringSSL? LibreSSL? OpenSSL!
date: 2023-12-24 00:01:26
description: 掐鸡说开才是对的！稣就是开。
categories: UMUTech
tags:
- boost
- cryptology
- dev
---
## 问题

- Windows 自带 OpenSSH 用的是 LibreSSL。要不要也用一下？甚至，直接动态链接到系统自带的 LibreSSL 的 dll，减少 exe 体积！

- Chrome 用的是 BoringSSL，要不要……折腾一下？

## 尝试

稣使用 vcpkg，安装几个库，还不是手到擒来？

结果……连装都不让装！这是 LibreSSL 的：

```
Building libressl:x64-windows...
CMake Warning at ports/libressl/portfile.cmake:2 (message):
  Can't build libressl if openssl is installed.  Please remove openssl, and
  try install libressl again if you need it.  Build will continue since
  libressl is a subset of openssl
```

这是 BoringSSL 的：

```
Building boringssl:x64-windows...
CMake Error at ports/boringssl/portfile.cmake:2 (message):
  Can't build BoringSSL if OpenSSL is installed.  Please remove OpenSSL, and
  try to install BoringSSL again if you need it.  Build will continue since
  BoringSSL is a drop-in replacement for OpenSSL
```

在忍痛 `vcpkg remove openssl` 后，陷入一阵思考——稣主要通过 Boost.Asio 使用 SSL，所以问题转换为：Boost.Asio 对这几个 SSL 库支持得如何？其中，OpenSSL 是使用多年没有任何问题的，只需要调研其它两个！

先试试 LibreSSL，毕竟是 OpenBSD 的，而且微软也用它。拿 [ClipboardSync](https://github.com/UMU618/ClipboardSync) 代码编译，发现顺利通过！但是运行时抛了异常，说不支持 TLS 1.3……遂查阅[官网](https://ftp.openbsd.org/pub/OpenBSD/LibreSSL/libressl-3.2.0-relnotes.txt)，说是从 3.2.0 就支持！那就是 Boost.Asio 不对了，果断给它提 [issue](https://github.com/chriskohlhoff/asio/issues/85#issuecomment-1850290442)。

接着尝试 BoringSSL，毕竟是 Google 的，号称重视安全，而且有 Chrome 这个大型流行软件做背书！然而很打脸的是：它居然不支持 SM3！稣当年特地选择用国密标准里的 SM3 做 Hash 算法，就是因为爱国！不支持国密这点岂能忍？立刻 `vcpkg remove boringssl`。

## 结论

Boost.Asio 和稣联合推荐 OpenSSL 为唯一好用又爱国的 SSL 库。
