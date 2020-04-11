---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇4）：配置网络
date: 2014-06-16 19:23:07
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
## 前情

在上篇《[跟 UMU 一起玩 OpenWRT（入门篇3）：工作模式开关](/2014/06/05/umutech-openwrt-primer-3-gpio-switch/)》中介绍了如何通过 GPIO 读取获得当前工作模式，现在来实现这个开关的功能。

## 配置开机启动脚本

主要配置 rc.local 脚本，内容如下：

```sh
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
	11000) v="4th";;
	10001) v="ap";;
	11001) v="repeater";;
	01001) v="hotspot";;
	*) v="error";;
esac

if [ "$v" != "error" ]; then
	old=`cat /etc/config/working_mode`
	if [ "$v" != "$old" ]; then
		echo "$v" > /etc/config/working_mode
		cp /etc/config/$v/* /etc/config/
		logger "working mode: $old -> $v"
	else
		logger "working mode: $old"
	fi
fi

exit 0
```

## 根据工作模式配置网络

上一步加的代码是，开机自动复制配置文件覆盖到 /etc/config/ 下，配置文件不需要全部，只要 firewall、network 和 wireless 就行了。按工作模式命名，创建三个目录：

```sh
mkdir /etc/config/ap
mkdir /etc/config/repeater
mkdir /etc/config/hotspot
```

后面，我们会把配置文件写好了，放在这些目录里。

### ap 模式

下面以 ap 为例，此模式是网口做 WAN，无线做 LAN，WAN 以 ADSL 接入为例，其它情况自行变通：

```sh
uci set wireless.@wifi-device[0].disabled=0;
uci set wireless.@wifi-iface[0].ssid='umu618.com';
uci set wireless.@wifi-iface[0].encryption='psk2+ccmp';
uci set wireless.@wifi-iface[0].key='WiFi 密码';
uci commit wireless;
wifi

uci delete network.lan.ifname
uci delete network.lan.type
uci add network interface
uci rename network.@interface[-1]='wan'
uci set network.@interface[-1].ifname='eth1'
uci set network.@interface[-1]._orig_ifname='eth1'
uci set network.@interface[-1]._orig_bridge='false'
uci set network.@interface[-1].proto='pppoe'
uci set network.@interface[-1].username='宽带账号'
uci set network.@interface[-1].password='宽带密码'
uci commit network

cp /etc/config/firewall /etc/config/ap/
cp /etc/config/network /etc/config/ap/
cp /etc/config/wireless /etc/config/ap/
```

### repeater 模式

再来说一下 repeater，网口做 LAN，无线做一个 AP Client 和一个 AP。注意，以下命令以初始化的配置文件为基础，请事先备份、还原，不要在配置过前面的 ap 模式后直接继续配置，可能会有问题。

```sh
uci add network interface
uci rename network.@interface[-1]='wwan'
uci set network.@interface[-1].proto='dhcp'
uci commit network
uci set wireless.@wifi-device[0].disabled=0;
uci set wireless.@wifi-iface[0].ssid='umu618.com';
uci set wireless.@wifi-iface[0].encryption='psk2+ccmp';
uci set wireless.@wifi-iface[0].key='WiFi 密码';

uci add wireless wifi-iface
uci set wireless.@wifi-iface[-1].network='wwan'
uci set wireless.@wifi-iface[-1].ssid='现有 WiFi 名字'
uci set wireless.@wifi-iface[-1].device='radio0'
uci set wireless.@wifi-iface[-1].mode='sta'
uci set wireless.@wifi-iface[-1].bssid='现有无线路由器的 MAC 地址'
uci set wireless.@wifi-iface[-1].encryption='psk2+ccmp'
uci set wireless.@wifi-iface[-1].key='现有 WiFi 密码'
uci commit wireless;
wifi

uci get firewall.@zone[1].network
uci show firewall.@zone[1]
uci set firewall.@zone[1].network='wan wan6 wwan'
uci commit firewall
/etc/init.d/firewall restart

cp /etc/config/firewall /etc/config/repeater/
cp /etc/config/network /etc/config/repeater/
cp /etc/config/wireless /etc/config/repeater/
```
