---
layout: post
title: 在 Armbian 安装 NFS 服务端
date: 2020-04-11 16:29:06
categories: UMUTech
tags:
- armbian
- debug
- embedded
- linux
- ops
- windows
---
## 需求

有个刷了 Armbian 的玩客云想当文件共享服务器。

## 问题

某些版本的 Armbian 内核不支持 nfsd，刚好稣就刷到！如果按照 debian 服务器玩法——安装 kernel 版服务端，是无法正常工作的：

```sh
apt install nfs-common nfs-kernel-server
```

启动时会提示：

```
mount: /proc/fs/nfsd: unknown filesystem type 'nfsd'.
proc-fs-nfsd.mount: Mount process exited, code=exited, status=32/n/a
proc-fs-nfsd.mount: Failed with result 'exit-code'.
nfs-mountd.service: Job nfs-mountd.service/start failed with result 'dependency'.
nfs-idmapd.service: Job nfs-idmapd.service/start failed with result 'dependency'.
nfs-server.service: Job nfs-server.service/start failed with result 'dependency'.
```

## 解决

### 服务端

使用用户态的 nfs-ganesha。

```sh
apt install nfs-ganesha nfs-ganesha-vfs
```

配置文件 /etc/ganesh/ganesha.conf，安装 nfs-ganesha-vfs 会多一个 vfs.conf 参考模板，但其实和初始的 ganesha.conf 一模一样。

以下配置共享 /root/share 和 /opt 两个目录：

```
EXPORT
{
        Export_Id = 77;
        Path = /root/share;
        Pseudo = /root/share;
        Access_Type = RW;
        FSAL {
                Name = VFS;
        }
}

EXPORT
{
        Export_Id = 78;
        Path = /opt;
        Pseudo = /opt;
        Access_Type = RW;
        FSAL {
                Name = VFS;
        }
}
```

改完重启服务：

```sh
systemctl restart nfs-ganesha.service
```

### 客户端

- Debian

```sh
apt install nfs-common
```

- Windows 10

![安装 NFS 客户端](/images/20200411-install-nfs-client.png)

![浏览 NFS 共享目录](/images/20200411-browse-nfs.png)
