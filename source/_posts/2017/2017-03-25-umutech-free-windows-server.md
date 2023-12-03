---
layout: post
title: 完全免费的 Windows Server 系统，不需要序列号、不需要激活、更不需要破解
date: 2017-03-25 23:34:47
categories: UMUTech
tags:
- ops
- windows
- 调研
---
2009-04-17 22:06 在百度空间上发表过一次，后来百度空间倒闭了……最近给自己家里搭建家庭文件共享服务器用到，所以在这边再发一次。

2009 年时，由于项目需要，用过 Hyper-V Server 2008。到了 2012-09-25 升级为 Hyper-V Server 2012。这次（2017-03-22）用的是 Hyper-V Server 2016。这么多年一直还是完全免费的。

Hyper-V Server 是基于 Windows Server Server Core x64 的虚拟机服务器系统，要正常提供虚拟机服务， CPU 必须满足三个条件：x64、DEP (Data Execution Prevention)、HV (Hardware Virtualization)，但 UMU 不需要它的专业本领——虚拟机服务，所以只需要有 x64 CPU 就可以了。目前只使用他的副业，作为网上邻居（SMB）服务器和静态文件 HTTP Server，就家用而言，绝对够用，前者是系统自带的共享功能，用 `net share` 命令开启，后者安装 node.js + http-server 模块。

但它不是完整的 Windows Server，比如您想跑 IIS，那就不能使用它了。它最适合的情况是您开发了一些系统服务（NT Service）类的应用，比如游戏服务端、聊天软件服务端，想发布到 Windows Server 上。

# 2023-12-04 补充

1. Hyper-V Server 2019 是最后一个免费的 Hyper-V Server。

2. 主流支持到 2024-01-09，扩展支持到 2029-01-09。

参考：<https://learn.microsoft.com/en-us/lifecycle/products/hyperv-server-2019>
