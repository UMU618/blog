---
layout: post
title: 管理 macOS 的登录项
date: 2022-05-08 16:22:29
categories: UMUTech
tags:
- debug
- ops
- macos
---
## 问题

稣在 macOS 上装了「Free Download Manager」，结果每次系统重启登录后它都会自启动，稣明明没让它这么做的！以下图形界面下的方法都试过，还是自启动！

- 在「Dock 栏」右击「Free Download Manager」的图标，选择「选项」，确认「登录时打开」没有打勾。

- 「系统偏好设置」-「用户与群组」-「登录项」里面也没有「Free Download Manager」。

## 学习

1. 从苹果官方文档开始，第一篇相关文档：[Designing Daemons and Services](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/DesigningDaemons.html#//apple_ref/doc/uid/10000172i-SW4-BBCBHBFB) 说有图形界面的自启动机制叫做 Login item。

2. 于是转到 [Adding Login Items](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLoginItems.html#//apple_ref/doc/uid/10000172i-SW5-BAJJBJEG)，这篇文章说明添加「登录项」有两种方式，具体方法已经是给开发者用的了，对于咱们反向操作（删掉它）帮助不大。

3. 但是通过搜索 `SMLoginItemSetEnabled` 可以知道相关路径为 `XX/Library/LaunchAgents`。

## 操作

1. 首先要得到「Free Download Manager」的程序 ID：

```sh
$ ls /Applications | grep 'Free Download Manager'
Free Download Manager.app

$ ls /Applications/Free\ Download\ Manager.app/Contents
Frameworks     Info.plist     MacOS          PkgInfo        PlugIns        Resources      _CodeSignature translations

$ grep -A1 CFBundleIdentifier /Applications/Free\ Download\ Manager.app/Contents/Info.plist
        <key>CFBundleIdentifier</key>
        <string>org.freedownloadmanager.fdm6</string>
```

以上，专业的一条命令为：

```sh
$ defaults read /Applications/Free\ Download\ Manager.app/Contents/Info.plist CFBundleIdentifier
org.freedownloadmanager.fdm6
```

2. 查找 `org.freedownloadmanager.fdm6` 有没有在 `XX/Library/LaunchAgents`

```sh
$ find ~/Library/LaunchAgents /Library/LaunchAgents /System/Library/LaunchAgents -name org.freedownloadmanager.fdm6\*
/Users/YourUserName/Library/LaunchAgents/org.freedownloadmanager.fdm6.plist
```

3. 改 `org.freedownloadmanager.fdm6.plist` 禁止自启动

```sh
$ defaults read ~/Library/LaunchAgents/org.freedownloadmanager.fdm6.plist RunAtLoad
1

$ defaults write ~/Library/LaunchAgents/org.freedownloadmanager.fdm6.plist RunAtLoad 0

$ defaults read ~/Library/LaunchAgents/org.freedownloadmanager.fdm6.plist RunAtLoad
0
```

搞定。
