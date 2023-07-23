---
layout: post
title: 优化思维【2】有符号和无符号的本质区别
date: 2019-05-16 23:07:02
categories: UMUTech
tags:
- dev
- cpp
- optimization
---
## 做题

以下代码打印什么？

```
auto count = sizeof (int);
if (count > -1) {
  std::cout << "> -1";
}
else {
  std::cout << "<= -1";
}
```

答案是：`<= -1`，因为 sizeof (int) 是无符号的，把 `auto` 改为 `int` 则结果是 `> -1`。

## 本质论

当我们声明 `unsigned/signed int count` 时，`unsigned/signed` 是变量 `count` 的使用属性，`int` 是其容量属性。

所谓使用属性，就是当它存在寄存器或内存时，不管是 `unsigned` 还是 `signed` 本质是一样的，但对它进行访问时，就区别对待。

比如对 `count` 进行加法，`unsigned` 时用的是 `ADD` 指令，`signed` 时用的是 `ADC` 指令，其余减乘除也都类似地使用不同指令。

再来一个问题：把一个变量保存到文件里，再读出来，怎么知道它是有符号还是无符号？

答案是：如果你不在序列化时考虑符号，则反序列化时，无法知道原来的符号，把它赋值给什么类型的变量它就变成什么类型。

这也是 JSON 文本转对象后，要自己选择数据类型的原因，因为 JSON 文本没表示符号的语法。

## 结论

**优化思路：理解本质，就能了解限制和优化方向。**
