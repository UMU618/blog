---
layout: post
title: 红米 AX6S 刷 OpenWRT
date: 2022-04-11 01:46:07
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
## 问题

按照[官方文档](https://openwrt.org/toh/xiaomi/ax3200)操作，结果重启后还是进入小米原版系统。

## 解决

在出厂版本上刷开发版时，是刷到 firmware1 上，openwrt 被刷到 firmware，默认还是启动 firmware1，所以应该：

```sh
nvram set flag_last_success=0
nvram commit
reboot
```

搞定。
