---
layout: post
title: VS2019 Makefile 型工程调用 Clang
date: 2020-08-31 23:11:53
categories: UMUTech
tags:
- debug
- dev
- windows
---
## 需求

VS2019 Makefile 型工程使用 Clang 编译，要同时支持 x86 和 x64 平台。

## 问题

找不到 Clang 参数可以指定目标平台，如果您知道请不吝赐教。（github 账号：UMU618）

## 解决

同时安装这两个平台的 Clang。x86 的由 VS2019 自带，在 VS 中被表示为 `"$(ClangAnalysisToolsPath)\clang.exe"`，宏展开后为：

```powershell
$ &"C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Tools\Llvm\bin\clang.exe" -v
clang version 10.0.0
Target: i686-pc-windows-msvc
Thread model: posix
InstalledDir: C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Tools\Llvm\bin
```

x64 是另外安装的，安装时选择把目录注册到 PATH：

```powershell
$ clang -v
clang version 10.0.0
Target: x86_64-pc-windows-msvc
Thread model: posix
InstalledDir: C:\Program Files\LLVM\bin
```

结论：只需在 Win32 的平台设置使用 `"$(ClangAnalysisToolsPath)\clang.exe"`，而 x64 则直接使用 `clang`（如果安装时忘记注册到 PATH，需要手动添加）。
