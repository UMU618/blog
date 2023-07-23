---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇2）：连上路由器
date: 2014-06-02 01:00:35
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
## 登录

刚刚刷好 OpenWRT 的 DIR-505，默认没有开启 WiFi，所以只能用网线连接，连起来后，将电脑的网卡配置为 192.168.1.X，或者自动获得地址也行。

默认也没有开启 SSH，只能用 `telnet 192.168.1.1` 来连，连上后改 root 密码：

```sh
passwd root
```

改好密码后，先不要关闭 telnet，因为一关闭，下次就连不上了。马上用 SSH 客户端（推荐使用 putty，文件复制、编辑则推荐 winscp）连一下路由器：

```sh
ssh 192.168.1.1
```

如果连接失败，需要在 telnet 会话下开启 SSH 服务：

```sh
/etc/init.d/dropbear enable
/etc/init.d/dropbear start
```

## 开启 WiFi

用有线连接比较麻烦，接下来开启 WiFi，实现无线连接：

```sh
uci set wireless.@wifi-device[0].disabled=0;
uci set wireless.@wifi-iface[0].ssid='umu618.com';
uci set wireless.@wifi-iface[0].encryption='psk2+ccmp';
uci set wireless.@wifi-iface[0].key='password';
uci commit wireless;
wifi
```

## 改时区

时间都用网络同步，所以使用正确的时区很重要，要改为当地的时区，比如 UMU 使用台北时间：

```sh
uci set system.@system[0].zonename='Asia/Taipei';
uci set system.@system[0].timezone='CST-8';
uci commit system;

echo CST-8 > /etc/TZ;
```

## 改机器名

个性化，非必要：

```sh
echo 'DIR-505' > /proc/sys/kernel/hostname;

uci set system.@system[0].hostname='DIR-505';
uci commit system;

/etc/init.d/dnsmasq restart;
```

## 改欢迎语

个性化，非必要：

```sh
vi /etc/banner
```
