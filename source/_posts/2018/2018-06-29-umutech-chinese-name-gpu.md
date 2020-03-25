---
layout: post
title: 显卡名称包含汉字导致 DX11 程序无法正常工作
date: 2018-06-29 11:04:11
categories: UMUTech
tags:
- dev
- windows
- debug
---
某游戏在 RemoteFX 远程桌面下无法正常运行。提示：

> 运行引擎需要DX11特征等级10.0

英文版提示：

> DX11 feature level 10.0 is required to run the engine.

稣立刻调用 dxdiag 查看，结果 Feature Level 10.0 是支持的！

然后决定自己写个 DX11 程序测试一下，于是找到这里例子：[Tutorial 3: Initializing DirectX 11](http://www.rastertek.com/dx11s2tut03.html)，稍加修改后运行，得到一个错误提示：

> MessageBox(hwnd, L"Could not initialize Direct3D.", L"Error", MB_OK);

接下来，仔细检查这个初始化过程，发现居然是因为 wcstombs_s 失败引起的：

```
// Convert the name of the video card to a character array and store it.
error = wcstombs_s(&stringLength, m_videoCardDescription, 128, adapterDesc.Description, 128);
```

原来是因为 RemoteFX 显卡的名字里有汉字……

> RemoteFX 3D 视频适配器

设备名称：

> Microsoft RemoteFX 图形设备 - WDDM

通过注册表改显卡名字，测试代码的问题解决！但 wcstombs_s 这块代码其实并无与显卡功能相关，去掉这段代码也可以解决问题。