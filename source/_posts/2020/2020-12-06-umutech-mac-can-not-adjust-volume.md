---
layout: post
title: Mac 外接显示器时无法调节音量？
date: 2020-12-06 23:35:03
categories: UMUTech
tags:
- debug
- macos
---
## 问题

用一根 Type-C 连笔记本和显示器，然后显示器插了一对音响。

1. 当笔记本是 Windows 笔记本时，可以调节外部音响的音量。
2. 当笔记本是 MBP13 M1 时，无法调节外部音响的音量。
3. 对比另一台 MBP 接 UltraFine 显示器时，却又可以调……怀疑是贵的才可以，便宜的阉割了【开玩笑的】。

![M1 + AOC 便宜的显示器不行](/images/20201206-mbp-m1.jpg)

![M1 + AOC 便宜的显示器无法调音量](/images/20201206-aoc.jpg)

![Intel + UltraFine 可以](/images/20201206-mbp-intel.png)

![Intel + 贵的 UltraFine 可以调音量](/images/20201206-ultrafine.png)

## 原理

这是因为 HDMI、DisplayPort 和 Thunderbolt 等接口传输的都是带有固定音量的数字音频信号，而能调节音量的音频信号属于模拟信号，因此只有外接显示器将数字信号转换为模拟信号后才能调节音量。UltraFine 可以调节，推测是因为它和 macOS 之间有魔法协议，直接调节硬件的音量，就好比直接去旋转音响上的音量旋钮。

## 解决

参考：[少数派：Mac 外接显示器时无法用键盘调节音量？这个方法能够帮助到你 | 一日一技](https://zhuanlan.zhihu.com/p/50912888)

然而……M1 是 Arm64e 的 CPU，而此软件的核心是个内核扩展模块，Release 出来的只有 x64 的 Mach-O 程序。

![Soundflower 安装失败](/images/20201206-install-failed.jpg)

赫赫，尴尬地笑出声！好在 [Soundflower](https://github.com/mattingalls/Soundflower) 是开源的，要自己编译个对应架构的版本……然后：

> 在搭载 Apple 芯片的 Mac 上，您可能首先需要使用“启动安全性实用工具”将安全策略设置为“降低安全性”，并选择“允许用户管理来自被认可开发者的内核扩展”复选框。

这么麻烦，果断放弃。再找找，发现这个可以直接用：<https://github.com/MonitorControl/MonitorControl>
