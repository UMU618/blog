---
layout: post
title: 解决通过 TTL 登陆 Armbian 时的 Login incorrect
date: 2020-04-09 23:18:36
categories: UMUTech
tags:
- armbian
- debug
- embedded
- linux
- ops
---
## 问题

用 TTL 连接刷 Armbian buster 的盒子，然后用 putty 和 plink 登陆，一输入 root 回车，就报 Login incorrect！但通过 ssh 远程登陆却没任何问题。

## 原因

ttyAML0 不在 /etc/securetty 里。

## 分析过程

一开始听运维小伙伴说：最可能的原因是键盘 Caps 开启了。轻松排除。

后来发现输入大写的 ROOT，反而提示输入 password，这让稣想到“枚举用户”攻击。开始思考，是不是通过 TTL 登陆被 Armbian 认为是不安全的？

于是学习 securetty 相关知识，发现确实在某些不安全场合 root 是不被允许登陆的，因为系统管理员一旦通过不安全渠道输入密码，那么密码就可能被盗取，所以一输入 root，就应该立刻报错，而不该继续让输入密码。而输入其它不存在的用户时（比如大写的 ROOT），反而应该让继续输入密码，最后再提示登陆失败，因为如果提示用户不存在，会让黑客穷举出系统里有什么账号。

```sh
grep securetty /etc/pam.d/login
# Disallows root logins except on tty's listed in /etc/securetty
auth [success=ok new_authtok_reqd=ok ignore=ignore user_unknown=bad default=die] pam_securetty.so
```

后来注意到 TTL 用的 tty 名字是 ttyAML0，`grep ttyAML0 /etc/securetty` 果然不存在。

## 解决

`echo ttyAML0 >> /etc/securetty` 搞定。
