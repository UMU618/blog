---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇12）：SSH 端口转发（代理上 QQ）
date: 2017-03-01 15:57:03
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
## 需求

在之前的文章《[跟 UMU 一起玩 OpenWRT（入门篇10）：穿透内网](/2014/07/27/umutech-openwrt-primer-10-through-the-intranet/)》，介绍了从家里连到公司内网，现在需求反过来了，想在公司代理到家里，让公司的 QQ 使用家里的网络出口。

## 解决

还是那些熟悉的工具！首先，家里的路由器要刷好 OpenWRT，绑定一个动态域名，记为 HomeRouter。

> 2020/04/05 23:59 添加：
>
> 绑定动态域名的方法可以参考：<https://github.com/UMU618/openwrt-ipv6-addresses>

在 Windows 下用 putty 连到 HomeRouter，基本就大功告成！开 tunnels 方法如图：

![PuTTY tunnels](/images/20170301-putty.png)

或者用：

```cmd
PLINK.EXE -N -D 1080 root@HomeRouter
```

最后是 QQ 的设置：

![QQ Proxy](/images/20170301-qq.png)
