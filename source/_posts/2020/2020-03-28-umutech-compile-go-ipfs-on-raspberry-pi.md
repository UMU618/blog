---
layout: post
title: 在树莓派上编译 go-ipfs
date: 2020-03-28 21:39:02
categories: UMUTech
tags:
- dev
- embedded
- go
- linux
- raspbian
---
## 1. 需求

用 PC 当 Server 测试环境，费电！挖出吃灰多年的树莓派 Model B Rev 2 000f，打算用它跑 ipfs！

## 2. 系统选型

- 较熟悉的 CentOS、FreeBSD、Ubuntu Server、Windows IoT 的当代主流版本都不支持这款古老的树莓派。

- ArchLinux 支持，然而稣个人认为 ArchLinux（~~属于 Linux 中的邪教~~）不适合当 Server。

- 尝试刷 OpenWRT，发现即使设置密码，本地控制台也是没密码就能登陆。这不太安全，虽然本地就是不安全的，但别的系统可不是这么设计的！

- 还是官方的 Raspbian Buster Lite 吧！

## 3. 安装系统

主要参考官方文档：

- [Setup](https://www.raspberrypi.org/documentation/setup/)：选个 16GB 的 SD 卡。
    
- [Installing operating system images](https://www.raspberrypi.org/documentation/installation/installing-images/README.md)：用官方 Raspberry Pi Imager 工具把系统镜像刷到 SD 卡。

- 接 HDMI 显示，通电。首次启动，系统会自动对 SD 卡的分区进行扩容，使第二个分区扩满未分配空间。

## 4. 配置系统

通过 `sudo raspi-config` 做基本配置：

- 进“本地化”把默认语言 en_GB.UTF-8 去掉，勾选 en_US.UTF-8。

- 键盘布局改为通用 105 键（国际）美国布局（默认的英国布局下按 | 会变 ~）。

- 时区改为当地。

- 改机器名（如果您有多个树莓派，不改会重名），稣将之改为 rp1b。

- 改 pi 用户的密码。

- 开启 SSH，插上网线或者 USB 无线网卡，就可以从别处远程登陆它了。

## 5. 配置国内 apt 源

```sh
sudo mv /etc/apt/sources.list /etc/apt/sources.list.bak
# 增加阿里云源
echo 'deb https://mirrors.aliyun.com/raspbian/raspbian/ buster main non-free contrib
deb-src https://mirrors.aliyun.com/raspbian/raspbian/ buster main non-free contrib' | sudo tee /etc/apt/sources.list.d/aliyun.list
# 增加清华大学源
echo 'deb https://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib
deb-src https://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib' | sudo tee /etc/apt/sources.list.d/tsinghua.list
```

## 6. 安装 Go 编译器

打算直接在树莓派上编译，所以要先在树莓派上安装编译环境。不过不要通过 `sudo apt install golang` 安装，因为截至今天（2020-03-28），这命令安装的是 1.11.6 版，这对 [go-ipfs](https://github.com/ipfs/go-ipfs) 项目来说太低了。

到 [golang 官网下载](https://golang.org/dl/) ARMv6 安装包，目前最新版本是 [1.14.1](https://dl.google.com/go/go1.14.1.linux-armv6l.tar.gz)。

压缩包里是有一个 go 文件夹的，所以只要解压到 /usr/local/ 下即可。

```sh
# aria2 比 wget 强大
# sudo apt install aria2
aria2c https://dl.google.com/go/go1.14.1.linux-armv6l.tar.gz
tar -C /usr/local -xzf go1.14.1.linux-armv6l.tar.gz
sudo ln -s /usr/local/go/bin/go /usr/bin/go
sudo ln -s /usr/local/go/bin/gofmt /usr/bin/gofmt
```

## 7. 编译项目

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

## 8. 在 macOS 上编译树莓派程序

树莓派的 CPU 架构是 armv6l，所以用以下命令编译：

```sh
CGO_ENABLED=0 GOOS=linux GOARCH=arm GOARM=6 make build
```

在 MBP15 上编译快很多！（~~前面纯属折腾！~~）编完复制到树莓派：

```sh
scp ./cmd/ipfs/ipfs pi@rp1b:/home/pi/
```

在树莓派上测试：

```
pi@rp1b:~ $ uname -a
Linux rp1b 4.19.97+ #1294 Thu Jan 30 13:10:54 GMT 2020 armv6l GNU/Linux

pi@rp1b:~ $ ./ipfs version
ipfs version 0.5.0-dev

pi@rp1b:~ $ ./ipfs init
initializing IPFS node at /home/pi/.ipfs
generating 2048-bit RSA keypair...

pi@rp1b:~ $ ./ipfs daemon
Initializing daemon...
go-ipfs version: 0.5.0-dev-3561de074
Repo version: 9
System version: arm/linux
Golang version: go1.14
Swarm listening on /ip4/127.0.0.1/tcp/4001
Swarm listening on /ip4/192.168.1.91/tcp/4001
Swarm listening on /ip6/240e:379:254c:9d00:969d:e453:9448:1efb/tcp/4001
Swarm listening on /ip6/::1/tcp/4001
Swarm listening on /p2p-circuit
Swarm announcing /ip4/110.87.124.125/tcp/21551
Swarm announcing /ip4/127.0.0.1/tcp/4001
Swarm announcing /ip4/192.168.1.91/tcp/4001
Swarm announcing /ip6/240e:379:254c:9d00:969d:e453:9448:1efb/tcp/4001
Swarm announcing /ip6/::1/tcp/4001
API server listening on /ip4/127.0.0.1/tcp/5001
WebUI: http://127.0.0.1:5001/webui
Gateway (readonly) server listening on /ip4/127.0.0.1/tcp/8080
Daemon is ready
```
