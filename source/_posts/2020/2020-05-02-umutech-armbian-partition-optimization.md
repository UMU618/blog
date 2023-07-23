---
layout: post
title: Armbian 分区优化
date: 2020-05-02 15:58:19
categories: UMUTech
tags:
- armbian
- debug
- embedded
- linux
- ops
- optimization
---
## 问题

Android 电视盒刷 Armbian，分区时，EMMC 前面一部分没被利用，为什么？以及怎么办？

## 原因

u-boot 是为 Android 设计的，分区是按照 Android 需求分的。比如：

```
Partition table get from SPL is :
        name                        offset              size              flag
===================================================================================
   0: bootloader                         0            400000                  0
   1: reserved                     2400000           4000000                  0
   2: cache                        6c00000          20000000                  2
   3: env                         27400000            800000                  0
   4: logo                        28400000           2000000                  1
   5: recovery                    2ac00000           2000000                  1
   6: misc                        2d400000           2000000                  1
   7: boot                        2fc00000           2000000                  1
   8: system                      32400000          40000000                  1
   9: data                        72c00000         15f400000                  4
```

其中 reserved 分区放着其它分区的名字、位移、大小等信息，如果被破坏 u-boot 将无法识别分区。

env 分区保存启动脚本，如果被破坏，可能导致系统无法启动。

## 解决

一般的 Armbian 安装脚本，都会跳过前面的分区，从偏移 700MB 处开始分区。

```sh
parted -s "${DEV_EMMC}" mklabel msdos
parted -s "${DEV_EMMC}" mkpart primary fat32 700M 828M
parted -s "${DEV_EMMC}" mkpart primary ext4 829M 100%
```

作为优化狂，UMU 显然无法接受这样的浪费！

- cache 分区有 0x20000000 Bytes，也就是 512MiB，拿来做 /boot 分区岂不美哉？

- logo 分区？不，UMU 不想看启动画面，直接覆盖掉吧！

- 分区之间还有空闲！也不能放过！

所以，reserved 分区之后到 env 分区之前的全部空间都拿来做 /boot 分区，env 分区之后全部做 / 分区：

```sh
parted -s "${DEV_EMMC}" mklabel msdos
parted -s "${DEV_EMMC}" mkpart primary fat32 100MiB 628MiB
parted -s "${DEV_EMMC}" mkpart primary ext4 636MiB 100%
```

## 测试

玩客云和斐讯 N1 测试通过。
