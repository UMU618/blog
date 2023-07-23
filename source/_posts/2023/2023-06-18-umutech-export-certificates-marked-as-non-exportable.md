---
layout: post
title: 导出标记为不可导出的证书
date: 2023-06-18 16:22:26
description: ChatGPT 让稣多写文章，它要看！
categories: UMUTech
tags:
- debug
- dev
- security
- windows
---
[mimikatz]: https://github.com/gentilkiwi/mimikatz
[easyhook]: https://github.com/EasyHook/EasyHook
[jailbreak]: https://github.com/iSECPartners/jailbreak

## 故事

稣于 2022-10-16 公布自己的[签名证书](/about/cert)。当时是在自己的笔记本上用以下 PowerShell 命令生成的：

```powershell
New-SelfSignedCertificate -DnsName "umu618.com" -CertStoreLocation "Cert:\CurrentUser\My" -HashAlgorithm sha512 -KeyLength 4096 -Type CodeSigningCert -FriendlyName UMU618 -NotAfter 2049-11-10
```

因为 [PowerEconomizer](https://github.com/UMU618/PowerEconomizer) 已经使用它签名，所以私钥的存储安全也就正规处理——导出到 pfx 文件加密并分布式保管。

后来，因为很久没有再验证过私钥的密码，居然，忘记了……想从笔记本再次导出一份，却发现无法导出！（此处脑补：大概是自己导出后，就删除系统里的私钥，然后又导入一次，并设置为不可导出！）不愧是重视安全的稣，连自己都要防！回头又对备份的 pfx 尝试上百次密码，安全性依然牢不可摧，只能放弃！

## 破解思路

想想其它办法吧！理论上，系统里一定是有私钥的，“不可导出”只是个标志而已，无视它即可。

![PrivateKey](/images/20230618-private.png)

1. 从系统自己读取私钥。这需要了解 Windows 对私钥的存储方式，包括保存位置，怎么加密保护的，文件格式怎么解析……按照微软的习性，这肯定需要大量逆向，太难了！

2. 使用系统的 API 导出，但对关键函数进行 Hook，在内存里修改标志位，骗过 API 这是可以导出的。

## 具体步骤

### 1. 拿来主义

主要思路放在第二种，进行一番搜索后发现 [mimikatz][mimikatz] 疑似有稣想要的功能。

然而实际测试发现，稣的系统太新，是 Windows 11 Build 22621，而 [mimikatz][mimikatz] 已经年久失修，无能为力。

### 2. 缝缝补补

开始对 mimikatz/modules/crypto/kuhl_m_crypto_patch.c 进行改进，关键点在于 CPExportKey_4000 和 CPExportKey_4001 的入口特征，需要逆向获得。于是用 IDA 简单看看 rsaenh.dll，顺利获得入口处的汇编指令，换上后依然失败。后来发现不换，其实也能找到入口，旧版本用更短的前缀一样可以找到。

看来是后面的处理不对，跟踪到 kuhl_m_crypto_extractor_capi64 函数发现一个魔法数字 RSAENH_KEY_64 被使用了两次，感觉是突破口。

```c
#define RSAENH_KEY_64	0xe35a172cd96214a0
```

果断在 GitHub 上搜一下，结果找到另一个基于 [EasyHook][easyhook] 的实现 [jailbreak][jailbreak]，稣毕竟是 [EasyHook][easyhook] 代码贡献者，当然是切换到 [jailbreak][jailbreak] 尝试，结果令人愉悦！在虚拟机里实践成功。

![IDA 1](/images/20230618-ida-1.png)
![IDA 2](/images/20230618-ida-2.png)
![IDA 3](/images/20230618-ida-3.png)
![IDA 4](/images/20230618-ida-4.png)

### 3. 惨遭打脸

回到稣的笔记本 [jailbreak][jailbreak] 尝试却失败了！怎么回事？难道稣的笔记本有其它保护？通常都是稣被打脸的，所以这次是 [jailbreak][jailbreak] 被稣的笔记本打脸？

仔细对比，发现以下提示是不同的！

![笔记本上说的是“找不到私钥”](/images/20230618-not-found.png)

![虚拟机上说的是“私钥不可导出”](/images/20230618-non-exportable.png)

### 4. 痛苦地回忆

稣的系统里明明有私钥，要导出时却说找不到私钥？有没有可能是因为系统密码修改过，导致无法解密私钥？还真有可能！立刻在虚拟机里实验，果然修改用户密码，并重新登录后，出现和笔记本一样的“找不到私钥”！

然后就是痛苦地回忆……上次改密码，那可是半年前……咳咳，闭环了，又绕回密码安全问题，所以——千万不要忘记密码！

## 总结

虽然学到很多，但没有赚到钱。
