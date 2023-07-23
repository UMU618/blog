---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇3）：工作模式开关
date: 2014-06-05 19:00:48
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
DIR-505 有一个工作模式开关，可以设定三个模式，但刷了 OpenWRT 后，默认没有任何功能，我们可以利用 GPIO 读取获得开关的位置。

```sh
#!/bin/sh

read_gpio() {
    (echo $1 > /sys/class/gpio/export) >& /dev/null
    (echo "in" > /sys/class/gpio/gpio$1/direction) >& /dev/null
    return `cat /sys/class/gpio/gpio$1/value`;
}

read_gpio 19;
v=$?;
read_gpio 20;
v=$v$?;
read_gpio 21;
v=$v$?;
read_gpio 22;
v=$v$?;
read_gpio 23;
v=$v$?;
case "$v" in
    10001) v="ap";;
    11001) v="repeater";;
    01001) v="hotspot";;
    11000) v="4th";;
    *) v="error";;
esac
echo $v;
```

按照 OpenWRT 官网资料（<http://wiki.openwrt.org/toh/d-link/dir-505#buttons>），有一款 DIR-505L 的开关是四个档位的，DIR-505 其实也有，但外壳把第四个档位给挡住了，掰不到。如果您足够蛋疼，可以用刻刀给它开开口……

我们可以在开机脚本（/etc/rc.local）里加入判断代码，根据档位做不同配置，以实现不同用途。
