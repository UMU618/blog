---
layout: post
title: 用 VS2019 应该尽量链接带有 Spectre 缓解措施的库
date: 2020-05-19 17:29:35
description:
categories: UMUTech
tags:
- debug
- dev
- windows
---
## 问题

愉快地装完 VS2019，编译一个使用 ATL 的工程，结果失败。

> LINK : fatal error LNK1104: cannot open file 'atls.lib'

## 分析

看 VC++ Directories 里的 Library Directories，有一个 `C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Tools\MSVC\14.25.28610\atlmfc\lib\spectre\x64`，但这个目录并没有 `atls.lib`。

反而 `C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Tools\MSVC\14.25.28610\atlmfc\lib\x64` 目录下有 `atls.lib`。

## 解决

安装 VS 时，应该选择“带有 Spectre 缓解措施、适用于最新 v142 生成工具的 C++ ATL (x86 和 x64)”。
