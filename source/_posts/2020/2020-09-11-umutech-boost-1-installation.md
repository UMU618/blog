---
layout: post
title: Boost【1】安装
date: 2020-09-11 14:22:09
categories: UMUTech
tags:
- cpp
- dev
- windows
---
## 软件环境

- Windows 10

- VS2019

其它系统的安装、编译过程都类似。即使您使用其它系统，本文仍然有参考意义。

## 1. 下载

官网：<https://www.boost.org/>，当前最新版本是 1.74.0。

在国内直接下载可能比较慢，您可以用~掩耳~下载，一般有 MB 级的速度。

## 2. 解压

假设解压到 D:\dev\boost_1_74_0，这个路径后面要设置为环境变量 BOOST_ROOT 的值。

最好检查目录结构，以防解压时弄错目录层级：

```powershell
# umutech @ UMU618 in D:\dev\boost_1_74_0 [14:31:58]
$ ls

    Directory:  D:\dev\boost_1_74_0

Mode                LastWriteTime     Length Name
----                -------------     ------ ----
d----         2020/8/11     23:28        1   boost
d----         2020/8/11     23:26        1   doc
d----         2020/8/11     23:28        1   libs
d----         2020/8/11     22:57        1   more
d----         2020/8/11     22:55        1   status
d----         2020/8/11     22:55        1   tools
-a---         2020/8/11     22:55      989   boost.css
-a---         2020/8/11     22:55     6.16KB boost.png
-a---         2020/8/11     22:55      850   boost-build.jam
-a---         2020/8/11     22:55    18.81KB boostcpp.jam
-a---         2020/8/11     22:55     2.39KB bootstrap.bat
-a---         2020/8/11     22:55    10.38KB bootstrap.sh
-a---         2020/8/11     22:55      769   index.htm
-a---         2020/8/11     23:28     5.46KB index.html
-a---         2020/8/11     22:55      291   INSTALL
-a---         2020/8/11     22:55    11.65KB Jamroot
-a---         2020/8/11     22:55     1.31KB LICENSE_1_0.txt
-a---         2020/8/11     22:55      541   README.md
-a---         2020/8/11     22:55     2.55KB rst.css
```

## 3. 添加 BOOST_ROOT 环境变量

- 您可以通过图形界面配置，右击【此电脑】-【属性】-【高级系统设置】-【环境变量】。新建一个，变量名为 BOOST_ROOT，变量值为 D:\dev\boost_1_74_0。

- 也可以使用命令行：`setx BOOST_ROOT D:\dev\boost_1_74_0`。

下面 Powershell 命令用于检查设置是否正确：

```powershell
# umutech @ UMU618 in D:\dev\boost_1_74_0 [14:39:22]
$ echo $env:BOOST_ROOT
D:\dev\boost_1_74_0
```

## 4. 编译 b2

直接运行 bootstrap.bat 即可，如果是非 Windows 系统，则是 bootstrap.sh。

```powershell
# umutech @ UMU618 in D:\dev\boost_1_74_0 [15:02:20]
$ .\bootstrap.bat
Building Boost.Build engine

Generating Boost.Build configuration in project-config.jam for msvc...

Bootstrapping is done. To build, run:

    .\b2

To adjust configuration, edit 'project-config.jam'.
Further information:

    - Command line help:
    .\b2 --help

    - Getting started guide:
    http://boost.org/more/getting_started/windows.html

    - Boost.Build documentation:
    http://www.boost.org/build/
```

## 5. 用 b2 编译 Boost

由于我们需要编译 Win32 和 x64 两种平台，所以给 b2 命令行加上个参数：

```powershell
# umutech @ UMU618 in D:\dev\boost_1_74_0 [15:03:43]
.\b2.exe --address-model=64
```
非 Windows 系统，可以直接运行 `./b2`。

新机器一般几分钟就能编译完毕。比如 OMEN 25L 只需要 3 分钟。~~廉想~~ L490 笔记本大约 14 分钟。

## 6. 配置 VS 工程

### 6.1 配置方法一

打开一个工程的属性页，定位到 VC++ Directories。

- Include Directories 加上 `$(BOOST_ROOT)`

- Library Directories 加上 `$(BOOST_ROOT)\stage\lib`

如果已经有其它值，记得用 ; 隔开。

### 6.2 配置方法二

打开一个工程的属性页。

- 定位到 C/C++，General，Additional Include Directories 加上 `$(BOOST_ROOT)`

- 定位到 Linker，General，Additional Library Directories 加上 `$(BOOST_ROOT)\stage\lib`

## 结语

- Boost 值得学习和使用。

- 本文本文对仅用 VS 写过 Hello world 的入门级程序员友好。（高手不会看到这末尾……）
