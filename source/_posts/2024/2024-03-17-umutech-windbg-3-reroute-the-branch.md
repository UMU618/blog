---
layout: post
title: 无银第八哥【3】修改分支走向
date: 2024-03-17 17:50:39
description: Jump over it, or nop it!
categories: UMUTech
tags:
- debug
- reverse-engineering
- windows
---
## 需求

从人类的角度看，无非有两种情况：要某功能和不要某功能。在代码层面上则是：

- 改变程序执行路径。

- 跳过部分代码。

## 分析

如果您不要某个功能，那就是找到这功能的调用指令，把它去掉。而想要某个功能，则有两种基本的做法：

1. 无视原条件，修改后续的条件跳转指令，强行跳到需要的功能代码；

2. 改变条件本身，达成此功能需要的条件。

下面用 C++ 代码解释，假设原始代码如下：

```cpp
bool flag = Condition();
if (flag) {
    Wanted();
}
```

如果正常情况下 Condition 返回 false，而我们想 Wanted 函数被调用，那么，方法 1 是将代码改为：

```cpp
bool flag = Condition();
Wanted();
```

而方法 2 则是改为：

```cpp
bool flag = true;
if (flag) {
    Wanted();
}
```

当然，还有两种方法一起用，那就是改为：

```cpp
Wanted();
```

## 实现

具体实现主要使用 `jmp` 和 `nop`，更复杂的场景可能需要配合其它指令，请自行领悟。

### 1. jz/jnz 改为 jmp

最简单的一种情况是：跳转地址为 1 字节的相对位移，这种直接把跳转指令改为 `jmp` 即可，例如：

```
X: jz Y
```

需要改为：`jmp Y`，只需：`eb X eb` 即可。

如果跳转地址为 4 字节的相对位移，则 `jz`/`jnz` 有 2 字节，而 `jmp` 只有 1 字节，所以需要先 nop 掉第一字节：`eb X 90 e9`。这种方法无需调整后面的地址，另外还可以不用 `nop`，但需要调整地址：

```
X: jmp Y+1
```

因为指令变短了 1 字节，相对位移就更远 1 字节，这种改法纯属蛋疼，严重不建议采用。

### 2. nop 掉 call 指令

这种方法的基本操作极其简单，只需要 eb 若干个 90。

```
X: call Y
```

只要先查看 X 地址的这条指令有几个字节，再 eb X 后面跟几个 90 即可。

但有些情况也比较复杂，比如一个 x86 的 __stdcall，单独把 call 指令 nop 掉，会导致栈失衡（短期看不出任何异常），所以需要把前面的 push 也 nop 掉。__cdecl 和 __fastcall 不用，所以需要先判断一个 call 是不是 __stdcall。em，还是 x64 省事。

## 最简单的例子

正常情况下，按下 `winver` 界面上的“确定”按钮可以退出，我们来把它无效掉！

![winver](/images/2023/20230915-winver.png)

1. 首先，推测按下“确定”按钮后，是调用 EndDialog 来退出，所以对它下断点：`bp USER32!EndDialog`。

2. 发现 USER32!EndDialog 确实能断，`gu` 使它返回到调用处，发现是 SHELL32!AboutDlgProc+0xea。

```
0:000> ub SHELL32!AboutDlgProc+0xea
SHELL32!AboutDlgProc+0xca:
00007fff`6d218f3a 895f18          mov     dword ptr [rdi+18h],ebx
00007fff`6d218f3d 488bce          mov     rcx,rsi
00007fff`6d218f40 89473c          mov     dword ptr [rdi+3Ch],eax
00007fff`6d218f43 e84c6e0000      call    SHELL32!_ApplyLayout (00007fff`6d21fd94)
00007fff`6d218f48 e9d2000000      jmp     SHELL32!AboutDlgProc+0x1af (00007fff`6d21901f)
00007fff`6d218f4d 488bd5          mov     rdx,rbp
00007fff`6d218f50 488bce          mov     rcx,rsi
00007fff`6d218f53 48ff15166f3700  call    qword ptr [SHELL32!_imp_EndDialog (00007fff`6d58fe70)]
```

地址 00007fff`6d218f53 处的 call 指令有 7 字节，所以它本身的地址是 SHELL32!AboutDlgProc+0xea-7，即 SHELL32!AboutDlgProc+0xe3。

3. 重新运行，开始 nop：`eb SHELL32!AboutDlgProc+0xe3 90 90 90 90 90 90 90`，此时再继续运行后，即可得到一个无法按“确定”关闭的对话框。

注意：这样改后，同时标题栏的“关闭”按钮也无法关闭了，所以 `winver` 已经无法在界面上退出，需要用任务管理器关闭！
