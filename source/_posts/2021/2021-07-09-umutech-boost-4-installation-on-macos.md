---
layout: post
title: Boost【4】在 macOS 上安装
date: 2021-07-09 14:58:35
categories: UMUTech
tags:
- boost
- cpp
- dev
- macos
---
其它系统请参考：

- Windows：《[Boost【1】安装](/2020/09/11/umutech-boost-1-installation/)》

- 《[Boost【3】在 Linux 上安装](/2021/07/06/umutech-boost-3-installation-on-linux/)》

## 软件环境

- macOS arm64

- XCode

```sh
brew install p7zip
```

## brew 安装法

macOS 的 brew 更进 boost 很积极，现在就是最新版 1.76：

```sh
# install b2 and boost
brew install boost-build boost
```

然后跳到文末的“测试安装”。

## 源码安装法

### 1. 下载

```sh
cd ~/Downloads
wget https://boostorg.jfrog.io/artifactory/main/release/1.76.0/source/boost_1_76_0.7z
```

### 2. 解压

```sh
7z x boost_1_76_0.7z
```

### 3. 编译和安装 b2

```sh
cd boost_1_76_0
# if you have multiple compilers
# ./bootstrap.sh --cxx=clang++
./bootstrap.sh
cd tools/build
cp ../../b2 ./
./b2 install
cd ../../
cp ./project-config.jam $HOME/user-config.jam
```

### 4. 用 b2 编译 Boost

```sh
b2 install
```

## 测试安装

可以用以下仓库验证前面操作是否正确：

<https://github.com/UMU618/test_boost>

```
git clone https://github.com/UMU618/test_boost
cd test_boost
sh build.sh
```

最终编译出来的程序应该打印“OK!”。
