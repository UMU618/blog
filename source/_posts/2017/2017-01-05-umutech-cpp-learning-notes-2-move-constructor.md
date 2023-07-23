---
layout: post
title: "[C++ 学习笔记 2] 为什么会有移动构造函数、std::move？"
date: 2017-01-05 17:30:26
categories: UMUTech
tags:
- cpp
- dev
---
UMU 认为有一个目的是：**需求细分**（另外还有优化的目的）。考虑以下代码：

```cpp
class Movable
{
public:
	Movable() : i(new int(3))
	{
		std::cout << __FUNCTION__ << std::endl;
	}

	Movable(Movable& m) : i(m.i)
	{
		m.i = nullptr; // 这里改变值是可以的
		std::cout << __FUNCTION__ << "&" << std::endl;
	}

	int* i;
};
```

因为 Movable& m 没有用 const 修饰，所以可以在内部改变 m 的状态。如果加上 const 则不行：

```cpp
Movable(const Movable& m) : i(m.i)
{
	//m.i = nullptr; // 不能改变 m
	std::cout << __FUNCTION__ << "&" << std::endl;
}
```

那么没加 const 的集合，减去有 const 的集合，等于什么？答案就是：移动构造函数

```cpp
Movable(Movable&& m) : i(m.i)
{
	m.i = nullptr;
	std::cout << __FUNCTION__ << "&&" << std::endl;
}
```

分成 const Movable& 和 Movable&& 两个，更严格、更清晰，这是好事。而 std::move 做的事情是为了正确调用移动构造函数（Movable&&），而不是被隐式转为 const 而错误地调用了复制构造函数（const Movable&），不要在意什么左值、右值的，太烧脑了……

扩展阅读：《从4行代码看右值引用》，<https://www.cnblogs.com/qicosmos/p/4283455.html>