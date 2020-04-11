---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇11）：文件共享
date: 2014-07-27 00:52:14
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
## 需求

组建文件共享服务。

## 解决

文件共享可以通过 FTP/FTPS、SFTP、NFS、Windows 文件共享（网上邻居）。其中 FTP/FTPS、SFTP 需要先下载，无法“直接打开”，实用性较差就不介绍了，重点放在 Windows 文件共享，最后再简单介绍一下 NFS。

### Windows 文件共享

OpenWRT 使用 samba 提供 Windows 文件共享服务。如有条件应该使用 samba4，安装命令为：

```sh
opkg update
opkg install samba4-server
# 如果 samba4 不可用，则安装 samba3.6
# opkg install samba36-server
```

参考：

- [Samba (smb)](https://openwrt.org/docs/guide-user/services/nas/samba)

- [【硬创邦】跟hoowa学做智能路由(十一)：实现网络存储与文件共享](https://www.leiphone.com/news/201406/diy-a-smart-router-topic-share.html) 的“安装 Windows 文件共享”章节。

如果配置后，无法正常访问，您可以参考一下《[Windows 7 无法访问 NAS 或 Samba 服务器之解决](https://www.win7china.com/html/5956.html)》。

### NFS

NFS 和 Windows 文件共享是两大文件共享服务，NFS 在 Linux 设备之间的传输效率高于 samba，但大部分客户端都是 Windows，所以 UMU 更推荐 samba。

参考：

- OpenWRT 安装 NFS 服务端：官方文档 [Network File System (NFS)](https://openwrt.org/docs/guide-user/services/nas/nfs.server)；

- Windows 安装 NFS 客户端：《[win7 中使用NFS共享](https://www.cnblogs.com/itech/archive/2012/06/17/2552514.html)》。
