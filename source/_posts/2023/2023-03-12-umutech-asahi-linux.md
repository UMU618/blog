---
layout: post
title: 在 MacBook Air M1 上体验 Asahi Linux
date: 2023-03-12 01:53:31
description: 稣是 macOS 用户，ChatGPT 让稣去搬砖！
categories: UMUTech
tags:
- linux
- macos
- ops
---
## 故事

macOS 在访问 samba 共享里的图片时，会留下 . 前缀文件，还找不到办法关闭。

```sh
defaults write com.apple.desktopservices DSDontWriteNetworkStores true
```

这只能解决 .DS_Store，无法解决对可写 samba 的破坏！要知道，稣的 samba 目录是 SD 卡，哪受得了这折磨？

所以，还是杀掉贵族 macOS，安装游侠 Linux 吧！

## 安装

按说明，一键安装，即可：

```sh
curl https://alx.sh | sh
```

第一次关机时，记得要等 15 秒，再长按电源键开机，直到看到文字提示，松开电源键。

## 国内源

它是 Arch Linux，所以：

```sh
$ vi /etc/pacman.d/mirrorlist
Server = https://mirrors.ustc.edu.cn/archlinuxarm/$arch/$repo
```

## 编译内核

内核代码下载：https://github.com/AsahiLinux/linux/tags

menuconfig 的配置文件：https://github.com/AsahiLinux/PKGBUILDs/tree/main/linux-asahi

运行 make 时，提示缺啥就装啥，没难度。

```sh
make menuconfig
make -j 8
sudo make modules_install
sudo make install
sudo mkinitcpio -g /boot/initramfs-6.2.6-asahi.img -k 6.2.6-asahi
sudo update-grub
sudo vim grub/grub.cfg
sudo reboot
```

## 卸载

Asahi Linux 的问题还很多，不能当桌面系统日常使用的原因是：

- 没有声音！

- 触控板在 Linux 下表现明显不如 macOS，可能是 Apple 的驱动优化太强了，而且是有专利的。

- arm64 的桌面软件生态不行，大家只给 Apple 面子……

- 指纹解锁也废了。

所以，还是卸了吧！

不过稣的 macOS 被玩坏过，重装了，所以分区可能不太一样，暂且看看，原理无非是删除分区。

在 macOS 里：

```sh
$ diskutil list
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *251.0 GB   disk0
   1:             Apple_APFS_ISC Container disk1         524.3 MB   disk0s1
   2:                 Apple_APFS Container disk2         125.1 GB   disk0s2
   3:                        EFI EFI - ASAHI             500.2 MB   disk0s3
   4:           Linux Filesystem                         119.6 GB   disk0s4
   5:        Apple_APFS_Recovery Container disk3         5.4 GB     disk0s5

/dev/disk2 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +125.1 GB   disk2
                                 Physical Store disk0s2
   1:                APFS Volume Macintosh HD - Data     40.6 GB    disk2s1
   2:                APFS Volume Macintosh HD            8.9 GB     disk2s3
   3:              APFS Snapshot com.apple.os.update-... 8.9 GB     disk2s3s1
   4:                APFS Volume Preboot                 4.8 GB     disk2s4
   5:                APFS Volume Recovery                783.7 MB   disk2s5
   6:                APFS Volume VM                      1.1 GB     disk2s6

# /dev/disk0 的 3: 和 4: 是要删除的
$ diskutil eraseVolume free free /dev/disk0s3
$ diskutil eraseVolume free free /dev/disk0s4
# 如果 EFI - ASAHI 上面有一个 2.5 GB 的 disk0sX，可以用以下命令删除：
$ diskutil apfs deleteContainer disk0sX
# 之后再用图像界面的“磁盘工具”操作，确保空间还给 Macintosh HD 宗卷
```
