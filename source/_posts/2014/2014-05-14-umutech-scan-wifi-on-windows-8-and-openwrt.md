---
layout: post
title: Windows 8 和 OpenWRT 下查看 WiFi 属性
date: 2014-05-14 18:39:33
categories: UMUTech
tags:
- embedded
- linux
- openwrt
- windows
---
从 Win7 到 Win8，部分 WiFi 属性被隐藏掉了，在图形界面上看不到……只好用命令行了：

```cmd
netsh wlan show networks mode=bssid
```

OpenWRT 上是：

```sh
iw dev wlan0 scan
```
