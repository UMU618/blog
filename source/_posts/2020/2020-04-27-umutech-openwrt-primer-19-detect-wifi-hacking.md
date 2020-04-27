---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇19）：检测 WiFi 入侵
date: 2020-04-27 23:49:22
categories: UMUTech
tags:
- embedded
- linux
- openwrt
- security
---
## 问题

我怀疑有人在用工具穷举我的 WiFi 密码，我该怎么确认？

## 解决

运行 `iw event`，如果看到频繁出现 `new station` 和 `del station` 的 log，说明有设备在频繁连接和断开。

如果您的路由器是小米路由器 Pro，则可以用 `iwevent` 代替 `iw event`，密码不对的 log 是 `had deauthenticated`，断开是 `had disassociated`。

## 安全建议

设置密码时，应该检查一下您的密码是否在“字典”里。字典参考：

- <https://www.kaggle.com/wjburns/common-password-list-rockyoutxt>

- <https://github.com/danielmiessler/SecLists/blob/master/Passwords/Leaked-Databases/rockyou.txt.tar.gz>

> rockyou.txt contains 14,341,564 unique passwords, used in 32,603,388 accounts.

举个例子吧！稣打算用 10 个 0 做密码，先查一下……嗯哼！

> valentine
> idontknow
> pikachu
> little
> diamond1
> iloveu1
> babyphat
> peanut1
> kittens
> goddess
> ballet
> damien
> nascar
> 171717
> rangers1
> winston
> **0000000000**
> rocky1
> coolgirl
> maymay
> charlene
> caramelo
> selena
> lucero
> wendy
> volcom
> 1435254
> copper
> cindy
> baby123

地球真危险！稣回月球了……
