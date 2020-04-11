---
layout: post
title: 解决 dropbear 和 openssh 的冲突
date: 2016-05-26 10:12:44
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
## 需求

在 OpenWRT 路由器 C 用公钥验证方式登录另一台 OpenWRT 路由器 S。

## 问题

一开始配置完很顺利登录，后来进行一些操作后，居然登录不了，提示输入密码……

类似情况：<https://github.com/rssnsj/openwrt-hc5x61/issues/99>

## 分析

一开始也以为是 OpenWRT 版本的问题，从 dd trunk 降到 cc，无用。再降到 bb，发现没问题了，因为 bb 没有 sshtunnel，最后把怀疑对象锁定到 sshtunnel。

sshtunnel 是基于 openssh 的，在装 sshtunnel 时，openssh 会作为依赖项被装上，然后替换了系统自带的 dropbear 客户端，所以后来使用的 ssh 是 openssh，但私钥文件却是一开始用 dropbearkey 产生的。两者格式并不兼容。

## 解决方案

装上 openssh-keygen，然后用 ssh-keygen 产生新的私钥，再用 ssh-keygen -y -f ~/.ssh/id_rsa 打印公钥。
