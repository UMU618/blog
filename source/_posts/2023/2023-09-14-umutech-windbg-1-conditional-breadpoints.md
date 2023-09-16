---
layout: post
title: 无银第八哥【1】条件断点
date: 2023-09-14 23:30:28
description: Conditional breakpoints in WinDbg
categories: UMUTech
tags:
- debug
- reverse-engineering
- windows
---
## 问题

使用“无银第八哥”给注册表 API 下断点，结果调用极其频繁，如果一个个人工去看，容易逐渐失去耐心。~~毕竟，挨踢太卷了！~~

## 解决

您只需要“条件断点”！但是怎么写“条件”成为拦路虎。

好在，“无银第八哥”自带了一份很简要的学习材料，您可以在帮助菜单打开，或按 F1，或在命令窗口输入 `.hh` 打开，然后输入“conditional breakpoints”，将进入一篇名为《Conditional breakpoints in WinDbg and other Windows debuggers》的帮助文档。

“条件断点”建议的使用方式是，把条件写到文件里，方便复用。指定一个断点的条件为某个文件内容的语法是：

```
bp function "$$<C:\\commands.txt"
```

当然，文件的内容才是重点，将在后面的实践例子里讲解。

## 实践

### 需求

`winver` 显示的 Windows 的 Relase 版本信息，比如“22H2”，是从注册表里读的，想断下这个读取。

![winver](/images/20230915-winver.png)

### 实现

以下在 Windows 11 x64 下进行。

1. 下断点前，需要先知道这个 Release 信息保存的注册表位置：

- 主键：HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion
- 值：DisplayVersion
- 数据：即是需要获取的 Release 信息

2. 但是 `winver.exe` 的导入表里并没有任何 Reg API，反而这个程序的核心就是调用 SHELL32!ShellAboutW 而已。

3. 查看 `SHELL32.dll` 的导入表，发现有 `api-ms-win-core-registry-l1-1-0.dll`，但 Reg API 里有两个可以读值，需要做个基本排序，按兼容性推测，使用 RegQueryValueExW 几率更高。

```
// 老 API
LSTATUS RegQueryValueExW(
  [in]                HKEY    hKey,
  [in, optional]      LPCWSTR lpValueName,
                      LPDWORD lpReserved,
  [out, optional]     LPDWORD lpType,
  [out, optional]     LPBYTE  lpData,
  [in, out, optional] LPDWORD lpcbData
);

// 新 API
LSTATUS RegGetValueW(
  [in]                HKEY    hkey,
  [in, optional]      LPCWSTR lpSubKey,
  [in, optional]      LPCWSTR lpValue,
  [in, optional]      DWORD   dwFlags,
  [out, optional]     LPDWORD pdwType,
  [out, optional]     PVOID   pvData,
  [in, out, optional] LPDWORD pcbData
);
```

4. 打开“无银第八哥”，按 Ctrl+E，打开 `C:\Windows\System32\winver.exe`。

5. 第一个需要尝试的断点是 RegQueryValueExW，输入 `bm KERNELBASE!RegQueryValueExW`，然后 g，发现可以断下。

6. 开始考虑“条件断点”，需要针对 lpValueName 判断是否为 "DisplayVersion"。lpValueName 为第二个参数，按照 x64 call，即为 rdx，所以编写 `C:\devel\windbg\RegQueryValueExW.txt` 代码如下：

```
.if (@rdx != 0) { as /mu ${/v:ValueName} @rdx } .else { ad /q ${/v:ValueName} }
.if ($scmp(@"${ValueName}", "DisplayVersion") == 0)  { .echo ValueName } .else { .echo ValueName; gc }
```

然后，输入 `bm KERNELBASE!RegQueryValueExW "$$<C:\\devel\\windbg\\RegQueryValueExW.txt"`，再 g，发现能断下：

```
DisplayVersion
KERNELBASE!RegQueryValueExW:
00007ffc`2bd79ff0 48895c2410      mov     qword ptr [rsp+10h],rbx ss:0000006c`ef69e768=00007ffc2e6731dc
```

7. 额外地，可以返回（gu）看看（ub）调用方长啥样的：

```
0:000> gu
shcore!_SHRegQueryValueW+0x8d:
00007ffc`2e53e6bd 0f1f440000      nop     dword ptr [rax+rax]
0:000> ub
shcore!_SHRegQueryValueW+0x6b:
00007ffc`2e53e69b 44897db0        mov     dword ptr [rbp-50h],r15d
00007ffc`2e53e69f 48894c2428      mov     qword ptr [rsp+28h],rcx
00007ffc`2e53e6a4 4c8d4db8        lea     r9,[rbp-48h]
00007ffc`2e53e6a8 498bcd          mov     rcx,r13
00007ffc`2e53e6ab 48897c2420      mov     qword ptr [rsp+20h],rdi
00007ffc`2e53e6b0 4533c0          xor     r8d,r8d
00007ffc`2e53e6b3 488bd0          mov     rdx,rax
00007ffc`2e53e6b6 48ff15d3750a00  call    qword ptr [shcore!_imp_RegQueryValueExW (00007ffc`2e5e5c90)]
```
