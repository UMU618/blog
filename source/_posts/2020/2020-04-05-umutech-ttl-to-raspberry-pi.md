---
layout: post
title: TTL 线连接树莓派
date: 2020-04-05 00:07:48
categories: UMUTech
tags:
- embedded
- windows
---
## 需求

有以下物品：

- PL2303 串口线（TTL 线）

- Windows 10 PC

- 树莓派 Model B

求：PL2303 串口线是好是坏？

## 解决

### 1. 使 Windows 10 正常驱动 PL2303

把 TTL 线插入 PC USB 口，Windows 10 会自动安装驱动，然而  `Prolific USB-to-Serial Comm Port 版本 3.8.31.0 [2019/7/30]` 和  `Prolific USB-to-Serial Comm Port 版本 3.8.18.0 [2017/10/17]` 都不能正常工作。**这说明 PL2303 芯片已经被淘汰……买 TTL 线请选当下流行的其它芯片。**

![Prolific USB-to-Serial Comm Port 版本 3.8.31.0 [2019/7/30] 不能正常工作](/images/2020/20200405-bad-driver-0.png)

![Prolific USB-to-Serial Comm Port 版本 3.8.18.0 [2017/10/17] 不能正常工作](/images/2020/20200405-bad-driver-1.png)

搜“PL2303_Prolific_GPS_1013_20090319”，装“Prolific USB-to-Serial Comm Port 版本 3.3.2.105 [2008/10/27]”，可正常工作。

![Prolific USB-to-Serial Comm Port 版本 3.3.2.105 [2008/10/27] 可正常工作](/images/2020/20200405-driver-version.png)

### 2. 配置串口

波特率设为 115200，其它默认，最终参数如图：

![串口参数](/images/2020/20200405-com1-settings.png)

![高级](/images/2020/20200405-com1-advance.png)

### 3. TTL 线连树莓派 GPIO

如果树莓派有用 MicroUSB 供电，则 VCC（红）可以不接，只把 GND（黑）、RX（白）、TX（绿）分别接到树莓派的 P6、P8、P10；如果需要直接用 GPIO 供电，则把 VCC（红）插到 P2 或 P4。

| PL2303 | 接线颜色 | 树莓派 GPIO | 树莓派针脚 |
| :- | :- | :- | :- |
| VCC | 红 | 5V | P2, P4 |
| RX | 白 | GPIO14(UART_TXD) | P8 |
| TX | 绿 | GPIO15(UART_RXD) | P10 |
| GND | 黑 | GND | P6, P39 |

![GPIO](/images/2020/20200405-gpio.png)

![TTL 接 GPIO：红接 1 或 2，黑接 3，白接 4，绿接 5](/images/2020/20200405-ttl-gpio.jpg)

![树莓派 GPIO 供电](/images/2020/20200405-raspberry-pi.jpg)

### 4. 用 PuTTY 连接串口

使用 plink 或 putty 皆可，注意：需使用管理员权限运行。

```cmd
plink -serial \\.\COM1 -sercfg 115200,8,n,1,n
```

![plink](/images/2020/20200405-plink.png)

![PuTTY](/images/2020/20200405-putty.png)

![COM1 - PuTTY](/images/2020/20200405-com1-putty.png)

## 相关

[在树莓派上编译 go-ipfs](/2020/03/28/umutech-compile-go-ipfs-on-raspberry-pi/)