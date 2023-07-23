---
layout: post
title: Boost【5】Boost.IO
date: 2021-07-15 20:12:28
categories: UMUTech
tags:
- boost
- cpp
- dev
---
本文代码：<https://github.com/UMU618/test_boost>

## 1. ios_state

### 痛点

使用 std::cout 指定进制打印数字时经常有一个烦恼：之前设置的进制会一直有效，比如临时想打印一个 16 进制数，然后都打印 10 进制，这时候需要 std::hex，打印，再 std::dec，如果忘记 std::dec，那么后面的数字就全是输出 16 进制形态了……而且，您怎么知道之前用的就是 std::dec？万一是 std::oct 呢？

### 解决方案

使用 boost::io::ios_all_saver 自动保存和还原 ios 状态。

```cpp
#include <iostream>

#include <boost/io/ios_state.hpp>

void PrintHex(std::ostream& os, char byte) {
  // Try commenting out the next line
  boost::io::ios_flags_saver ifs(os);

  os << byte << " = " << std::hex << static_cast<unsigned>(byte) << '\n';
}

int main() {
  PrintHex(std::cout, 'A');
  std::cout << 123 << '\n';
  PrintHex(std::cerr, 'b');
  std::cout << 456 << '\n';
  PrintHex(std::cerr, 'C');
  std::cout << 789 << '\n';
}
```

## 2. ostream_joiner

## 需求

打印数组时，不想最后一个元素后面跟着一个分隔符。~~因为这会让完美主义纠结症患者抓狂！~~

## 解决方案

C 语言奇葩版，连 if 都不需要：

```c
#include <stdio.h>

int main(void) {
  int a[6] = {1, 2, 3, 4, 5, 6}, i;
  for (i = 0; i < 6; i++) {
    printf(",%d" + !i, a[i]);
  }
  return 0;
}
```

上面的方案纯属炫技，还是用 ostream_joiner 来搞定，一样看不到 if：

```cpp
#include <iostream>
#include <array>

#include <boost/io/ostream_joiner.hpp>

int main() {
  std::array<int,6> a{1, 2, 3, 4, 5, 6};
  std::copy(a.begin(), a.end(), boost::io::make_ostream_joiner(std::cout, ','));
}
```

## 3. quoted

## 需求

大部分语言的字符串都是需要转义的，除非用原始字符串（raw string），有时候想打印出转移后的字符串。

## 解决方案

使用 C++14 的 std::quoted，或者 boost::io::quoted，默认参数就是 C/C++ 的转义风格。

```cpp
#include <iostream>
#include <string>

#include <boost/io/quoted.hpp>

int main() {
  std::string buffer;
  std::getline(std::cin, buffer);
  std::cout << boost::io::quoted(buffer, '\\', '"') << '\n';
}

/*
* Input: C:\Program Files
* Output: "C:\\Program Files"
* 
* Input: {"name":"UMU618","male":true}
* Output: "{\"name\":\"UMU618\",\"male\":true}"
*/
```