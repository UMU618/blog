---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇9）：远程监听
date: 2014-07-21 23:47:09
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
## 需求

在《[跟 UMU 一起玩 OpenWRT（入门篇8）：网络摄像机](/2014/07/14/umutech-openwrt-primer-8-webcam)》介绍的 mjpg_streamer 并不能传输声音，所以有了本文！

## 参考

<http://forum.anywlan.com/thread-282658-1-1.html>

## 安装

### 1. 示例硬件信息

硬件还是选用微软 LifeCam HD-3000，您也可以去淘个便宜的带麦克风的 USB 声卡。

### 2. 安装程序

```sh
opkg update
opkg install kmod-usb-audio
opkg install icecast
```

如果您打算使用 ogg 格式则安装 ices：

```sh
opkg install ices
```

用 mp3 格式则安装 darkice：

```sh
opkg install darkice
```

### 3. 配置

ices 的配置文件（ices-oss.xml）可以去官网（<http://www.icecast.org/ices.php>）下载整个压缩包，里面有。

## 经验

由于涉及声音编码，需要大量计算，经过实践，CPU 才 400MHz 的 DIR-505，无论是 ogg 还是 MP3 格式都卡成翔……

请用配置更好的路由器测试，比如如意云 RY-01 的 CPU 是 600MHz 的，勉强可行。
