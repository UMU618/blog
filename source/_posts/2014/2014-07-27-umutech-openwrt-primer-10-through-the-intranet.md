---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇10）：穿透内网
date: 2014-07-27 00:45:32
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
UMU 把路由器放在公司，然后在家里想登陆它，这时候就有一个问题：如何穿越到公司内网呢？本文给出的一种解决方案：SSH 反向连接。涉及的软件是 autossh。

您需要准备一台有固定外网 IP 的服务器，UMU 使用的是某某云主机（避免广告嫌疑就不说了，呵，广告位招租），如果不想出钱购买，可以用家庭 ADSL + 动态域名代替，效果可能差一些，但基本可用。

为了更清晰地说明，列一下各个角色：

1. 控制端：UMU 的笔记本，不管在什么网络，都要求能够连接到放在公司的路由器；

2. 中转服务器：一台某某云主机，固定 IP，用 cloud_ip 表示；

3. 被控端：放在公司的路由器，内网 IP，用 internal_ip 表示。

基本原理：让被控端主动连接中转服务器，然后控制端连接中转服务器，就可以间接连接被控端了。

被控端安装、设置，主要参考：<http://wiki.openwrt.org/doc/howto/autossh>

```sh
opkg update
opkg install autossh
dropbearkey -t rsa -f /etc/dropbear/id_rsa
dropbearkey -y -f /etc/dropbear/id_rsa | grep ssh-rsa
```

把上面最后一行命令的输出复制下，注意只有一行，待会儿要上传到中转服务器。或者也可以把最后一条命令改为打印到文件，再用 WinSCP 下载到本地。

```sh
dropbearkey -y -f /etc/dropbear/id_rsa | grep ssh-rsa > /tmp/pubkey
```

查看一下 autossh 配置：

```sh
uci get autossh.@autossh[0].ssh
```

如果没有问题，就把中转服务器的信息设置上去：

```sh
uci set autossh.@autossh[0].ssh='-i /etc/dropbear/id_rsa -f -N -T -R 2222:localhost:22 <user>@<cloud_ip>'
uci commit
```

接下来登录到中转服务器（Linux Server，如果是 OpenWRT，要把以下的 `~/.ssh/authorized_keys` 换成 `/etc/dropbear/authorized_keys`），把公钥（`/tmp/pubkey`）上传：

```sh
echo "key 内容" >> ~/.ssh/authorized_keys
## 或者
#cat pubkey >> ~/.ssh/authorized_keys
chmod 0700 ~/.ssh/
chmod 0600 ~/.ssh/authorized_keys
vi /etc/ssh/sshd_config
## 改为允许证书登录
service sshd restart
```

`/etc/ssh/sshd_config` 需要打开的有：

```
RSAAuthentication yes

PubkeyAuthentication yes

AuthorizedKeysFile .ssh/authorized_keys

GatewayPorts yes
```

到路由器上测试：

```sh
ssh -i /etc/dropbear/id_rsa -f -N -T -R 2222:localhost:22 <user>@<cloud_ip>
```

如果成功则大功告成，以后只需要 ssh 到中转服务器的 2222 端口就等于连接到路由器了。最后配合本地端口转发，可以连接很多内网机器了。如下图：

![Putty](/images/20140727-putty.jpg)

再加一台路由器，用于做本地端口转发，就可以让 Surface、iPad 之类的设备也能快乐地穿透到内网了。
