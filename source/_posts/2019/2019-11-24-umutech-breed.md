---
layout: post
title: 极路由和 newifi 刷 BREED
date: 2019-11-24 01:40:40
categories: UMUTech
tags:
- embedded
- router
---
## 极路由 1S

1S 有两款硬件：hc5661 和 hc5661a，刷错变砖，而且要刷的文件名有点迷惑性，意不意外？

- hc5661: <https://breed.hackpascal.net/breed-mt7620-hiwifi-hc5761.bin>

- hc5661a: <https://breed.hackpascal.net/breed-mt7628-hiwifi-hc5661a.bin>

原版 bootloader 有 DHCP 功能，自身 IP 是 192.168.2.1，比较特殊。

这里以 hc5661 为例，版本是：HC5661 - 1.4.11.21001s，破解式 root 或开发者解锁后，可以直接 `mtd unlock u-boot`，开不开心？

```sh
BusyBox v1.22.1 (2018-05-10 05:32:57 CST) built-in shell (ash)
Enter 'help' for a list of built-in commands.

***********************************************************
              __  __  _              _   ____  _   TM
             / / / / (_) _      __  (_) / __/ (_)
            / /_/ / / / | | /| / / / / / /_  / /
           / __  / / /  | |/ |/ / / / / __/ / /
          /_/ /_/ /_/   |__/|__/ /_/ /_/   /_/
                  http://www.hiwifi.com/      rooted by UMU
***********************************************************
root@HiUMU:~# cat /proc/mtd
dev:    size   erasesize  name
mtd0: 00030000 00010000 "u-boot"
mtd1: 00010000 00010000 "hw_panic"
mtd2: 00010000 00010000 "Factory"
mtd3: 00020000 00010000 "oem"
mtd4: 00010000 00010000 "bdinfo"
mtd5: 00010000 00010000 "backup"
mtd6: 00f70000 00010000 "firmware"
mtd7: 00120000 00010000 "kernel"
mtd8: 00e50000 00010000 "rootfs"
mtd9: 00400000 00010000 "rootfs_data"
root@HiUMU:~# mtd unlock u-boot
Unlocking u-boot ...
root@HiUMU:~# cd /tmp
root@HiUMU:/tmp# mtd write ./breed-mt7620-hiwifi-hc5761.bin u-boot
Unlocking u-boot ...

Writing from ./breed-mt7620-hiwifi-hc5761.bin to u-boot ...
[e:0]   [w0]
[e:1]   [w1]
```

刷完，捅菊花进入的信息是：

| CPU | MediaTek MT7620A ver 2, eco 3 |
| :- | :- |
| 内存 | 128MB DDR2 |
| Flash | Winbond W25Q128 @ 24MHz (16MB) |
| 以太网 | MediaTek MT7620A built-in 5-port 10/100M switch |
| 时钟频率 | CPU: 580MHz, Bus: 193MHz |
| 编译日期 | 2018-12-29 [git-135bed9] |
| 版本 | 1.1 (r1266) |

## 极路由 3

版本 HC5861 - 1.4.10.20837s，一样能开发者解锁后直接刷。这种不保护 bootloader 的 ROM，也是醉了，真香……已经过保，也不想恢复官方 ROM，直接刷 BREED，再刷 OpenWRT 18.06.5。

```sh
BusyBox v1.22.1 (2018-03-10 04:32:13 CST) built-in shell (ash)
Enter 'help' for a list of built-in commands.

***********************************************************
              __  __  _              _   ____  _   TM
             / / / / (_) _      __  (_) / __/ (_)
            / /_/ / / / | | /| / / / / / /_  / /
           / __  / / /  | |/ |/ / / / / __/ / /
          /_/ /_/ /_/   |__/|__/ /_/ /_/   /_/
                  http://www.hiwifi.com/
***********************************************************
root@Hiwifi:~# cat /proc/mtd
dev:    size   erasesize  name
mtd0: 00030000 00010000 "u-boot"
mtd1: 00010000 00010000 "hw_panic"
mtd2: 00010000 00010000 "Factory"
mtd3: 00020000 00010000 "oem"
mtd4: 00010000 00010000 "bdinfo"
mtd5: 00010000 00010000 "backup"
mtd6: 00f70000 00010000 "firmware"
mtd7: 00120000 00010000 "kernel"
mtd8: 00e50000 00010000 "rootfs"
mtd9: 00380000 00010000 "rootfs_data"
root@Hiwifi:~# mtd unlock u-boot
Unlocking u-boot ...
root@Hiwifi:~# cd /tmp
root@Hiwifi:/tmp# mtd write breed-mt7620-hiwifi-hc5861.bin u-boot
Unlocking u-boot ...

Writing from breed-mt7620-hiwifi-hc5861.bin to u-boot ...
[e:0]   [w0]
[e:1]   [w1]
```

BREED 信息：

| CPU | MediaTek MT7620A ver 2, eco 6 |
| :- | :- |
| 内存 | 128MB DDR2 |
| Flash | Winbond W25Q128 @ 24MHz (16MB) |
| 以太网 | MediaTek MT7620A built-in 5-port 10/100M switch |
| 时钟频率 | CPU: 580MHz, Bus: 193MHz |
| 编译日期 | 2018-12-29 [git-135bed9] |
| 版本 | 1.1 (r1266) |

嗯……这款路由器就是骗钱的。

## Lenovo Y1S

这款最简单，直接捅菊花进官方恢复模式，刷这个文件：<https://breed.hackpascal.net/breed-mt7620-lenovo-y1s.bin>！

## Newifi D1

先官方 ROM 降级：xCloudOS_newifi-d1_Build20150922_v0.0.4.3500_beta_sign.bin 或更早的 xCloudOS_newifi-d1_Build_v0.0.4.2100_beta_sign.bin，自寻下载。

较老的版本可能和 Y1S 一样可以直接在恢复模式下刷，不过 UMU 忘记这样尝试。

0.0.4.3500, r33798 的版本，布局如下：

```sh
## cat /proc/mtd
dev:    size   erasesize  name
mtd0: 00030000 00010000 "u-boot"
mtd1: 00010000 00010000 "u-boot-env"
mtd2: 00010000 00010000 "Factory"
mtd3: 02000000 00010000 "fullflash"
mtd4: 01fb0000 00010000 "firmware"
mtd5: 0012a926 00010000 "kernel"
mtd6: 01e656da 00010000 "rootfs"
mtd7: 01080000 00010000 "rootfs_data"
mtd8: 00020000 00010000 "panic_oops"
```

网上的刷机教程如下：

```sh
cd /tmp
wget https://breed.hackpascal.net/breed-mt7621-newifi-d1.bin --no-check-certificate
dd if=/dev/zero bs=1024 count=192 | tr "\000" "\377" >breed_192.bin
dd if=breed-mt7621-newifi-d1.bin of=breed_192.bin conv=notrunc
cat /tmp/breed_192.bin /dev/mtd1 /dev/mtd2 /dev/mtd4 >fullflash_with_breed.bin
mtd write fullflash_with_breed.bin fullflash
```

原理：u-boot 分区不能直接刷，但 fullflash 分区可以刷，fullflash 其实包含了 u-boot。

fullflash = u-boot + u-boot-env + firmware

以上命令就是组合成一个适合刷到 fullflash 的文件，而且是用 BREED 覆盖了 u-boot，然后刷到 fullflash，这样 u-boot 就被覆盖了，其它分区还是原来的内容。

值得注意的是：刷完 BREED，**捅菊花的位置变了**，不再是捅原来的菊花……而是天线下方那个红十字按钮，这原是用于 WPS 的。

## Newifi D2

访问 <http://192.168.99.1/newifi/ifiwen_hss.html> 开启 SSHD，查看分区布局：

```sh
BusyBox v1.24.1 (2018-02-27 16:23:44 CST) built-in shell (ash)


  _______________________________________________________________
 |    ____                 _                 ____               |
 |   |  _ \ __ _ _ __   __| | ___  _ __ __ _| __ )  _____  __   |
 |   | |_) / _` | '_ \ / _` |/ _ \| '__/ _` |  _ \ / _ \ \/ /   |
 |   |  __/ (_| | | | | (_| | (_) | | | (_| | |_) | (_) >  <    |
 |   |_|   \__,_|_| |_|\__,_|\___/|_|  \__,_|____/ \___/_/\_\   |
 |                                                              |
 |                  PandoraBox SDK Platform                     |
 |                  The Core of SmartRouter                     |
 |       Copyright 2013-2016 D-Team Technology Co.,Ltd.SZ       |
 |                http://www.pandorabox.org.cn                  |
 |______________________________________________________________|
  Base on OpenWrt BARRIER BREAKER (3.2.1.7418, 2018-03-06-git-bbccda9)
#cat /proc/mtd
dev:    size   erasesize  name
mtd0: 00010000 00010000 "u-boot-env"
mtd1: 00010000 00010000 "Factory"
mtd2: 01fb0000 00010000 "firmware"
mtd3: 00146bf9 00010000 "kernel"
mtd4: 01e49407 00010000 "rootfs"
mtd5: 00d40000 00010000 "rootfs_data"
mtd6: 00020000 00010000 "panic_oops"
mtd7: 00010000 00010000 "nvram"
```

这是被隐藏掉两个分区的！

高端刷法，是用一个内核模块来刷的，自寻 newifi-d2-jail-break.ko，参考文章：<https://www.right.com.cn/forum/thread-365936-1-1.html>

按以上链接刷好是 1.1 (r1237) 版，进 BREED 刷最新 BREED：<https://breed.hackpascal.net/breed-mt7621-newifi-d2.bin>

信息为：

| CPU | MediaTek MT7621A ver 1, eco 3 |
| :- | :- |
| 内存 | 512MB DDR3 |
| Flash | Winbond W25Q256 @ 48MHz (32MB) |
| 以太网 | MediaTek MT7530 Gigabit switch |
| 时钟频率 | CPU: 880MHz, DDR: 1066MHz, Bus: 293MHz, Ref: 40MHz |
| 编译日期 | 2018-12-29 [git-135bed9] |
| 版本 | 1.1 (r1266) |
