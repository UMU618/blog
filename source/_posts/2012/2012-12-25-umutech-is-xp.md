---
layout: post
title: 判断系统是不是 XP
date: 2012-12-25 11:16:06
categories: UMUTech
tags:
- dev
- windows
---
## 需求

实现 Edit 控件文字透明，在 XP 下有 Bug，得做特殊处理，所以要判断当前系统是不是 XP，蛋似，要考虑到“兼容性”：如果系统是 Vista、Win7、8、9、10，则他们都有兼容 XP 的模式，GetVersion/GetVersionEx 都会被欺骗。

## 解决

这种情况需要的是：VerifyVersionInfo

## 参考

<http://msdn.microsoft.com/en-us/library/ms725492(v=vs.85).aspx>
