---
layout: post
title: 跟 UMU 一起玩 OpenWRT（高级篇3）：在线升级内核
date: 2022-03-27 17:20:08
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
## 需求

想升级 OpenWRT 路由器的内核，又不想每次都把 SD 卡拿下来刷，怎么办？

## 前提

- 本文将以 Nano Pi R4S 为例。

- 这里的“在线”指的是：不重新刷 ROM，也不用 `sysupgrade`，纯手动替换文件，然后 `reboot` 生效。

## 解决

### 1. 先查看当前版本

```sh
uname -a
Linux UMUR4S 5.10.107 #0 SMP PREEMPT Sat Mar 19 19:05:00 2022 aarch64 GNU/Linux
```

目前是 5.10.107，想升级到 5.10.108！

### 2. 下载新 ROM 到路由器

可以直接在 OpenWRT 里用 `wget` 下载到 /tmp 下，或者在其它机器下载好，用 `scp` 上传到路由器的 /tmp 下。

下载地址：<https://github.com/UMU618/openwrt-config/releases>

### 3. 解压新 ROM、挂载分区

```sh
cd /tmp/

# wget

gunzip -d openwrt-rockchip-armv8-friendlyarm_nanopi-r4s-ext4-sysupgrade.img.gz

parted openwrt-rockchip-armv8-friendlyarm_nanopi-r4s-ext4-sysupgrade.img
unit B
p
```

可以看到 img 里的两个分区：

```
Model:  (file)
Disk /tmp/openwrt-rockchip-armv8-friendlyarm_nanopi-r4s-ext4-sysupgrade.img: 176160768B
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start      End         Size        Type     File system  Flags
 1      33554432B  50331647B   16777216B   primary  ext2         boot
 2      67108864B  176160767B  109051904B  primary  ext2
```

把这两个分区别分挂载：

```sh
mkdir new_boot
mount -o loop,offset=33554432 openwrt-rockchip-armv8-friendlyarm_nanopi-r4s-ext4-sysupgrade.img new_boot

mkdir new_root
mount -o loop,offset=67108864 openwrt-rockchip-armv8-friendlyarm_nanopi-r4s-ext4-sysupgrade.img new_root
```

### 4. 挂载待更新的 boot 分区

```sh
mkdir boot
mount /dev/mmcblk1p1 boot
```

检查一下，时间是不一样的：

```
root@UMUR4S:/tmp# ll boot/
drwxr-xr-x    3 root     root          4096 Jan  1  1970 ./
drwxrwxrwt   23 root     root           660 Mar 27 18:17 ../
-rw-r--r--    1 root     root           352 Mar 21 01:17 boot.scr
-rw-r--r--    1 root     root      14860296 Mar 21 01:17 kernel.img
drwx------    2 root     root          4096 Jan  1  1970 lost+found/
-rw-r--r--    1 root     root         55738 Mar 20 19:26 rockchip.dtb
root@UMUR4S:/tmp# ll new_boot/
drwxr-xr-x    3 root     root          4096 Jan  1  1970 ./
drwxrwxrwt   23 root     root           660 Mar 27 18:17 ../
-rw-r--r--    1 root     root           352 Mar 27 17:56 boot.scr
-rw-r--r--    1 root     root      14860296 Mar 27 17:56 kernel.img
drwx------    2 root     root          4096 Jan  1  1970 lost+found/
-rw-r--r--    1 root     root         55738 Mar 27 17:45 rockchip.dtb
```

开始复制文件：

```sh
cp new_boot/* boot/
cp -r new_root/lib/modules/5.10.108 /lib/modules/
```

检查一下，现在应该同时存在两个版本的 modules：

```sh
root@UMUR4S:/tmp# ll /lib/modules/
drwxr-xr-x    4 root     root          4096 Mar 27 18:28 ./
drwxr-xr-x   10 root     root          4096 Mar 21 05:24 ../
drwxr-xr-x    2 root     root          4096 Mar 20 03:05 5.10.107/
drwxr-xr-x    2 root     root          4096 Mar 27 18:28 5.10.108/
```

### 5. 重启验证

`reboot` 后查看版本，如果没问题就清理旧版本：

```sh
uname
Linux UMUR4S 5.10.108 #0 SMP PREEMPT Sun Mar 27 04:00:50 2022 aarch64 GNU/Linux

rm -rf /lib/modules/5.10.107
```
