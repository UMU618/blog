---
layout: post
title: 华为擎云 L410
date: 2022-06-18 19:01:56
categories: UMUTech
tags:
- debian
- debug
- linux
- ops
---
## 用户故事

上个月写的[《用华为擎云 L420 体验国产操作系统（UOS 和银河麒麟）》](/2022/05/21/umutech-huawei-qingyun-l420-uos-kylinos/)导致本博客流量暴增，从默默无闻到有人访问，实在太荣幸了……

其实当时稣几乎是同时买 L410 和 L420 的，但由于 L410 的 UFS 是板载的，所以没怎么折腾，而是拿来日常使用，所以反而没写它。现在就来补一补。

先说结论：如果要买量产版的 L410 或 L420，建议后者。

1. 肉眼可感知 L420 比 L410 快；

2. L410 板载 UFS，而 L420 可更换，所以 L410 坏了更不好办；

3. L420 做工更好，尤其是触摸板可以明显感知。（这点可能是个例）

补充：某鱼兼某论坛大佬说，L410 和 L420 触摸板是一样的。但稣手里的 L410 触控板确实八哥比较大，按下去比较松垮，所以这里定义为个例，仅供参考。

如果是买便宜很多的工程机，那务必小心咨询 BIOS 和 EC（Firmware）版本，太低的很可能无法升级，就只能一直忍受 bug 状态。

## 硬件环境

华为擎云 L410 工程机，型号是 KLVU-WDU0A。

- BIOS Revision: 1.30
- Firmware Revision: 1.78
- hisi Version: 2.0.0.17

由于是打算日常使用的，特地选择 12G 内存+512G 存储的版本。外观比 L420 新，除了 A 面有不明显的小划痕外，没有其它问题。价格也来到惊人的 2000 人民币，真贵……

## 软件环境

到手时，是个根本不能用的 UOS，要啥啥没有，比如：

- 指纹解锁——没有！

- 外接显示器——没有任何反应！

- 播音乐——wma 无法播放，进度在走，就是没声音……mp4 倒是可以，莫名其妙。

稣自己装上银河麒麟试用版 Desktop-V10-SP1-kirin990-Release-20211228，几乎是可以日常使用，然而只是几乎！八哥如下：

- 如果没有在开机前就接着电源，那就无法充电。没错，就是开机后，中途想充电，没门！

- 睡眠或合上盖子屏幕关闭后，屏幕就再也无法亮起，只能重启恢复。不过这时候系统还是正常运行的，外接显示器可以正常使用。由于稣一般都是外接 4K 显示器使用，所以这点倒不是很致命。（多吐槽一句：稣的 L420 至今无法外接显示器！）

- 麒麟自带的固件升级工具无法使用，一运行就卡着，log 里大量反复的错误。

```sh
tail -3 /var/log/hwupdate/checkapp.log
[2022-06-18 19:39:13 379]INFO Entery timer
[2022-06-18 19:39:13 417]INFO recv data size is null or fail
[2022-06-18 19:39:13 417]INFO Entry IPCMessageClient::SendData
```

- 移动应用无法使用，因为 KMRE 启动不起来。不过稣也用不上这货……毕竟稣有小米平板 5 破落。

## 总结

这台 L410 的 BIOS 版本不算低，所以八哥没 BIOS 版本过低的 L420 多。

编译 C++ 代码还是挺好用的。尝试编译了一个 gcc13，速度感人，秒杀公司发的联想 L490。而且 512G 版本可以 Clone 好多仓库，所以稣认为性比价还行。