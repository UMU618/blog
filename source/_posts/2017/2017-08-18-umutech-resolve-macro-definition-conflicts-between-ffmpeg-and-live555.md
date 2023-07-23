---
layout: post
title: 解决 ffmpeg 与 live555 宏定义冲突
date: 2017-08-18 11:03:06
categories: UMUTech
tags:
- dev
- ffmpeg
- streaming-media
---
一个工程同时使用了 ffmpeg 和 live555，结果一不注意就混乱了……原因如下：

```cpp
// ffmpeg 的 error.h 里 include 了 errno.h，有以下定义：
#define EAGAIN          11


// 而 live555 的 NetCommon.h 里有以下定义：
#ifdef EAGAIN
#undef EAGAIN
#endif

// WSAEWOULDBLOCK == 10035
#define EAGAIN WSAEWOULDBLOCK
```

很明显，live555 这么做，违背了面向对象的基本特征——封装，这种平台相关的抽象应该封装在源文件里面，而不是放在头文件。挪个位置即可。
