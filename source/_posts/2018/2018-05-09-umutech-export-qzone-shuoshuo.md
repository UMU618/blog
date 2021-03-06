---
layout: post
title: 批量导出 QQ 空间说说
date: 2018-05-09 22:22:15
categories: UMUTech
tags:
- 作品
---
## 创作故事
　　2015-03-16 为了导出自己的说说，写了个半自动的程序，手动分析几个参数填到代码理，很快就刷刷地下载了 7 年的说说。第二天，就在知乎回答了两贴。

[如何一次性导出QQ空间说说？](https://www.zhihu.com/question/20935223/answer/42220600)

[如何批量导出特定 QQ 号的所有说说？](https://www.zhihu.com/question/24577183/answer/42220837)

　　2015-09-07 发现导出程序失效了，参数有点变化，但很快又跟进。

　　2015-11-27 又失效了，除了 json 字段有变化，还增加了对 Cookie 的验证，于是又加上了 Cookie 的模拟。

　　2015-12-11 又又失效了，这次增加了对 UserAgent 的验证……继续跟进。

　　由于知乎的热度，越来越多的人找 UMU 导出，**但这个程序是半自动的**，会占用宝贵的时间，所以要收点人工费。需要的可以联系微信 UMUTech，QQ：154401181（验证消息：qzone，QQ 比较少用，尽量先加微信）。

## 收费规则

### 1. 8.88 元起，每 1000 条 8.88 元人民币；
### 2. 未满 1000 条部分，采用进一法，即 10001 条，是收费 8.88 * 2 = 17.76 元；
### 3. 封顶 88.88 元，所以，如果您的说说很多也不用害怕会很贵。
### 4. 支持 QQ 红包、微信、支付宝。

## FAQ

### 1. 要交出 QQ 密码吗？
答：不必须。如果您的说说都是公开的，则完全没有必要交出密码。如果您发过只有自己或者少数好友可见的说说，则需要用您的账号密码登陆才能抓全。

### 2. 我的 QQ 空间被封了，别人都无法访问，可以导出吗？
答：只要您自己能访问就可以，但要您协助登录。比如让您刷一个二维码，（二维码在本人的机器上产生，截图给您，所以您刷完是在本人机器上登录）。极少数时候因为登录保护，刷二维码可能失败，实在不行，要提供您的账号和密码。您可以事先改一个临时密码，事后再改掉。

### 3. 导出的格式是什么样的？
答：主体是一个 json 文件，里面有您全部说说，包括说说本身、别人的回复、图片链接（没有图片本身）。另外生成一份 txt 文件，只有发帖时间和说说本身，其它都没有。

## 备注

如果有时间会改进，比如说搞成图文并茂的格式，也导出博客。