---
layout: post
title: Debian Testing
date: 2023-02-12 23:22:48
description: 稣是 Debian Testing 用户，ChatGPT 让稣去搬砖！
categories: UMUTech
tags:
- debian
- debug
- linux
- ops
---
## 故事

稣玩 Debian 10 几年了！大约 2011-2012 年开始，先在树莓派、电视盒子上玩，后来在 PC、服务器上。目前，所有家庭服务器、《[智能时钟](/2022/05/15/umutech-smart-clock/)》都用 Debian Bullseye，而桌面则用 Debian Testing。

其它发行版也尝试过，详情可见《[Linux 桌面玩稣](/2022/04/25/umutech-linux-desktop/)》。曾经有一段时间，《[用华为擎云 L420 体验国产操作系统（UOS 和银河麒麟）](/2022/05/21/umutech-huawei-qingyun-l420-uos-kylinos/)》，打算从此只用 UOS，结果统信发了一个文章，说以后要脱离 Debian……

十几年间，有不少道友试图说服稣改投其它派系的发行版，比如：

- Arch Linux：它很极客，而且它是滚动更新模型，可以很快使用新内核。

- Manjaro：它是最容易安装的 Arch 系，国内用户多。

- openSUSE：它对新手友好，而且很稳，也有滚动更新的“风滚草”，咳，风滚草耶！

对此，稣只有一个坚决的答复：不！稣只要 Debian！因为……它不只名字像大便，连 Logo 都像大便！这么一本正经地搞笑，显然要~~大便~~大力支持！

![Debian Logo](https://www.debian.org/Pics/debian-logo-1024x576.png)

还有啊，你们要的“滚动更新”，不就是 Debian 的 Testing 版本吗？

## ChatGPT 乱入

稣是 Debian Testing 用户，擅长修复声卡外接 HDMI 设备无声问题，请问稣适合做什么职业？
> ChatGPT：电子城低级技术支持和运维。

这个职业容易被 ChatGPT 取代吗？
> ChatGPT：容易

那有啥职业不容易被 ChatGPT 取代？
> ChatGPT：搬砖

## 折腾记

### 1. 某 J4125 迷你电脑

用途：下载机、文件共享服务器，兼职电视盒子

由于主要是台微型服务器，系统选择 Debian Bullseye，桌面环境还是反人类的 sddm + i3。

服务器软件的安装可以参考《[智能时钟](/2022/05/15/umutech-smart-clock/)》。至于电视盒子嘛……其实它是一个浏览器：

```sh
sudo apt install chromium
```

但是有个问题——用 HDMI 线连电视后，发现没有声音！

```sh
sudo apt install firmware-linux-nonfree firmware-iwlfifi firmware-sof-signed
sudo apt install pulseaudio
cat /etc/modprobe.d/alsa.conf
echo 'options snd-intel-dspcfg dsp_driver=1' | sudo tee /etc/modprobe.d/alsa.conf
```

### 2. 某 N5095A 迷你电脑

用途：娱乐桌面

这次选择 GNOME，依然是轻松搞定，但是有个问题——用 HDMI 线连电视后，发现没有声音！

装完 non-free 的声卡驱动后还是一样。

但是有第二个问题——没有无线网卡的驱动！

是 Realtek RTL8821CE，那就自己编译一个：<https://github.com/UMU618/rtw88>

又但是，编译后也无法加载呀，它是个没签名的内核模块，稣又不想关闭 Secure Boot！只好装 MOK (Machine Owner Key) 搞定签名。

转念一想，会不会 Testing 版本已经支持这款硬件了？果断切过去。

目前 Testing 的 GNOME 已经使用 PipeWire 取代 PulseAudio，所以不能再安装 PulseAudio，因为这会卸载 GNOME！坑稣呢……只要加个声卡的调教参数，即可！

```sh
echo 'options snd-intel-dspcfg dsp_driver=1' | sudo tee -a /etc/modprobe.d/alsa-legacy.conf > /dev/null
```

6.X 的内核和 GNOME 桌面下，娱乐的那点事，一切完美。