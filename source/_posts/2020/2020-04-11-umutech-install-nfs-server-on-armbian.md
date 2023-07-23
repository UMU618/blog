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

配置文件为 /etc/ganesh/ganesha.conf。nfs-ganesha-vfs 包另带一个 vfs.conf 参考模板。以下配置创建读写共享 /root/share 和只读共享 /opt：

```
EXPORT_DEFAULTS
{
        Protocols = 4;
}

EXPORT
{
        Export_Id = 77;
        Protocols = 3, 4;
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
        Access_Type = RO;
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
sudo apt install nfs-common

showmount -e u1
Export list for u1:
/root/share (everyone)

sudo mkdir /mnt/share
# sudo mount -t nfs u1:/root/share /mnt/share
sudo mount.nfs u1:/root/share /mnt/share
```

**注意**：如果提示 `mount.nfs: No such device`，说明内核没有 NFS 模块，~~洗洗睡了~~，换 Windows 10 吧！

- Windows 10

![安装 NFS 客户端](/images/20200411-install-nfs-client.png)

![浏览 NFS 共享目录](/images/20200411-browse-nfs.png)

![NFS 属性](/images/20200411-nfs-property.png)

**注意**：Windows 10 目前只有 NFS v3 客户端。服务端如果只开 v4 协议，则 Windows 10 将无法访问。

## 参考

<https://github.com/nfs-ganesha/nfs-ganesha/blob/next/src/config_samples/config.txt>

<https://github.com/nfs-ganesha/nfs-ganesha/blob/next/src/config_samples/export.txt>
