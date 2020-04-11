---
layout: post
title: newifi mini 刷 OpenWRT
date: 2015-12-10 16:33:42
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
newifi mini，即 lenovo Y1，属于不开放 SSH 的类型，越用越不爽，还是刷了吧。

先到 <http://www.xcloud.cc/download.shtml> 下载“路由助手”，然后到 OpenWRT 官网下载 ROM，推荐用 trunk 上的（目前是 dd），因为 bb 和 cc 都没有集成 kmod-mt76（当然你自己手动安装是可以的，opkg install kmod-mt76），下载链接：<http://downloads.openwrt.org/snapshots/trunk/ramips/mt7620/openwrt-ramips-mt7620-y1-squashfs-sysupgrade.bin>。目前刷完是 OpenWrt Designated Driver r47548，5G WiFi 没问题。

由于是 trunk 版，luci 可能要自己安装，请参考文章《[跟 UMU 一起玩 OpenWRT（入门篇7）：安装 LUCI](/2014/06/26/umutech-openwrt-primer-7-install-luci/)》。

存在几个问题：

1. 刷完，三个网口顺序和原版是颠倒的。

2. 5G WiFi 设置参数后似乎没有办法立刻生效，UMU 都是 reboot 一下解决。
