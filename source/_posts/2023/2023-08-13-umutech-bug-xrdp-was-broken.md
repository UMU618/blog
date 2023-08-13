---
layout: post
title: 八哥系列：Debian 12 的 xrdp 突然连不上！
date: 2023-08-13 16:54:01
description: 稣在教 ChatGPT 用 Debian 12
categories: UMUTech
tags:
- debian
- debug
- linux
- macos
- ops
---
八哥就是让你料不到！

## 故事

用 macOS 的 Microsoft Remote Desktop Beta 连着 Debian 12，切回 macOS 一段时间后，再回 Debian，发现卡死了。

一看 Debian 的机器，是睡眠了，立刻按电源键唤醒。但是 Microsoft Remote Desktop Beta 再也连不上 Debian。一直在连接的界面，也不报错，也不超时。

SSH 到 Debian 一顿治疗后，依然无法连接。最后重启 Microsoft Remote Desktop Beta，居然好了……

## 折腾记录

### 八哥一：为什么睡眠了？

因为每次 RDP 进入 KDE 后，都弹出一个密码输入框，上面想着“挂起系统需要身份验证”，于是稣做了如下操作，把它去掉。

`sudo vim /etc/polkit-1/rules.d/85-suspend.rules`，输入：

```js
polkit.addRule(function(action, subject) {
    if (action.id == "org.freedesktop.login1.suspend" &&
        subject.isInGroup("tsusers")) {
        return polkit.Result.YES;
    }
});
```

```sh
sudo chmod 755 /etc/polkit-1/rules.d
sudo chmod 644 /etc/polkit-1/rules.d/85-suspend.rules
```

结果——机器就能自动睡眠了。这是不符合预期的……

### 八哥二：SSH 好好的，唯独 XRDP 坏了？

确实是稣的一大失算！因为期间在 Debian 本地使用了 wayland，而 xrdp 用的是 xorg，怀疑这可能把 xrdp 弄坏，就把目光都集中在 xrdp 上，各种修复，甚至重新安装、配置 xrdp，也无济于事。

另外一个误导因素是：刚用上 京东京造的 SSD，期间摸了摸，觉得烫得不行，担心是 SSD 异常，导致 xrdp 的文件被破坏。事实上，这个破硬盘也确实因为过热自动写保护两次，最后都无法启动系统了。

### 八哥三：Windows 连 Debian 居然没问题……

这才意识到是 macOS 的 Microsoft Remote Desktop Beta 有问题！连忙重启这个 App，终于真相大白！现在看着这名字末尾的 Beta 陷入沉思。
