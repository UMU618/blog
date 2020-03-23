---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇7）：安装 LUCI
date: 2014-06-26 20:30:15
description: 
categories: UMUTech
tags:
- embedded
- openwrt
---
UMU 不推荐安装 LUCI，还是多打命令好，可以学习更多东西，而且 LUCI 比较浪费存储空间！

```shell
opkg update
opkg install luci-ssl
```

推荐使用 SSL 版本，比较安全，但比较大，如果装不下可以试试不带 SSL 的：

```shell
opkg install luci
```

您可能不习惯默认的主题（luci-theme-bootstrap），Flash 够大的话，还是装个常用的：

```shell
opkg install luci-theme-openwrt
```

开启服务：

```shell
/etc/init.d/uhttpd start
```

设置开机自动运行（不推荐）：

```shell
/etc/init.d/uhttpd enable
```
