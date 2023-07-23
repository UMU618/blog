---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇6）：挂接 U 盘
date: 2014-06-23 00:49:39
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
## 需求

DIR-505 有一个 USB2.0 接口，给它带来了很大的扩展性，我们可以插 U 盘、移动硬盘等，来做文件共享，或者离线下载等。接下来就先搞定 U 盘，主要参考资料：<http://wiki.openwrt.org/doc/howto/usb.storage>

## 安装

### 1. 安装 USB 和存储器的内核模块

```sh
opkg update
opkg install kmod-usb-storage
opkg install kmod-scsi-generic
```

### 2. 安装文件系统内核模块

```sh
opkg install kmod-fs-ext4
```

### 3. 安装应用工具

USB 辅助工具、分区、格式化工具，这些非必要，看情况安装：

```sh
opkg install usbutils fdisk e2fsprogs
```

## 调试

### 1. 测试能否识别 U 盘

不插 U 盘时，输入 `lsusb`，显示如下

```
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

插入 U 盘后，再 `lsusb`，发现多了一条，这说明工作正常：

```
Bus 001 Device 002: ID 0603:0908 Novatek Microelectronics Corp.
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

### 2. 分区

如果 U 盘需要重新分区，请用 `fdisk`，这里不具体介绍，也可以在 Windows 上用 `diskpart` 完成，如果您还在路由器上操作，可以参考这个：<http://www.leiphone.com/diy-a-smart-router-topic-increase-memory-3.html>，第一步: 对设备进行分区。

分区完后，查看一下 /dev 目录里有没有出现硬盘符和分区符：

```sh
ls /dev | grep sd
```

上面装了 ext4 文件系统的驱动，因为这个对 OpenWRT 最合适，如果原来不是这个格式，那么安装了 e2fsprogs 后就可以用下面命令格式化了：

```sh
mkfs.ext4 /dev/sda1
```

### 3. 挂载

接下来用 mount 挂接，头尾行是必须，中间的三行是一种防止未挂接好就被写入的机制：

```sh
mkdir /mnt/usb
touch /mnt/usb/USB_DISK_NOT_PRESENT
chmod 555 /mnt/usb
chmod 444 /mnt/usb/USB_DISK_NOT_PRESENT
mount /dev/sda1 /mnt/usb
```

到这里就可以通过 /mnt/usb 来访问 U 盘了，不过工作才完成一半而已……下面还要让 U 盘能开机自动加载，主要参考：<http://wiki.openwrt.org/doc/techref/block_mount> 和 <http://wiki.openwrt.org/doc/uci/fstab>。先安装程序：

```sh
opkg install block-mount blkid
```

如果您比较珍惜存储空间，可以不用安装 blkid，block-mount 就行。用 `blkid` 或 `block info`，查看分区的 UUID。比如用 `blkid`，则 UMU 的 U 盘是显示：

```
/dev/mtdblock7: TYPE="squashfs"
/dev/sda1: UUID="16e381bc-d9bb-40fd-9e98-410b346931ef" TYPE="ext4"
```

接下来输入 `block detect` 查看一下：

```
config 'global'
    option anon_swap '0'
    option anon_mount '0'
    option auto_swap '1'
    option auto_mount '1'
    option delay_root '5'
    option check_fs '0'
config 'mount'
    option target '/mnt/sda1'
    option uuid '16e381bc-d9bb-40fd-9e98-410b346931ef'
    option enabled '0'
```

生成配置文件，并把上面的 target 改为您想要的，enabled 值改为 1：

```sh
block detect > /etc/config/fstab
uci set fstab.@mount[-1].target='/mnt/usb'
uci set fstab.@mount[-1].enabled=1
uci commit fstab
```

## 参考

本文介绍的都是精简过的必须操作，其它可选项请自行参考：<http://wiki.openwrt.org/doc/uci/fstab>。
