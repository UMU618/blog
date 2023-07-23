---
layout: post
title: macOS 研究经验【2】：简单的破解
date: 2020-03-18 13:19:23
categories: UMUTech
tags:
- debug
- macos
---
## 1. Beyond Compare

BC 试用版过期，思考 3 秒钟：~~稣太穷，买不起！~~

macOS 和 iOS 一样，App 都是独立存储，找出安装信息保存在哪个文件应该很容易。

确实如此！居然只要两步：

```sh
cd ~/Library/Application Support/Beyond Compare
rm registry.dat
```

## 2. X-NG

由于子公司、分公司众多，稣的服务器列表里有好多个项，想备份这个列表，发现还不是很容易！

![ServerProfileManager](/images/20200318-0.jpg)

首先，找到 `~/Library/Application Support/X-NG/-local-config.json`，但这个文件里只有当前选择的项。

然后，就看代码吧！Swift 写的，应该还好：

```swift
let defaults = UserDefaults.standard
let keys = [
```

根据代码线索找到：

```sh
defaults read ~/Library/Preferences/com.yuzhou.X-NG.plist
```

哇~全部出来了！
