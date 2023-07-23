---
layout: post
title: __BASE_FILE__ 和 __STEM__
date: 2022-12-04 17:38:05
description: 防逆向
categories: UMUTech
tags:
- cpp
- dev
- optimization
- security
- windows
---
## 问题

为了更好追踪产品 bug，程序员很可能在日志里打印代码文件名和行号。例如：

```cpp
std::clog << "(" __FILE__ ":" << __LINE__ << "): Failed to initialize!\n";
```

以上代码有两个问题：

1. 打印出代码文件的全路径，可能太长影响阅读，而且也没有必要。

2. 代码文件的全路径暴露在二进制文件（可执行程序）里，有一定安全风险，也更容易被逆向。

## 分析

第一个问题很容易，打印文件 base name 即可。代码可能如下：

```cpp
consteval std::string_view GetFileBaseName(std::string_view path) noexcept {
  return path.substr(path.rfind('\\') + 1); // for Windows path only
}

// Test
std::cout << GetFileBaseName({__FILE__, sizeof(__FILE__) - 1}) << '\n';
std::cout << GetFileBaseName(__FILE__) << '\n';
```

但是，第二个问题并没有解决，`__FILE__` 依然存在于二进制文件里，用任意十六进制编辑器都能很轻易地找到“代码文件的全路径”。

## 解决

方法一：去掉 `Use Full Paths (/FC)` 即可把 `__FILE__` 设置为只有文件名，没有全路径。

![Use Full Paths](/images/20221205-fc.png)

方法二：改用 `__BASE_FILE__` 吧！新问题是：msvc 不支持。那就自己定义一个：

```sh
/D__BASE_FILE__="\"%(Filename)%(Extension)\""
```

接着又想：后缀名有必要吗？如果保持优良习惯，从不在头文件里打日志，那确实没必要。于是再定义一个“文件主干名”：

```sh
/D__STEM__="\"%(Filename)\""
```

## 更多

`std::source_location` 依赖 `__FILE__` 或 `__builtin_FILE()`，所以如果开了 `/FC`，会有一样的安全问题。
