---
layout: post
title: 优化思维【1】字符串去空格
date: 2019-04-10 23:15:22
categories: UMUTech
tags:
- dev
- optimization
---
## In place 版本

传入的字符串将被改变。

为方便复用，一般会实现 ltrim 和 rtrim 两个函数，然后 trim 函数调用这两者实现。

- ltrim：从 str 头部开始找到非空格字符，偏移量记为 offset，将 str 左移（move）offset 个字符。

- rtrim：从 str 尾部向头部找到非空格字符，偏移量记为 offset，将 str 截断为 offset。

一个想当然的实现：

```
trim(str) {
    ltrim(str)
    rtrim(str)
}
```

假设 str 是 x 个空格 + y 个非空格 + z 个空格，则以上代码需要把 y + z 个字符向左移动 x 个位置。

更好的实现是：

```
trim(string& str) {
    rtrim(str)
    ltrim(str)
}
```

由于先截断，剩 x + y，再去左移，只需要把 y 个字符左移 x 个位置。

**优化思路：尽量减少复制，调整顺序也是优化手段。**

## Copy 版本

传入的字符串不会被改变，返回一个新的字符串。

一个复用前面代码的实现：

```
string trim_copy(string str) {
    // str is a copy
    rtrim(str)
    ltrim(str)
    return str
}
```

这个版本需要复制 x + y + z 个字符，ltrim 和 rtrim 里面都有找偏移量的代码可以复用，直接找到 y 个非空格字符是起点和终点，复制这 y 个字符就好了。

```
string trim_copy(const string& str) {
    // str is a copy
    l = lfind(str)
    r = rfind(str)
    return str[l, r]
}
```

**优化思路：尽量减少复制。**

## move

strlen、strcpy、memmove 这类函数，都有一个优化思路：机器字长对齐，一次处理一个机器字。对于长字符串，效果显著。

**优化思路：针对硬件特征调整策略。**
