---
layout: post
title: nfs-ganesha 端口绑定
date: 2020-04-16 22:15:55
categories: UMUTech
tags:
- armbian
- debug
- embedded
- linux
- openwrt
- ops
---
## 需求

前文《[在 Armbian 安装 NFS 服务端](/2020/04/11/umutech-install-nfs-server-on-armbian/)》介绍 nfs-ganesha 的安装配置。经过几天使用，发现效果还可以，但一直都是在局域网内使用，突然有一天想在公司访问家里的 NFS 共享……

## 问题

直接 mount 会一直卡着。

开放 111、2049 端口，再 mount，还是卡着。

通过反复重启 nfs-ganesha 并 `netstat -nalp | grep ganesha.nfsd` 观测，发现 mountd 端口不固定！给开放端口配置带来困难。

## 解决

### Armbian 配置

将 mountd 端口绑定，比如 2618，但配置的方法和常规 Debian 服务器的内核级 NFS Server 不同。修改 /etc/services 添加 mountd 绑定是无用的，应该编辑 /etc/ganesha/ganesha.conf，添加以下配置：

```
NFS_CORE_PARAM
{
        MNT_Port = 2618;
}
```

改完重启服务：

```sh
systemctl restart nfs-ganesha.service
```

### 防护墙配置

UMU 用的是 OpenWRT 路由器作为家庭网络出口，firewall 配置文件是 /etc/config/firewall，添加以下几行：

```
config rule
        option src 'wan'
        option name 'Allow-NFS'
        option dest '*'
        option target 'ACCEPT'
        option dest_port '111 2049 2618'
        option family 'ipv6'
        option start_time '00:09:00'
        option stop_time '00:21:00'
        option enabled '0'
```

以上配置开启了 111、2049、2618 三个端口的转发，其中 udp/tcp 111 是 portmap 端口，udp/tcp 2049 是 nfsd 端口，udp/tcp 2618 是上一步绑定的 mountd 端口。

`/etc/init.d/firewall restart` 重启后就可以在办公室通过 IPv6 访问家里的 NFS 共享了。


## 参考

<https://github.com/nfs-ganesha/nfs-ganesha/blob/next/src/config_samples/config.txt>
