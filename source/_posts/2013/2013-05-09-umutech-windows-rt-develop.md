---
layout: post
title: 开发 Windows RT 桌面应用（来自 Surface RT）
date: 2013-05-09 22:37:39
categories: UMUTech
tags:
- dev
- windows
- 作品
---
## 闲谈

这篇文章是用 Surface RT 写的。先喷一下这设备不爽的地方！

1. 请看 PPI 对比：

  - Surface RT = `sqrt(1366^2+768^2)/10.6` = 147.8

  - XPS 15 = `sqrt(1920^2+1080^2)/15.6` = 141.2

  居然才比 XPS 15 高了一小点！

2. 后摄像头成像质量太一般。

再来要说的是，微软的自残行为。UMU 用过 iOS、Android 平板，甚至见过有人用 XP 平板，但相信，论系统本身 Windows RT 是最强大的。不过微软为了战略目标，把 RT 强大的一面给锁起来了。对开发人员来说，这锁表现在以下几点：

1. 系统本身不允许运行没有微软签名的 EXE；

2. VS2012 默认无法编译 ARM 程序；

3. VS2012 自带的 ARM lib 缺失。

## 开始折腾

下面就是简单介绍一下如何突破这三个封锁：

### 1. 解锁签名限制

RT Jailbreak Tool By Netham45, Version 1.20

<http://forum.xda-developers.com/showthread.php?t=2092158>

另外，有很多开源软件已经移植，在开发自己的程序之前，可以先试试，Desktop apps ported to Windows RT：<http://forum.xda-developers.com/showthread.php?t=2092348>

### 2. 开启 VS2012 的 ARM 支持

来自 <http://stackoverflow.com/questions/11151474/can-arm-desktop-programs-be-built-using-visual-studio-2012> 的答案

> You can edit the file:
>
> `C:\Program Files (x86)\MSBuild\Microsoft.Cpp\v4.0\V110\Platforms\ARM\Microsoft.Cpp.ARM.Common.props`

对 VS2013 路径是：

`C:\Program Files (x86)\MSBuild\Microsoft.Cpp\v4.0\V120\Platforms\ARM\Platform.Common.props`

> In the<PropertyGroup>section add the line:
>
> `<WindowsSDKDesktopARMSupport>true</WindowsSDKDesktopARMSupport>`
>
> before `</PropertyGroup>`
>
> And that's all, you can build ARM desktop apps with VS2012.

某些工程需要强制定义 _ARM_WINAPI_PARTITION_DESKTOP_SDK_AVAILABLE 才可以。

### 3.获取更多的 ARM libs

开源工具应运而生：<https://github.com/peterdn/dll2lib>

然后，炫耀一下，UMU 已经把自己的一个小作品“[天翼宽带智能提速](/2011/12/13/umutech-tianyi-speed/
)”移植成功。这个程序比较小，一两个小时从解锁到移植开发全部搞定。

最后，如果程序是 .NET 4.x 写的，是可以直接跑在 RT 上的，所以为了省力气，也许应该考虑多用 .NET。