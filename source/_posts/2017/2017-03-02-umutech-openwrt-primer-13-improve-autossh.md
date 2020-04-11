---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇13）：改进 autossh 支持多实例
date: 2017-03-02 11:46:52
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
## 需求

在之前的文章《[跟 UMU 一起玩 OpenWRT（入门篇10）：穿透内网](/2014/07/27/umutech-openwrt-primer-10-through-the-intranet/)》，介绍了 autossh 的使用，现在多个需求：想在内网打通多条隧道，即让 autossh 能运行多个 ssh 实例。

## 解决

- 首先在 /etc/config/autossh 里增加一个 section，看起来如下：

```
config autossh
        option gatetime '0'
        option monitorport '0'
        option poll '600'
        option ssh '-i /etc/dropbear/id_rsa -N -T -R 2222:localhost:22 root@Server1'

config autossh
        option gatetime '0'
        option monitorport '0'
        option poll '600'
        option ssh '-i /etc/dropbear/id_rsa -N -T -R 2222:localhost:22 root@Server2'
```

- 然后改进一下 /etc/init.d/autossh，让它支持多实例，给 start_instance() 函数增加两行：

```
export SERVICE_MATCH_NAME=1
export SERVICE_NAME="$section"
```

- 最终 start_instance() 函数看起来是这样：

```
start_instance() {
	local section="$1"

	config_get ssh "$section" 'ssh'
	config_get gatetime "$section" 'gatetime'
	config_get monitorport "$section" 'monitorport'
	config_get poll "$section" 'poll'

	export AUTOSSH_GATETIME="${gatetime:-30}"
	export AUTOSSH_POLL="${poll:-600}"
	export SERVICE_MATCH_NAME=1
	export SERVICE_NAME="$section"
	#export SERVICE_DEBUG=1
	service_start /usr/sbin/autossh -M ${monitorport:-20000} -f ${ssh}
}
```

## 注意事项

这样改是有副作用的，您反复启动多次就知道了……启动的命令是：

```sh
/etc/init.d/autossh start
```
