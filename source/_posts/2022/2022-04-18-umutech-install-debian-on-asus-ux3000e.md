---
layout: post
title: 在华硕灵耀 X 纵横上装 Debian 桌面的经验
date: 2022-04-18 00:46:52
categories: UMUTech
tags:
- debian
- linux
- ops
---
## 需求

今天发现华硕灵耀 X 纵横已经降价到 6999 元。所以，应该给它装上 Debian 了。

## 准备

- [Debian ISO](https://www.debian.org/distrib/index.zh-cn.html)：稣选择的是 [netinst](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-11.3.0-amd64-netinst.iso)。

- [Rufus](https://rufus.ie/zh/)

## 解决

1. 需要准备一个 U 盘，用 Rufus 做一个 UEFI 格式的启动盘。

2. 如果有 USB 有线网卡，插上去，联网安装更省事，如果没有那就断网安装一个最小系统，然后去借一个 USB 有线或者无线网卡……

3. 需要 non-free 驱动，先选择不安装，后面再装。

4. 选择中国的镜像服务器，南方选中科大，北方选择阿里云或者清华大学。

5. tasksel 阶段，不要安装桌面环境，先安装最小的系统。

6. 安装完，重启。

6. 登陆系统，编辑 `/etc/apt/sources.list`，加 non-free 和配置 debian-security：

```
deb https://mirrors.ustc.edu.cn/debian/ bullseye main contrib non-free
#deb-src http://mirrors.ustc.edu.cn/debian/ bullseye main contrib non-free

deb https://mirrors.ustc.edu.cn/debian-security bullseye-security main contrib non-free
#deb-src https://mirrors.ustc.edu.cn/debian-security bullseye-security main contrib non-free

deb https://mirrors.ustc.edu.cn/debian/ bullseye-updates main contrib non-free
#deb-src http://mirrors.ustc.edu.cn/debian/ bullseye-updates main contrib non-free
```

这步很重要，如果不改 debian-security，那么接下来在国内网络环境下，可能会很慢……

7. 安装 non-free 驱动，主要是无线网卡和声卡：

```
sudo apt install firmware-iwlwifi firmware-sof-signed
```

装完无线网卡驱动后，网卡名字是 wlo1，配置见 <https://wiki.debian.org/WiFi/HowToUse>。

8. 用 `tasksel` 安装桌面，稣选择 KDE Plasma。

9. 【可选】开远程桌面：

```
sudo apt install xrdp
```

![Plasma UX3000E](/images/2022/20220418-ux3000e.png)
