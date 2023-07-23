---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇8）：网络摄像机
date: 2014-07-14 23:31:30
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
## 需求

买了一个微软 LifeCam HD-3000，已经过了视频聊天的年纪，插到 DIR-505 玩一下吧。

## 安装

安装过程出奇简单：

```sh
opkg update
opkg install kmod-video-uvc
```

如果安装失败，看看是不是固件应该升级了，请参考《[跟 UMU 一起玩 OpenWRT（入门篇5）：升级固件](/2014/06/20/umutech-openwrt-primer-5-sysupgrade)》。

插上摄像头，检查一下 /dev/video0 是否存在，如果不存在，`reboot` 一下，如果存在，接下来安装和运行应用：

```sh
opkg install mjpg_streamer
# 使用 MJPG 格式，推荐：
mjpg_streamer -i "input_uvc.so -d /dev/video0" -o "output_http.so -p 8080 -w /www/webcam"
# 使用 YUV 格式，多了一个 -y 参数，比较慢，不推荐： 
mjpg_streamer -i "input_uvc.so -y -d /dev/video0" -o "output_http.so -p 8080 -w /www/webcam"
```

其它可选输入参数，例如：

> -r 320x240 设置分辨率为320x240
>
> -f 10 设置刷新率

设置用户密码，加输出参数：

> -c user:password

## 调试

打开浏览器，输入 <Mhttp://192.168.1.1:8080/>，如果没问题，参考网页上的说明操作即可。

由于 DIR-505 性能一般，效果可能不理想，建议在更高配置的路由器上尝试。
