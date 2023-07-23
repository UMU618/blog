---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇20）：WOL
date: 2020-05-04 23:43:01
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
## 需求

家里有个 PC，关机状态，想在公司远程开机，可是家里没人，怎么办？

## 条件

- 机器支持并开启 WOL (Wake On LAN)。

- 机器通过板载网卡连接路由器（USB 有线网卡不行）。

## 解决

1. 路由器需要有公网地址，如果没有，请参考《[跟 UMU 一起玩 OpenWRT（入门篇10）：穿透内网](/2014/07/27/umutech-openwrt-primer-10-through-the-intranet/)》做中转，总之需要能 SSH 到路由器上。

2. 路由器上安装 etherwake 或 wakeonlan。两者差别是：wakeonlan 是个 Perl 脚本，使用 UDP 包，不需要 root 权限。

- 如果是在 openwrt 直接使用 root 账号，建议用 etherwake。

```sh
opkg update
opkg install etherwake
etherwake MAC_ADDRESS_OF_PC
```

- 如果是 armbian，建议平时使用非 root，所以推荐 wakeonlan。

```sh
sudo apt update
sudo apt install wakeonlan
wakeonlan MAC_ADDRESS_OF_PC
```

## 测试

组装 PC 两台、Intel NUC 7i7BNH 一台测试用过。
