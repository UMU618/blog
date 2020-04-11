---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇5）：升级固件
date: 2014-06-20 01:05:47
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
## 问题

OpenWRT 的主线于 2014-06-19 06:07:37 UTC 更新了固件，如果不跟随升级，安装内核模块时会失败，比如以下命令：

```sh
opkg update
opkg install kmod-hid
```

## 解决

升级固件：

```sh
cd /tmp
wget http://downloads.openwrt.org/snapshots/trunk/ar71xx/openwrt-ar71xx-generic-dir-505-a1-squashfs-sysupgrade.bin 
sysupgrade openwrt-ar71xx-generic-dir-505-a1-squashfs-sysupgrade.bin
```
