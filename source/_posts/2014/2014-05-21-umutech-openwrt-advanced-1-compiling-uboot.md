---
layout: post
title: 跟 UMU 一起玩 OpenWRT（高级篇1）：编译不死 U-Boot
date: 2014-05-21 16:14:31
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
UMU 2010 年初就玩 OpenWRT/DD-WRT 了，蛋似编译东西还是初学者，本文纯属蛋疼的过程，欢迎批评教育，谢谢……

首先到 <https://github.com/pepe2k/u-boot_mod> 看明白作者的说明。这里简单说一下原理：固件（firmware）刷坏，但 U-Boot 没坏，这是半砖，可以用 TTL 线连路由器，通过 U-Boot 的功能刷好 firmware。如果两者都坏了，叫全砖，只能把 Flash 拆下来，用编程器刷好 U-Boot 和 firmware。不死 U-Boot 就是修改了 U-Boot 的实现，使我们可以用 RJ-45 网线来救砖，省去拆机搭 TTL 线的麻烦。

本质上说，这东西并非真的不死，只要 U-Boot 被刷坏，还是会死，不过几率不大，因为 OpenWRT 官方发行的 ROM 全都是保护 U-Boot 区域的，根据 UMU 的经验，只有三个情况会不小心或故意刷坏：

1. 从原厂固件刷不良固件；
2. 在 DD-WRT 下搞破坏（DD-WRT 没有保护 U-Boot）；
3. 自制固件去掉 U-Boot 写保护后搞破坏……

如果您真的这么蛋疼，还是准备编程器吧，只要是软件问题，在编程器面前没有砖的概念。（JTAG 也是救砖神器，但不是每台路由器都有，比如 DIR-505 就没有！）

由于 UMU 是 Windows 程序员，平时没有安装 Linux 桌面的习惯，蛋似由于做快游项目，买了不少服务器，都是 CentOS，所以第一步就是在 CentOS 上尝试编译，后来，您们懂的，爆出各种 213 码！服务器系统还是不适合开发！

不得已就在 Hyper-V Server 2012 上安装了 Ubuntu 12.04.1-desktop-i386，本来是想安装 x64 版本的，但又怕这些嵌入式的东西对 x64 可能支持不够好，算了，不要装 13 了。

接下来是选择编译环境了，按照 UMU 对 OpenWRT 的好感，明显是选择 [OpenWrt Toolchain for AR71xx MIPS](http://downloads.openwrt.org/attitude_adjustment/12.09/ar71xx/generic/OpenWrt-Toolchain-ar71xx-for-mips_r2-gcc-4.6-linaro_uClibc-0.9.33.2.tar.bz2)，然后开始编译，哗哗哗，编译好了……最后编译出来的 bin 却是 64KB+110B，尼玛，这 size 超标了，刷进去不是不死，是立刻死！

试验 [Sourcery CodeBench Lite Edition for MIPS GNU/Linux](https://sourcery.mentor.com/GNUToolchain/subscription3130?lite=MIPS) 可行。推测作者其实并没有用 OpenWrt Toolchain 编译过，而是用 Sourcery CodeBench，所以……为了节省时间，还是用后者吧！吐槽一下，这是商业软件，虽然有免费的 Lite 版本……

make dlink_dir505 一下，顺利编译出来，UMU 还小修改了一下 Web 界面，加入了自己的特色，不过要提醒一下，不要加太多，会爆……只有 64KB 的空间！
