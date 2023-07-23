---
layout: post
title: 【GSL 系列 2】为什么需要 gsl::narrow_cast？
date: 2022-09-18 15:07:30
categories: UMUTech
tags:
- cpp
- dev
- gsl
- optimization
---
## 问题

- gsl::narrow_cast 不就是 static_cast 吗？为啥要用 gsl::narrow_cast？

## 分析

gsl::narrow_cast 的注释写着：

> // narrow_cast(): a searchable way to do narrowing casts of values

其实已经很直白。用它的好处就是——以后要搜索将更容易！那么问题转变为：为什么要搜索？当然是因为将数据变窄是可能有潜在问题的！

下面的例子里，有 gsl::narrow_cast 把数据转窄，又有 static_cast 把数据转宽。如果只用 static_cast，那么在排查哪里把数据丢失时，需要将第二个无关的 static_cast 也查一遍，这就浪费时间了。

```cpp
#include <iostream>

#include <gsl/gsl>

int main()
{
	int i = 0xff;
	auto b = gsl::narrow_cast<std::uint8_t>(i);

	// std::cout << b;
	std::cout << static_cast<int>(b);
}
```

总之，**使用 gsl::narrow_cast 是为了写出更健壮更好维护的代码。**

## 更多

还有一个 gsl::narrow，会在转换丢失数据时抛出 gsl::narrowing_error 异常，适合在不允许数据丢失的场景使用。
