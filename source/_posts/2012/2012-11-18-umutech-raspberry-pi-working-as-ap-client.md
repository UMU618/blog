---
layout: post
title: 用树莓派 + USB 无线网卡做了一个蛋疼的 AP Client
date: 2012-11-18 23:21:09
categories: UMUTech
tags:
- embedded
---
## 需求

两个困境：

- 有个只有 RJ45 接口的旧设备要上网，它的位置离路由器很远，家里的网线不够长……

- UMU 买了一个支持 AP Client 的无线路由器（TP-Link TL-WR800N）每次把这个 AP Client 断电时，提供网络的主路由器都会被这个 AP Client 搞死掉，原因未知……

还没有树莓派的时候，UMU 用笔记本上的 Windows 的 ICS 功能给它提供网络，当然这方法很不好，于是有了现在的方案。

## 解决

仔细查看了树莓派支持的 USB WiFi Adapters 列表，挑选了 TP-LINK TL-WN823N（RT8192CU 芯片），这个在 Raspbian 上是 Work out-of-box。Mercury 150Mbps MW150U（Realtek RTL8188CU 芯片）也可以。

硬件准备好后，第一步，配置 Wifi，连上主路由器，假定，此步将 wlan0 的 IP 配置为 192.168.1.2，`/etc/network/interfaces` 的内容如下：

```
auto lo

iface lo inet loopback
iface eth0 inet static
address 192.168.24.51
netmask 255.255.255.0

auto wlan0
iface wlan0 inet static
address 192.168.1.2
netmask 255.255.255.0
gateway 192.168.1.1
wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

iface default inet dhcp
```

192.168.24.51 这个 IP 可以改为别的，UMU 习惯用这个当网关地址。`/etc/wpa_supplicant/wpa_supplicant.conf` 的内容这里就忽略了，参考 <http://elinux.org/RPi_Peripherals#Wireless:_TP-Link_TL-WN722N_USB_wireless_adapter_.28Debian_6.29>。设置完，`ifdown wlan0` 再 `ifup wlan0`，看看 USB 无线网卡的指示灯应该闪起来了……

第二步，配置 RJ45 网口的 NAT。首先，修改 `/etc/sysctl.conf`，增加以下两行：

```
net.ipv4.ip_forward=1
net.ipv4.conf.all.accept_source_route = 1
```

运行 `echo 1 > /proc/sys/net/ipv4/ip_forward` 和 `iptables -t nat -A POSTROUTING -s 192.168.24.0/24 -o wlan0 -j SNAT --to 192.168.1.2`，并将这条命令写到 `/etc/rc.local` 中的 `exit` 前。

最后，`reboot` 一下试试。可以用网线把 PC 和树莓派连起来，PC 的网卡设为 24 段地址，网关 192.168.24.51，试一下 PC 通过树莓派的网口上网吧！
