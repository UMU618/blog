---
layout: post
title: Lenovo Y1S 千兆 LAN 改 WAN
date: 2016-04-28 00:34:07
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
## 需求

Lenovo Y1S 的原 WAN 口是百兆的，连接千兆网络时，WAN 口成为瓶颈，需把两个千兆 LAN 其中一个改为 WAN。

## 实现

UMU 一开始直接在官方 ROM 上去改 switch，却发现行不通，因为原 WAN 口被废掉后，如果不插着网线，官方 ROM 会很智能地以为路由器是没上网的，子网内终端浏览网页时，会一直被重定向到路由器设置页面。

尝试修改 `/etc/config/system` 还是没解决，所以……直接刷 OpenWRT 吧……三步走：

1. 参考《[newifi mini 刷 OpenWRT](/2015/12/10/umutech-install-openwrt-on-newifi-mini/)》。

2. 下载 ROM。开发版地址如下，稳定版请根据当前情况自寻链接。

    <https://downloads.openwrt.org/snapshots/trunk/ramips/mt7620/openwrt-ramips-mt7620-y1s-squashfs-sysupgrade.bin>

3. 刷完再改 switch，搞定。
