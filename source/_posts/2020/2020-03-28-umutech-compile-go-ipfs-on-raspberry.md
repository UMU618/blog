---
layout: post
title: 在树莓派上编译 go-ipfs
date: 2020-03-28 21:39:02
description:
categories: UMUTech
tags:
- dev
- embedded
- go
- linux
---
## 需求

用 PC 当 Server 测试环境，费电！挖出吃灰多年的树莓派 Model B Rev 2 000f，打算用它跑 ipfs！

## 系统选型

- 较熟悉的 CentOS、FreeBSD、Ubuntu Server、Windows IoT 的当代主流版本都不支持这款古老的树莓派。

- ArchLinux 支持，然而稣个人认为 ArchLinux ~~属于 Linux 中的邪教~~，不适合当 Server。

- 尝试刷 OpenWRT，发现即使设置密码，本地控制台也是没密码就能登陆。这不太安全，虽然本地就是不安全的，但别的系统可不是这么设计的！

- 还是官方的 Raspbian Buster Lite 吧！

## 安装和配置系统

主要参考官方文档：

1. [Setup](https://www.raspberrypi.org/documentation/setup/)：选个 16GB 的 SD 卡。
    
2. [Installing operating system images](https://www.raspberrypi.org/documentation/installation/installing-images/README.md)：用官方 Raspberry Pi Imager 工具把系统镜像刷到 SD 卡。

3. 接 HDMI 显示，通电。首次启动，系统会自动对 SD 卡的分区进行扩容，使第二个分区扩满未分配空间。

4. 通过 `raspi-config` 做基本配置：把默认语言 en_GB.UTF-8 去掉，勾选 en_US.UTF-8；键盘布局改为通用 104 键；时区改为当地。

5. 还是通过 `raspi-config` 改 pi 用户的密码，再开启 SSH，插上网线，就可以从别处远程登陆它了。

## 安装 Go 编译器

不要通过 `sudo apt install golang` 安装，因为截至今天（2020-03-28），这命令安装的是 1.11.6 版，这对 [go-ipfs](https://github.com/ipfs/go-ipfs) 项目来说太低了。

到 [golang 官网下载](https://golang.org/dl/) ARMv6 安装包，目前最新版本是 [1.14.1](https://dl.google.com/go/go1.14.1.linux-armv6l.tar.gz)。

压缩包里是有一个 go 文件夹的，所以只要解压到 /usr/local/ 下即可。

然后：

```sh
sudo ln -s /usr/local/go/bin/go /usr/bin/go
sudo ln -s /usr/local/go/bin/gofmt /usr/bin/gofmt
```

## 编译

```sh
git clone https://github.com/ipfs/go-ipfs
cd go-ipfs
make build
```

有很多依赖库需要下载，开始漫长等待……如果代码都下载完，则 `make build` 的输出为：

```
go version go1.14.1 linux/arm
bin/check_go_version 1.14.1
plugin/loader/preload.sh > plugin/loader/preload.go
go fmt plugin/loader/preload.go >/dev/null
go build  "-asmflags=all='-trimpath='" "-gcflags=all='-trimpath='" -ldflags="-X "github.com/ipfs/go-ipfs".CurrentCommit=3561de074-dirty" -o "cmd/ipfs/ipfs" "github.com/ipfs/go-ipfs/cmd/ipfs"
```

最后一行会卡很久！em……用高性能机器来交叉编译才是正确的方式！
