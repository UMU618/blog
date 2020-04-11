---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇18）：更换 opkg 源
date: 2020-04-02 22:20:19
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
## 问题

默认源在国内访问速度普遍比较慢。

PS: 本篇理论上应该几年前就写的……以前经常用台湾省的网络，没发现，现在补一篇。

## 解决

1. 更换清华大学源

```sh
sed -i 's/downloads\.openwrt\.org/mirrors\.tuna\.tsinghua\.edu\.cn\/openwrt/g' /etc/opkg/distfeeds.conf
opkg update
```

2. 使用 https

```sh
opkg install libustream-mbedtls
sed -i 's/http:/https:/g' /etc/opkg/distfeeds.conf
```

## 相关

如果您想把整个软件源下载到本地，可以参考：<https://github.com/UMU618/openwrt-opkg-cache>
