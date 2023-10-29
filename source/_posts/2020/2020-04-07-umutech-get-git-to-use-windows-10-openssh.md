---
layout: post
title: 让 git 使用 Windows 10 OpenSSH
date: 2020-04-07 23:02:18
categories: UMUTech
tags:
- debug
- ops
- windows
---
## 问题

在 Windows 10 安装 [git](https://git-scm.com/) 的同时，开启系统自带的 OpenSSH，则系统里存在两套 ssh，git 会默认使用它自己的那套。

## 分析

由于 Windows 10 的 sshd、ssh-agent 做成服务，比较容易管理，而且微软改造的版本会更注重安全，所以 UMU 决定舍弃 git 带的那套。

## 解决

思路：把 git 那套 ssh 指向 Windows 10 OpenSSH。

用管理员权限运行 `cmd`，输入：

```cmd
cd "%ProgramFiles%\git\usr\bin"
ren ssh.exe -ssh.exe
mklink ssh.exe %windir%\System32\OpenSSH\ssh.exe

ren scp.exe -scp.exe
mklink scp.exe %windir%\System32\OpenSSH\scp.exe
```

![git ssh 链接到 OpenSSH](/images/2020/20200407-mklink.png)
