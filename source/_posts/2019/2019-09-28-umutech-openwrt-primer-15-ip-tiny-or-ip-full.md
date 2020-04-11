---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇15）：ip-tiny 和 ip-full 的区别
date: 2019-09-28 21:56:16
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
## 起源

今天看到有网文《[iptables+tproxy实现ss-redir的UDP转发的方法](https://blog.csdn.net/lvshaorong/article/details/53203674)》说：“OpenWrt 做 UDP 转发需要的依赖是：iptables-mod-tproxy, kmod-ipt-tproxy 和 ip-full”。使用 `opkg install ip` 安装的默认是 ip-tiny，一般情况下都是够用的，不禁想弄明白两者有何区别。

## 探索

拿 `ip` 命令对比测试：

- ip-tiny

```
OBJECT := { link | address | route | rule | neigh | tunnel | maddress |
            mroute | mrule | monitor | netns | macsec | token | ila |
            vrf | sr }
```

- ip-full

```
OBJECT := { link | address | addrlabel | route | rule | neigh | ntable |
            tunnel | tuntap | maddress | mroute | mrule | monitor | xfrm |
            netns | l2tp | fou | macsec | tcp_metrics | token | netconf | ila |
            vrf | sr }
```

即 ip-full 多了这些对象： `addrlabel | ntable | tuntap | xfrm | l2tp | fou | tcp_metrics | netconf`。举个例子，只安装 ip-tiny 时，运行 `ip xfrm` 报错如下：

> Object "xfrm" is unknown, try "ip help".

相关知识：

> xfrm is an IP framework for transforming packets (such as encrypting
> their payloads). This framework is used to implement the IPsec
> protocol suite (with the state object operating on the Security
> Association Database, and the policy object operating on the Security
> Policy Database). It is also used for the IP Payload Compression
> Protocol and features of Mobile IPv6.

## 结论

实际上，转发普通 UDP 包，并不需要 ip-full，ip-tiny 即可。
