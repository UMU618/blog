---
layout: post
title: chsh -s zsh
date: 2020-03-23 15:53:16
categories: UMUTech
tags:
- ops
- linux
---
## 标题是个坑

不要在 Ubuntu 上运行这条命令！都说 CentOS 比 Ubuntu 稳定，总算见识到具体案例！

## 没有对比就没有伤害

### CentOS

```sh
$ cat /etc/centos-release
CentOS Linux release 7.7.1908 (Core)

$ chsh -s zsh
Changing shell for root.
chsh: shell must be a full path name
```

可见，机智的 CentOS，早就料到这个运维事故！

### Ubuntu

```sh
$ cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=18.04
DISTRIB_CODENAME=bionic
DISTRIB_DESCRIPTION="Ubuntu 18.04.4 LTS"

$ chsh -s zsh
chsh: Warning: zsh does not exist

$ cat /etc/passwd
root:x:0:0:root:/root:zsh
```

SSH 到 Ubuntu Server 上，运行 `chsh -s zsh`，`exit` 后就再也无法登陆……

如果您要远程做这个试验，记得 `exit` 前 `chsh -s /bin/zsh`，或者 `vi` 手动纠正 /etc/passwd。
