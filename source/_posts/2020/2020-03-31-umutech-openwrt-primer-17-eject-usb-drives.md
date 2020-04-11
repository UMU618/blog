---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇17）：卸载 U 盘
date: 2020-03-31 00:05:47
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
## 需求

在 PC 插入 U 盘/移动硬盘，Windows 会发出令人愉悦的“灯等灯”声，然后 U 盘灯开始牛逼闪闪（如果有灯）；安全弹出时，又会发出“的的等”，灯熄灭（有些 U 盘不会灭灯，而是常亮着，不会再闪；移动硬盘一般都会灭灯）。

OpenWRT 这么强大，怎么能不支持？

- 什么？你说 `umount`？那 /dev/sda 还能重新挂载呢！~~那灯还亮着呢！~~（有些 U 盘弹出后灯常亮，设计好的才会灭灯。）

- 什么？你说直接拔掉？你赢了！但有时候，稣是在远程操作，要是没人配合拔掉，岂不是要插着耗电？穷人可是交不起电费的……

## 解决

```sh
opkg update
opkg install eject

eject /dev/sda
```

终于安全弹出啦！

**注意**：弹出后，​/dev/sda 还会存在，但无法再 mount，而且它下面的分区 /dev/sda1 ​等，都会消失。

eject 默认会先后尝试使用 CD-ROM 和 SCSI 命令弹出设备，可以用 `-v` 参数查看详细流程，一般 U 盘用 `-s` 参数指定使用 SCSI 命令更为直接。

## 相关

[跟 UMU 一起玩 OpenWRT（入门篇6）：挂接 U 盘](/2014/06/23/umutech-openwrt-primer-6-mount-usb-storage/)
