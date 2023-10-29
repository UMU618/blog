---
layout: post
title: 无银第八哥【2】修改 API 返回结果
date: 2023-09-16 16:26:47
description: Accessing Memory by Virtual Address with WinDbg
categories: UMUTech
tags:
- debug
- reverse-engineering
- windows
---
## 需求

想修改 API 的返回结果。举个例子，想把前文提到的 Windows Release 信息改为 "0618"。

![winver](/images/2023/20230916-winver.png)

## 分析

从技术角度看，需求就是在 RegQueryValueExW 返回时，改 lpData 指向的内存。

```
LSTATUS RegQueryValueExW(
  [in]                HKEY    hKey,
  [in, optional]      LPCWSTR lpValueName,
                      LPDWORD lpReserved,
  [out, optional]     LPDWORD lpType,
  [out, optional]     LPBYTE  lpData,
  [in, out, optional] LPDWORD lpcbData
);
```

lpData 为第 5 个参数，根据《[x64 软件约定](https://learn.microsoft.com/zh-cn/cpp/build/x64-software-conventions?view=msvc-170)》，它位于栈。具体来说，断点时，rip 处于 call 指令执行完毕的时间点，这时调用方的返回地址已经被压入栈里，所以此时的栈顶为返回地址。按照内存地址递增方向，返回地址后面就是 6 个参数。

## 实现

1. 按照前文方法，断到 DisplayVersion 的读取：

```
DisplayVersion
KERNELBASE!RegQueryValueExW:
00007ffc`2bd79ff0 48895c2410      mov     qword ptr [rsp+10h],rbx ss:00000073`b6cfe618=00007ffc2e6731dc
```

2. 用 `dq rsp L7` 显示返回地址和 6 个参数：

```
0:000> dq rsp L7
000000d7`e975e7e8  00007ffc`2e53e6bd 00007ffc`2d600000
000000d7`e975e7f8  00007ffc`2e6731dc 00000268`e4a4a6a0
000000d7`e975e808  00000000`0000003c 00000268`e4a96d10
000000d7`e975e818  000000d7`e975e820
```

其中，``00000268`e4a96d10`` 对应 lpData，``000000d7`e975e820`` 对于 lpcbData。

3. 可选地，查看 lpcbData 指向的值，发现它是 0x100：

```
0:000> dd 000000d7`e975e820 L1
000000d7`e975e820  00000100
```

4. 输入 gu 运行到函数返回，再查看 lpData 的内容：

```
0:000> gu
shcore!_SHRegQueryValueW+0x8d:
00007ffc`2e53e6bd 0f1f440000      nop     dword ptr [rax+rax]
0:000> du 00000268`e4a96d10
00000268`e4a96d10  "22H2"
```

5. 用 eu 指令修改 lpData 的内容：

```
eu 00000268`e4a96d10 "0618"
```

6. 输入 g，再继续运行，即可大功告成。

## 埋前文的坑

1. 前文“实现”节的第 5 步，`bm KERNELBASE!RegQueryValueExW` 断不下来？

您可能是在 Windows 10 下实践才遇到这个问题。可以用 `x KERNELBASE!RegQueryValueExW*` 看看是不是有多个。一般来说，即使有多个，无银第八哥也能都断，用 bl 可以看到多个都被加入。万一多个函数的类型不一样，可能就只加了一个 void 类型的，可以直接用地址指定另外类型的（没有被添加的）。

Windows 11 是这样：
```
0:000> x KERNELBASE!RegQueryValueExW*
00007ffc`2bd79ff0 KERNELBASE!RegQueryValueExW (void)
00007ffc`2beb2090 KERNELBASE!RegQueryValueExW (void)
0:000> bl
     1 e Disable Clear  00007ffc`2bd79ff0     0001 (0001)  0:**** KERNELBASE!RegQueryValueExW "$$<C:\\devel\\windbg\\RegQueryValueExW.txt"
     2 e Disable Clear  00007ffc`2beb2090     0001 (0001)  0:**** KERNELBASE!RegQueryValueExW "$$<C:\\devel\\windbg\\RegQueryValueExW.txt"
```

Windows 10 可能是这样：

```
0:000> x kernelbase!RegQueryValueExW*
00007ff8`2a400a20 KERNELBASE!RegQueryValueExW (void)
00007ff8`2a303700 KERNELBASE!RegQueryValueExW (RegQueryValueExW)
```

2. `KERNELBASE!RegQueryValueExW` 返回后的指令怪怪的？

```
0:000> gu
shcore!_SHRegQueryValueW+0x8d:
00007ffc`2e53e6bd 0f1f440000      nop     dword ptr [rax+rax]
```

前一条的 `call    qword ptr [shcore!_imp_RegQueryValueExW (00007ffc`2e5e5c90)]` 指令是 7 字节的，如果为了对齐也不应该补 5 字节呀。

```
0:000> u 00007ffc`2e53e6b6
shcore!_SHRegQueryValueW+0x86:
00007ffc`2e53e6b6 48ff15d3750a00  call    qword ptr [shcore!_imp_RegQueryValueExW (00007ffc`2e5e5c90)]
00007ffc`2e53e6bd 0f1f440000      nop     dword ptr [rax+rax]
00007ffc`2e53e6c2 8bd8            mov     ebx,eax
00007ffc`2e53e6c4 85c0            test    eax,eax
00007ffc`2e53e6c6 755a            jne     shcore!_SHRegQueryValueW+0xf2 (00007ffc`2e53e722)
00007ffc`2e53e6c8 8b55b8          mov     edx,dword ptr [rbp-48h]
00007ffc`2e53e6cb 83fa01          cmp     edx,1
00007ffc`2e53e6ce 0f85b7000000    jne     shcore!_SHRegQueryValueW+0x15b (00007ffc`2e53e78b)
```

看！后面的指令是从 ``00007ffc`2e53e6c2`` 开始的，也没有对齐。所以，如果不是为了对齐，那就可能是为了方便调试时在函数返回时加 `int 3` 了。如果有新的答案，将在本系列后续文章分享。
