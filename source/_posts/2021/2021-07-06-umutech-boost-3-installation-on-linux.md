---
layout: post
title: Boost【3】在 Linux 上安装
date: 2021-07-06 16:36:49
categories: UMUTech
tags:
- boost
- cpp
- dev
- linux
---
在《[Boost【1】安装](/2020/09/11/umutech-boost-1-installation/)》中以 Windows 为例介绍 Boost 的安装，最近主要在 Linux 上开发，发现差异还是比较大的，于是又有了本文。

## 软件环境

- Debian Buster/Bullseye

- clang-11/gcc-10

```sh
apt install wget -y
apt install p7zip-full -y
# apt install clang-11 -y
# ln -s /usr/bin/clang-11 /usr/bin/clang
# ln -s /usr/bin/clang++-11 /usr/bin/clang++
# clang++ --version
apt install g++-10 -y
ln -s /usr/bin/gcc-10 /usr/bin/gcc
ln -s /usr/bin/g++-10 /usr/bin/g++
g++ -v
apt install python3-dev -y
```

## apt 安装法

如果您怕麻烦，而且不在乎使用最新版本 Boost，还可以直接用 apt 安装 1.74 的版本：

```sh
# install b2
apt install libboost1.74-tools-dev
# install boost
apt install libboost1.74-all-dev
```

然后跳到文末的“测试安装”。

## 源码安装法

### 1. 下载

```sh
cd ~
# wget https://boostorg.jfrog.io/artifactory/main/release/1.76.0/source/boost_1_76_0.7z
wget https://boostorg.jfrog.io/artifactory/main/release/1.78.0/source/boost_1_78_0.7z
```

### 2. 解压

```sh
# 7z x boost_1_76_0.7z
7z x boost_1_78_0.7z
```

### 3. 编译和安装 b2

```sh
# use /usr to avoid setting PATH
INSTALL_DIR=/usr

# cd boost_1_76_0
cd boost_1_78_0
# if you have multiple compilers
# ./bootstrap.sh --cxx=clang++
./bootstrap.sh --with-python-version=3.7
cd tools/build
cp ../../b2 ./
apt install bison -y
./b2 install --prefix=$INSTALL_DIR
cd ../../
cp ./project-config.jam $HOME/user-config.jam
```

### 4. 用 b2 编译 Boost

```sh
# b2 install --build-type=complete --layout=versioned --prefix=$INSTALL_DIR --exec-prefix=$INSTALL_DIR
b2 install --prefix=$INSTALL_DIR --exec-prefix=$INSTALL_DIR
```

## 测试安装

```sh
mkdir ~/boost-3-installation-on-linux
cd ~/boost-3-installation-on-linux

cat > main.cpp << EOF
// clang -std=c++20 main.cpp -ldl -lstdc++ -lstdc++fs -lboost_program_options
#include <filesystem>
#include <iostream>
#include <string>

#include <boost/program_options.hpp>

namespace po = boost::program_options;

int main(int argc, char* argv[]) {
  try {
    std::filesystem::path me(argv[0]);
    po::options_description desc(std::string("Usage of ") + me.filename().string());
    desc.add_options()("help,h", "produce help message");
    po::variables_map vm;
    po::store(po::parse_command_line(argc, argv, desc), vm);
    po::notify(vm);

    if (vm.count("help")) {
      std::cout << desc;
      return EXIT_SUCCESS;
    }
  } catch (std::exception& e) {
    std::cerr << "Invalid argument: " << e.what() << "\n";
    return EXIT_FAILURE;
  } catch (...) {
    std::cerr << "Invalid argument: unknown exception!\n";
    return EXIT_FAILURE;
  }

  std::cout << "OK!\n";
  return EXIT_SUCCESS;
}
EOF

cat > Jamfile.v2 << EOF
import os ;
  BOOST_ROOT = [ os.environ BOOST_ROOT ] ;

lib dl ;
lib stdc++fs ;
lib boost_program_options ;

project boost3
  : requirements
  <cxxstd>latest
    <target-os>linux:<library>dl
    <target-os>linux:<library>stdc++fs
    <target-os>linux:<library>boost_program_options
    <target-os>windows:<include>\$(BOOST_ROOT)
    <target-os>windows:<library-path>\$(BOOST_ROOT)/stage/lib
  <threading>multi
  : default-build release
  : build-dir ./bin
  ;

exe boost3
  : main.cpp
  ;
EOF

touch project-config.jam

b2
# b2 link=static
# b2 link=static runtime-link=static
```
