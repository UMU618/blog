---
layout: post
title: CppWinRT 经验【1】链接 winmm.dll 而不是 api-ms-win-mm-time-l1-1-0.dll
date: 2022-12-05 21:52:25
description: CppWinRT 排坑记【1】
categories: UMUTech
tags:
- cpp
- cppwinrt
- dev
- windows
---
## 前提

稣使用 ATL/WTL 开发 Windows 程序多年，慢慢地，它们就不太时髦，尤其是命名风格和 STL 不同，显得十分不现代。比如 [ATL::CComPtr](/2020/05/31/umutech-ccomptr-and-ccomqiptr/)，和 `std::unique_ptr` 确实风格迥异。

微软还搞了一套 WRL，例如：`Microsoft::WRL::ComPtr`，去掉了一个 C 是比 ATL 风格略好一点，但依然是不现代的（不像 STL 的）。

微软说：CppWinRT 才是王道，已经搞成现代 C++ 风格，例如：`winrt::com_ptr`。你们呀，用就行了。

好的！稣试试。不管用不用，先把它弄进来。这时候 `packages.config` 长得像下面：

```xml
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="Microsoft.Windows.CppWinRT" version="2.0.221121.5" targetFramework="native" />
</packages>
```

## 问题

原本设计支持 Windows 7 的程序突然无法在 Windows 7 正常运行了！提示找不到 `api-ms-win-mm-time-l1-1-0.dll`。

## 分析

找出原本可以在 Windows 7 正常运行的旧版本程序，发现其导入表里链接的是 `winmm.dll`，而不能正常运行的新版本则是链接并不存在的 `api-ms-win-mm-time-l1-1-0.dll`。

在 Windows 10 上，`api-ms-win-mm-time-l1-1-0.dll` 会被映射到 `kernel32.dll`，但 Windows 7 没有这个映射，所以报错。

那么只有让程序链接 `winmm.lib` 就行了……可是，一直就是链接它的呀！

尝试把 `#pragma comment(lib, "winmm.lib")` 去掉，居然不报错！

稣开始回忆最开始调用 time API 时，不加 `winmm.lib` 是链接不过的。很明显加了 CppWinRT 后，它的 lib 重载了 time API 的链接。

## 解决

CppWinRT 很好！稣用 [WIL](https://github.com/microsoft/wil)……例如：`wil::com_ptr`。

割爱吧！稣在 `Manage NuGet Packages...` 里卸载了 CppWinRT。
