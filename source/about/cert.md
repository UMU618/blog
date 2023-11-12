---
layout: page
title: 签名证书
date: 2022-10-16 01:18:05
---
## 为什么需要这个证书？

稣开发了一些软件，发布的时候想声明为其安全性负责。一般的做法是去买个证书，然后给这些软件签名。

但是购买证书不仅有成本，还是一件有门槛的事。对于个人开发者来说，不划算，并且也不是一件很有必要的事情。

于是弄了这个自签名证书。只要把证书的指纹公开给大家，那么也有一定的说服力。

## 公示信息

TestSign cert

CN=umu618.com

Fingerprint(指纹): **ff8a8160116e35775506d0dd86360935877aaf3b**

**注意**：重点是指纹，CN 是可以自定义的，别人也可以设置为 umu618.com。

![微信](/images/cert.png)

## 更多验证的地方

以下 URL 也公开了一样的指纹：

- <https://github.com/UMU618/nv-video-info>

- <https://github.com/UMU618/PowerEconomizer>

## 安装证书并信任它【非必要】

可以[下载证书](/images/umu618.com.cer)，或从已签名的程序的属性里查看，再安装。

![安装证书到“受信任的根证书颁发机构”](/images/install_cert.png)
