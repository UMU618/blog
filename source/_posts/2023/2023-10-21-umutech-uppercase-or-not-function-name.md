---
layout: post
title: 函数名首字母用大写还是小写？
date: 2023-10-21 15:14:07
description: 大写还是小写？这是个问题！
categories: UMUTech
tags:
- dev
- coding-style
- cpp
---
## 问题

不同编码规范对函数名的命名格式有不同要求，但主流有以下几类：

- PascalCase：比如 [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html#Function_Names)

- camelCase：比如 [LLVM Coding Standards](https://llvm.org/docs/CodingStandards.html#name-types-functions-variables-and-enumerators-properly)

- snake_case：比如 [PPP Styple Guide](https://www.stroustrup.com/Programming/PPP-style.pdf)、[K&R Style](https://www.kernel.org/doc/html/v4.15/translations/zh_CN/coding-style.html#id5)

哪种适合 Windows 开发呢？

## 分析

### 1. PascalCase 的坑

PascalCase 偶尔会遇到和 Win32 API 宏冲突的情况：

```cpp
#include <Windows.h>

#include <iostream>

namespace umutech {

int CreateWindow() noexcept {
  MessageBox(nullptr, L"For disassembling", L"CreateWindow", MB_OK);
  return 0;
}

}  // namespace umutech

int main() {
  umutech::CreateWindow();
}
```

以上代码无法编译，因为 SDK 头文件里有这样的定义：

```cpp
#define CreateWindowA(lpClassName, lpWindowName, dwStyle, x, y,\
nWidth, nHeight, hWndParent, hMenu, hInstance, lpParam)\
CreateWindowExA(0L, lpClassName, lpWindowName, dwStyle, x, y,\
nWidth, nHeight, hWndParent, hMenu, hInstance, lpParam)
#define CreateWindowW(lpClassName, lpWindowName, dwStyle, x, y,\
nWidth, nHeight, hWndParent, hMenu, hInstance, lpParam)\
CreateWindowExW(0L, lpClassName, lpWindowName, dwStyle, x, y,\
nWidth, nHeight, hWndParent, hMenu, hInstance, lpParam)
#ifdef UNICODE
#define CreateWindow  CreateWindowW
#else
#define CreateWindow  CreateWindowA
#endif // !UNICODE
```

导致以下编译错误：

```
1>function_name.cpp(7,5): warning C4003: not enough arguments for function-like macro invocation 'CreateWindowW'
1>function_name.cpp(7,5): error C2059: syntax error: ','
1>function_name.cpp(7,29): error C2143: syntax error: missing ';' before '{'
1>function_name.cpp(7,29): error C2447: '{': missing function header (old-style formal list?)
1>function_name.cpp(15,12): warning C4003: not enough arguments for function-like macro invocation 'CreateWindowW'
1>function_name.cpp(15,12): error C2059: syntax error: ','
```

改成下面这样，才能编译：

```cpp
#include <Windows.h>

#include <iostream>

namespace umutech {

int CreateWindowEx() noexcept {
  MessageBox(nullptr, L"For disassembling", L"CreateWindowEx", MB_OK);
  return 0;
}

}  // namespace umutech

int main() {
  umutech::CreateWindowEx();
}
```

以上代码虽然编译通过，但实际上 CreateWindowEx 还是个宏。用 IDA 逆向编译后的 exe，并加载 pdb 后，可以看到：

```asm
; int __cdecl main(int argc, const char **argv, const char **envp)
main proc near
sub     rsp, 28h

; int __fastcall umutech::CreateWindowExW()
umutech__CreateWindowExW: ; uType
xor     r9d, r9d
lea     r8, Caption     ; "CreateWindowEx"
lea     rdx, Text       ; "For disassembling"
xor     ecx, ecx        ; hWnd
call    cs:__imp_MessageBoxW
xor     eax, eax
add     rsp, 28h
retn
main endp
```

~~虽然没啥危害，但 C++ 20 程序员不喜欢宏！~~

### 2. camelCase 是不是更好？

如果函数名使用 camelCase，则没有机会与 Win32 API 的宏定义冲突。

另一个好处是在做 API Hooking 时，命名可以更短。比如 Hook ShowWindow，那么替代函数可以就叫 showWindow，而用 PascalCase，则可能需要叫 MyShowWindow。

那么是不是把 Google C++ Style 的函数名由 PascalCase 改为 camelCase 就完美了？

更好，并不是完美……camelCase 也有个小问题——只有一个单词时，无法区分是 camelCase，还是 snake_case。比如 size，是函数（camelCase），还是临时变量（snake_case）？

## 总结

UMU 建议，如果已经在使用 Google C++ Style，应该避免函数名与 Win32 API 一样。如果正在从头制定一套 Coding Style，则可以考虑函数用 camelCase。