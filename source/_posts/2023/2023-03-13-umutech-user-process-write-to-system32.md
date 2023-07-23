---
layout: post
title: 用户进程狂写日志到 system32 目录？
date: 2023-03-13 22:51:02
description: 稣是 Windows 用户，ChatGPT 让稣去搬砖！
categories: UMUTech
tags:
- debug
- dev
- windows
---
## 问题

开发了一个服务 S，它偶尔需要做一些用户才能做的事情，所以创建了一个用户进程 U。后来开发自测时，发现用户进程 U 总是往 C:\Windows\system32\config 里写日志。设计上，明明是让进程 U 往用户的 %AppData% 目录写日志的。

## 分析

服务 S 使用 [WTSQueryUserToken](https://learn.microsoft.com/zh-cn/windows/win32/api/wtsapi32/nf-wtsapi32-wtsqueryusertoken) 获取用户的 Token，然后使用 [CreateProcessAsUser][cpau] 创建用户进程 U。

这一步是得到验证的，使用任务管理器查看进程 U 确实是当前登陆的用户身份，而不是 Session 0 里的 SYSTEM 身份。

但是进程 U 去获取 %AppData% 时却依然拿到 SYSTEM 身份的目录。这点可以用 [Process Explorer](https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer) 验证，进程属性里有一页“环境”。这说明 [CreateProcessAsUser][cpau] 时，继承了服务 S 的环境。

```cpp
BOOL CreateProcessAsUserW(
  [in, optional]      HANDLE                hToken,
  [in, optional]      LPCWSTR               lpApplicationName,
  [in, out, optional] LPWSTR                lpCommandLine,
  [in, optional]      LPSECURITY_ATTRIBUTES lpProcessAttributes,
  [in, optional]      LPSECURITY_ATTRIBUTES lpThreadAttributes,
  [in]                BOOL                  bInheritHandles,
  [in]                DWORD                 dwCreationFlags,
  [in, optional]      LPVOID                lpEnvironment,
  [in, optional]      LPCWSTR               lpCurrentDirectory,
  [in]                LPSTARTUPINFOW        lpStartupInfo,
  [out]               LPPROCESS_INFORMATION lpProcessInformation
);
```

经过排查，确实 lpEnvironment 是传了 nullptr。

## 解决

先用 [CreateEnvironmentBlock](https://learn.microsoft.com/zh-CN/windows/win32/api/userenv/nf-userenv-createenvironmentblock) 创建用户身份的环境块，然后传给 [CreateProcessAsUser][cpau]。

```cpp
CreateEnvironmentBlock(&environment, user_token, FALSE);
```

其中，第三个参数传 FALSE 是关键，表示不继承服务 S 的环境。

[cpau]: https://learn.microsoft.com/zh-cn/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessasuserw