---
layout: post
title: 设置 Debian 不自动进入桌面
date: 2022-04-16 11:56:06
categories: UMUTech
tags:
- debian
- linux
- ops
- optimization
---
## 需求

在 Debian 上装了 KDE Plasma 桌面，但使用率不高，毕竟主要是当服务器使用，所以不想每次启动都自动进入桌面，以节约内存。

## 解决

```sh
sudo systemctl set-default multi-user.target
```

重启后就是默认的控制台登陆。如果想直接以当前控制台登陆的身份进入桌面，运行 `startx` 即可。但这种方式桌面是跑在当前控制台上，不是第 7 个控制台（Ctrl+Alt+F7），如果想尽量保持和原来自动启动桌面的环境一样，应该用：

```sh
sudo systemctl isolate graphical.target

# 或者更直接地【不建议使用】
sudo systemctl start sddm.service
```

这将以服务身份进入桌面，后面还要再通过图形界面登陆一次。

如想恢复自动进入图形界面：

```sh
sudo systemctl set-default graphical.target
```

如想知道当前处于哪种方式，可使用：

```sh
systemctl get-default
```
