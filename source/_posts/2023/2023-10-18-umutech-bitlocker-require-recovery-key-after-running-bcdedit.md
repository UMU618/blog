---
layout: post
title: 解决运行 bcdedit 后 BitLocker 提示输入恢复密钥
date: 2023-10-18 00:06:31
description: ChatGPT 说这很险啊！
categories: UMUTech
tags:
- debug
- dev
- ops
- windows
---
## 问题

因为种种原因……需要调试本机内核，用 bcdedit 开启调试模式：

```sh
bcdedit -debug ON
```

结果重启后，BitLocker 提示输入恢复密钥！

## 分析

首先，要强调——调试本机内核，本身就是一个很危险的操作！建议还是用虚拟机调试。

其次，您一定已经备份了 BitLocker 的恢复密钥。只是它不一定在身边，比如说放在家里，人在公司，一来一回需要很长时间，所以得想办法节省时间。

最后，不要被 BitLocker 吓倒！即使，您没有备份恢复密钥，也还有救！在这个界面按 ESC，再点“跳过此驱动器”，后面是可以进入“控制台”的，只需要在“控制台”里撤销操作即可！

## 解决

尝试关闭调试模式：

```sh
bcdedit -debug OFF
```

先别急着重启，因为这么敲——无效！不信您可以重启后再运行 bcdedit，会发现这个 debug 选项还是 Yes。

看来在 WindowsRE 环境下，直接运行 bcdedit，并不能修改 C 盘里的启动选项。您需要加上个 ID，一般为 {default}。

```sh
bcdedit -debug {default} OFF
```

先别急着重启，因为这么敲——还是无效！原本没有 debug 这个值，现在多了一个，数据是 No 而已，因为默认值就是 No，看似没有改变“调试模式”，但其实 BCD 数据库是变了的。正确的做法是删除这个 debug 值：

```sh
bcdedit -deletevalue {default} debug
```

重启后不再要求输入恢复密钥。这时可以在 BitLocker 的控制面板里先暂停保护，然后再操作 BCD，即可。
