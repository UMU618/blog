---
layout: post
title: 自动添加 IP 黑名单到防火墙
date: 2023-11-02 22:59:10
description: ChatGPT 说：网络太危险了！
categories: UMUTech
tags:
- ops
- powershell
- security
- windows
---
## 问题

前文《[防爆破远程桌面密码](/2023/10/29/umutech-prevent-remote-desktop-password-attacks/)》提到可以用 Powershell 实现自动加 IP 黑名单到防火墙，这个坑还是得填，毕竟爆破依然在持续……

## 分析

核心点：

- IP 黑名单会持续新增，过去已经加入防火墙的名单也需要保存。所以应该把新增 IP 和防火墙已有 IP 求并集。

- 如果名单没变，不应该覆盖防火墙规则。

## 代码

见 Github 仓库：

<https://github.com/UMU618/windows-scripts/blob/master/pwsh/add-ip-blacklist-to-firewall.ps1>
