---
layout: post
title: Linux 桌面玩稣
date: 2022-04-25 00:00:20
categories: UMUTech
tags:
- linux
- ops
---
## 问题

- 那么多 Linux 发行版，哪个桌面最好用？

- i3wm 到底是不是反人类？

- FreeOffice 究竟是不是免费？

- 以极客理念做的产品，究竟是不是坑人的？

- 国产 OS 到底有没有戏？

## 体验

| 系列 | 包管理器 | 防火墙 | 经验 | 主观感到的坑 |
| :- | :- | :- | :- | :- |
| OpenWRT | `opkg` | 很容易理解的文件配置：`vi /etc/config/firewall` 或者 `uci`，然后 `/etc/init.d/firewall reload` | 从 2010 年初开始一直在用，能刷它的路由器没有一台能逃过。轻量是它的特点。 | `ash` 不够智能，当然如果硬件允许，可以装 `zsh`；不适合做桌面，但其实也可以哦（肯定是坑）！|
| Ubuntu/Debian、Armbian | `apt` | 好用的：`ufw` | 大学就见好多学长用 Ubuntu，默认情况下，它的桌面比 Debian 漂亮，Debian 比较适合当服务器，实际上稣一般使用 Debian。物理机装了好多台，一些电视盒子也刷成 Armbian 在做测试机。 | 目前 Bullseye 用的内核是 5.10，比较保守。默认桌面都很丑。 |
| CentOS（后续 RockyLinux、AlmaLinux） | `yum` | 有点忘记了，是不是有个 `firewall-cmd`？ | 很久以前被迫用过…… | 就是没好感，反正也挂了（才怪）。 |
| Manjaro/ArchLinux | `pacman`、`yay` | 好用的：`ufw` | Manjaro 桌面体验很好，ArchLinux 只在虚拟机里体验。内核 5.15 是比 Debian 新。 | i3 版装完中文支持有问题，KDE 则没问题。 |
| PostMarketOS/AlpineLinux | `apk`（真的不是安卓啊～） | 不知道怎么喷的：`awall` | 也是内核 5.15，和 OpenWRT 的理念很像，而且注重轻量和安全。AlpineLinux 适合做容器的基础镜像。 | 对中文真不友好。进入系统后还是把 `ash` 换成 `zsh` 吧。还有这个 `awall`……和 `ufw` 比，真的很想说 ashit！ |

列这个表其实想说几个感受：

1. 体验这么多不同发行版真浪费生命。尤其想说：包管理器有必要这么多吗？对软件开发商来说，真的很无语呀！

2. 在用 Manjaro 时发现，网易云音乐这个软件，其实是来自 Debian 系的 deb 包，国产的 OS 大部分也都是基于 Debian 的。也就是说：如果有流行的软件，会有人重新打包成其它包。怎么说呢……国产 OS 如果开发了啥国民软件，是不是 Manjaro 也会吸收过去？那国产 OS 岂不是又没啥优势了？

3. 如果习惯 Windows 或者 macOS，最好还是选择 KDE Plasma，然后装合适的主题，让它更像 Windows 或者 macOS。i3wm 这种极客型的产品，不适合大部分人。而且 Manjaro i3 是个社区版，不是官方版，中文支持是有问题的。

4. 有小伙伴问稣：怎么会用 [PostMarketOS](https://postmarketos.org/) 这种乱七八糟的东西？稣内牛满面，还不是因为很早以前买了台 [Surface RT](https://openrt.gitbook.io/open-surfacert/)……自从微软抛弃它之后，稣挣扎过一次，装了 Windows 10 ARM，可现在不是已经 11 了吗？于是一不做二不休装 Linux，然后就装了这个奇怪的 [PostMarketOS](https://openrt.gitbook.io/open-surfacert/surface-rt/linux/root-filesystem/distros/postmarketos)。当然后悔了，它能刷 [Ubuntu Server](https://wiki.ubuntu.com/ARM/SurfaceRT) 的，真是脸疼……

5. Wayland 吗？不了，谢谢，稣用 X11 就行。

6. 最后一个问题：稣叛变到 Arch 系了吗？没有！选 Linux，稣还是用 Debian，毕竟要支持国产嘛（间接）！
