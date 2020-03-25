---
layout: post
title: XP 下 Edit 控件透明字体时的 Bug
date: 2012-12-25 10:06:29
categories: UMUTech
tags:
- dev
- windows
- cpp
---
## 前提

XP 系统，程序使用了 Manifest 指定使用 Microsoft.Windows.Common-Controls 
现象：Edit 控件处理 WM_CTLCOLOREDIT 改变颜色，问题出在 SetBkMode 设置透明后，控件删除字符时无法立刻刷新，即会残留。

## 解決

### 方法 1

自残，别用 Microsoft.Windows.Common-Controls，删除类似下列的代码：

```cpp
#if defined _M_IX86
    #pragma comment(linker, "/manifestdependency:\"type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='x86' publicKeyToken='6595b64144ccf1df' language='*'\"")
#elif defined _M_IA64
    #pragma comment(linker, "/manifestdependency:\"type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='ia64' publicKeyToken='6595b64144ccf1df' language='*'\"")
#elif defined _M_X64
    #pragma comment(linker, "/manifestdependency:\"type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='amd64' publicKeyToken='6595b64144ccf1df' language='*'\"")
#else
    #pragma comment(linker, "/manifestdependency:\"type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")
#endif
```

### 方法 2

放弃“透明”，采用“伪透明”——如果您的背景是纯色，用 SetBkColor 就行。

### 方法 3（推荐）

检测到 XP 时，WM_CTLCOLOREDIT 返回画刷前，自己用 FillRect/Rectangle 涂一下……或者发一个 WM_ERASEBKGND，当然这个做法的前提是 WM_ERASEBKGND 的处理就是自己涂一下，如果就一句 return TRUE 那是等于啥也没干。 

## 参考

同样悲剧的一个描述：<http://zhidao.baidu.com/question/9749770.html>
