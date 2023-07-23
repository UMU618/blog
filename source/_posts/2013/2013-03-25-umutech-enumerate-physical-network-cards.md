---
layout: post
title: 枚举物理网卡
date: 2013-03-25 15:25:41
categories: UMUTech
tags:
- dev
- windows
- 调研
---
　　其实目的是获取靠谱的 MAC 地址，但这个任务真蛋疼！不信您看看搜索出来的乐射……

　　神马 GetAdaptersInfo、GetIfEntry、GetAdaptersAddresses、NetWkstaTransportEnum，还有读取注册表 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkCards`。这些都会枚举到虚拟网卡，给您举个例子“VirtualBox Host-Only Ethernet Adapter”，读取神马 NetCfgInstanceId、MediaSubType，都不靠谱，没有平台移植性！

　　用 Setup API 枚举 Interface，匹配 PCI 和 USB 类型是比较靠谱的。

　　蛋似，虚拟机的网卡也是虚拟的，为了方便在虚拟机测试，您要注意放开一些特例……很抱歉，领导说代码要保密，自己搜吧，关键字：SetupDiGetDeviceInterfaceDetail、OID_802_3_PERMANENT_ADDRESS。

　　给个蛋碎的例子：`\\.\pci#ven_10ec&dev_8168&subsys_050e1028&rev_06#4&224db6dd&0&00e5#{ad498944-762f-11d0-8dcb-00c04fc3358c}\{4cc0ea76-88b7-40e1-8b4b-6339f8dd49bf}` 可以简称为 `\\.\{4cc0ea76-88b7-40e1-8b4b-6339f8dd49bf}` 或者 `\\.\Global\{4cc0ea76-88b7-40e1-8b4b-6339f8dd49bf}`。
