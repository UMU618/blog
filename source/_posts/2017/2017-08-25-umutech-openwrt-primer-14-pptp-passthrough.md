---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇14）：PPTP 穿透
date: 2017-08-15 00:49:39
description: 
categories: UMUTech
tags:
- embedded
- openwrt
---
刚刷完 OpenWRT trunk 版本，默认不支持 PPTP passthrough，表现为此路由器内网的 PC 拨号时，认证很快成功，但迟迟不能完成，最终报错误码 619。这是因为默认不支持 GRE 协议的 NAT。

官方就有解决方案，简单地说是运行一下两条：

```shell
opkg update
opkg install kmod-nf-nathelper-extra
```

立刻生效。
