---
layout: post
title: 开发 LSP 遇到的蛋疼问题
date: 2012-12-11 18:16:00
categories: UMUTech
tags:
- dev
- windows
---
今天发生一个莫名其妙的问题，导致浪费一个早上的时间排查问题。

测试 LSP 期间，已经反注册它，netsh winsock reset 加重启好几次……依然有程序加载它，用 Process Explorer 查了一下是：IpOverUsbSvc.exe 和 daemonu.exe，后来把 LSP 文件删掉，再重启，这时候当然无法加载了，可是 UMU 又想重现一下这个莫名其妙的问题，结果一个早上没了，无法重现……

15:38 2012/12/25 补充

今天这个现象又出现了。再手动重启这两个服务后，不再加载 LSP 了。它们对应的服务名是：IpOverUsbSvc 和 nvUpdatusService。这说明这两个服务很可能每次重启机器时都没有正常关闭，系统提供了某种机制让他们在下一次重启后快速恢复了运行现场（保留了有 LSP 注册时的环境）。
