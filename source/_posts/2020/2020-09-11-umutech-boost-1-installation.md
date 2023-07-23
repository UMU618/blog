---
layout: post
title: Boost【1】安装
date: 2020-09-11 14:22:09
categories: UMUTech
tags:
- boost
- cpp
- dev
- windows
---
## 软件环境

- Windows 10

- VS2019

其它系统的安装、编译过程都类似。即使您使用其它系统，本文仍然有参考意义。

其它系统请参考：

- 《[Boost【3】在 Linux 上安装](/2021/07/06/umutech-boost-3-installation-on-linux/)》

- 《[Boost【4】在 macOS 上安装](/2021/07/09/umutech-boost-4-installation-on-macos/)》

## 1. 下载

官网：<https://www.boost.org/>，当前（2020-09-11）最新版本是 1.74.0。

在国内直接下载可能比较慢，您可以用~~掩耳~~下载，一般有 MB 级的速度。

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

```powershell
cd D:\dev\boost_1_74_0
#$env:BOOST_ROOT=$pwd.Path
Set-Item -Path env:BOOST_ROOT -Value $pwd.Path
setx BOOST_ROOT $pwd.Path
```

下面 Powershell 命令用于检查设置是否正确：

```powershell
# umutech @ UMU618 in D:\dev\boost_1_74_0 [14:39:22]
$ echo $env:BOOST_ROOT
D:\dev\boost_1_74_0
```

## 4. 编译 b2

直接运行 bootstrap.bat 即可，~~但是这样编译出来的是 32 位的 b2.exe~~。

如果您需要明确地编译为 x64 的 b2.exe，可以在 `x64 Native Tools Command Prompt for VS 2019` 下面运行 bootstrap.bat。

如果是非 Windows 系统，则是 bootstrap.sh。

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

为了方便调用 b2，可以把 BOOST_ROOT 加入到 PATH

```powershell
# setx PATH "$env:PATH;%BOOST_ROOT%"

# $target = 'User'
# $oldPath = [Environment]::GetEnvironmentVariable('Path', $target)
# $newPath = $oldPath
# if (!$oldPath.EndsWith(';')) {
#     $newPath += ';'
# }
# $newPath += '%BOOST_ROOT%'
# [Environment]::SetEnvironmentVariable('Path', $newPath, $target)

# Get-ItemPropertyValue -Path HKCU:\Environment -Name "Path"

$oldPath = (Get-Item -Path "HKCU:\Environment").GetValue('Path', '', 'DoNotExpandEnvironmentNames')
$newPath = $oldPath
if (!$oldPath.EndsWith(';')) {
    $newPath += ';'
}
$newPath += '%BOOST_ROOT%'
Set-ItemProperty -Path HKCU:\Environment -Name 'Path' -Value $newPath
```

## 5. 生成 user-config.jam

可以复制一个到 $HOME 目录，如有定制需求可以再编辑：

```powershell
cp .\project-config.jam $HOME\user-config.jam
```

## 6. 用 b2 编译 Boost

完全编译比较耗时，但后期比较省事：

```powershell
.\b2.exe --build-type=complete
```

新机器一般几分钟就能编译完毕。比如 OMEN 25L 只需要 9 分钟。~~廉想~~ L490 笔记本大约 37 分钟。

【推荐】如果只想用 lib，不想用 dll，可以节省编译时间：

```powershell
.\b2.exe address-model=32 address-model=64 link=static runtime-link=shared runtime-link=static
```

下面将介绍参数的意义，如果已选择完全编译，本节下面内容可以跳过。

以下命令行可以编译 x64 平台：

```powershell
.\b2.exe address-model=64
```

以上命令将同时编译 runtime-link=shared 的 Debug 和 Release 两种配置，产生的 lib 文件名会分别带有 `-mt-gd-x64` 和 `-mt-x64`，比如：

- Debug 版本：libboost_program_options-vc142-mt-gd-x64-1_75.lib

- Release 版本：libboost_program_options-vc142-mt-x64-1_75.lib

您有可能需要编译不同配置的版本，比如指定 runtime-link 为 static，这样可以不依赖 VC 的运行时 DLL，此时您可以用下面命令：

```powershell
.\b2.exe address-model=64 runtime-link=static
```

它编译出来的 lib 文件名中会带有 `-mt-s-x64`，例如：libboost_program_options-vc142-mt-s-x64-1_75.lib。

## 7. 测试编译

可以用以下仓库验证前面操作是否正确：

<https://github.com/UMU618/test_boost>

```powershell
git clone https://github.com/UMU618/test_boost
cd test_boost
.\build.cmd
# ./build.sh
```

请根据操作系统选择合适的脚本编译，最终编译出来的程序应该打印“OK!”。

## 8. 配置 VS 工程

### 8.1 配置方法一

打开一个工程的属性页，定位到 VC++ Directories。

- Include Directories 加上 `$(BOOST_ROOT)`

- Library Directories 加上 `$(BOOST_ROOT)\stage\lib`

如果已经有其它值，记得用 ; 隔开。

### 8.2 配置方法二

打开一个工程的属性页。

- 定位到 C/C++，General，Additional Include Directories 加上 `$(BOOST_ROOT)`

- 定位到 Linker，General，Additional Library Directories 加上 `$(BOOST_ROOT)\stage\lib`

## 结语

- Boost 值得学习和使用。

- 本文对仅用 VS 写过 Hello world 的入门级程序员友好。

- 更多应用 Boost 可以参考[鎏光云游戏引擎](https://github.com/ksyun-kenc/liuguang)。
