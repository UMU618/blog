---
layout: post
title: GetEnvironmentStrings 函数的八哥史
date: 2023-09-13 23:18:42
description: 稣教 ChatGPT 用 API
categories: UMUTech
tags:
- debug
- dev
- windows
---
## 故事

故事总由八哥开始！今天稣看到一个 buggy 的 API 声明：

```c
#ifdef UNICODE
#define GetEnvironmentStrings  GetEnvironmentStringsW
#else
#define GetEnvironmentStringsA  GetEnvironmentStrings
#endif // !UNICODE
```

![GetEnvironmentStrings](/images/2023/20230913-get-environment-strings.jpg)

但是经历过 OutputDebugString 逆向的稣十分淡定地推测，这一定是故意的！毕竟，微软为了兼容性，啥都干得出来。

> 稣的逆向经验：一般 API 都是 A 的版本调用 W，而 OutputDebugString 是例外，OutputDebugStringW 调用 OutputDebugStringA。

## 搜索知识

找到 Raymond Chen 写的《[A brief history of the GetEnvironmentStrings functions](https://devblogs.microsoft.com/oldnewthing/20130117-00/?p=5533)》。原来，这个 API 早在 Windows NT 3.1 时就烙下八哥！

> The Get­Environment­Strings function has a long and troubled history.
>
> The first bit of confusion is that the day it was introduced in Windows NT 3.1, it was exported funny. The UNICODE version was exported under the name Get­Environment­StringsW, but the ANSI version was exported under the name Get­Environment­Strings without the usual A suffix.
>
> A mistake we have been living with for over two decades.

虽然后来可以解决这个例外，但微软选择保留此例外。

## 结论

大家可以不必担心相关的可能问题，因为现代的 Windows 会同时导出 GetEnvironmentStrings 和 GetEnvironmentStringsA。

![GetEnvironmentStrings functions](/images/2023/20230913-get-environment-strings-functions.jpg)
