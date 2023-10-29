---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇1）：硬件选型和刷机
date: 2014-05-24 01:14:14
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
看了《[跟hoowa学做智能路由](https://www.leiphone.com/news/201406/diy-a-smart-router.html)》系列，也想写点自己的经验。

UMU 用的第一款硬件是 D-Link DIR-505。首先声明，UMU 不是 D-Link 员工，也不卖 DIR-505，用它完全是当下对比几个可选项筛选后的结果。理由：

1. 本身就是不死 Bootloader，刷坏了固件用网线就可以救，把电脑 IP 设为 192.168.0.100，按住 Reset 开机，Web 浏览器访问 192.168.0.1。前面写的《[不拆机给 D-Link DIR-505 刷上不死 U-Boot](/2014/05/23/umutech-openwrt-advanced-2-mtd-write-uboot-on-openwrt/)》完全是蛋疼地研究过程，对一般用户 UMU 建议不要刷，因为没有 JTAG，刷固件本来就不死，不小心刷坏 Bootloader，就只能拆机上编程器了，这明显作死。

2. 配置比较高（相比 TP-Link 坑爹级同价位产品），8MB Flash，64MB RAM，UMU 手头上还有三个 TP 的（TL-WR841N、743N、941N）都只有它一半。有 USB 2.0 接口，743N 的 USB 是 1.1 的。

3. 国内电商有得买，而且价格便宜，UMU 买的时候是 78 块。

4. 小巧，方便携带，随时开撸！

也说一下它的缺点：没有外接天线，所以您懂的，信号必然比较弱，不适合“穿墙”族……然后 RJ45 口只有一个，有时候会不太方便。它最适合的使用场景是研究 OpenWRT、短距离和出差便捷使用。

**请自行根据当下情况选择，毕竟新的硬件总是越来越强大，还越来越便宜。**

接下来就刷个 OpenWRT 先~目前没有稳定发行版支持 DIR-505，所以要在 trunk 下找，下载目录是：<http://downloads.openwrt.org/snapshots/trunk/ar71xx/>。如果直接开刷，很可能失败，因为 D-Link 是有锁区的，OpenWRT.org 编译的 ROM 不是为中国版准备的，所以要动一下手脚先。上 WinHex 改 ROM，下面两张图分别是中国版和国际版：

![中国版](/images/2014/20140524-dir-505-cn.jpg)

![国际版](/images/2014/20140524-dir-505-def.jpg)

两者只是图片指出的位置不同而已，可以自己改，如果把 OpenWRT 的 ROM 改为 CN 也无法在原厂 ROM 下刷成功的话，可以先找个官方的 DEF ROM 改为 CN，刷一下，再刷 OpenWRT 原版的 DEF ROM。
