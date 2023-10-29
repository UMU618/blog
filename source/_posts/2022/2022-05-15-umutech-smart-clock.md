---
layout: post
title: 智能时钟
date: 2022-05-15 00:06:18
categories: UMUTech
tags:
- debian
- dev
- linux
---
## 智能时钟是什么？

- 基本功能：时钟、闹钟，并能进行自定义，比如老年人大字体，自定义闹铃等。

- 额外功能：下载、文件共享……

## 为什么需要智能时钟？

- 手机是私人物品，而且小孩子不一定有，家庭还是需要一个时钟的。

- 看时间这事，时钟比手机更有仪式感。比如稣就很怀念小时候的机械时钟，几点就响几下，曾经觉得特别神奇。

- PC 上的迅雷越来越大坨了，而且稣特别怀疑它老在扫描硬盘里价值千万的代码，所以想让它在一个独立的设备运行，那可不就是运行在智能时钟里最合适吗？

- 少量文件共享，特别是看完就删的电影，如果买 NAS，那多贵呀，多费电呀……NAS 显然不适合穷稣，稣只愿意为文件共享付出一张 32G 的 MicroSD 卡。

- 全家可以参与制作，是一个家庭娱乐项目。

- 时钟拥有智能后，您还可以想出更多好玩的！

## 怎么做一个智能时钟？

稣正好有一个屏幕失灵的八英寸的平板电脑，酷比魔方 iWork 8，拿它来做智能时钟刚好合适。

![SmartClock](/images/2022/20220515-clock.jpg)

当然，如果能定制一个，那更好：

- 摄像头都拿掉：后摄像头是贴墙的，肯定没用了，前摄像头也许能想到用途，但稣暂时用不上，所以也把它挡起来。

- HDMI 接口可以拿掉：已经不需要外接显示器。

- 需要屏，但没必要是触摸屏：事实上稣的 iWork 8 就是触摸坏掉，作为时钟并没有啥不便。

- 耳机接口没必要：它只会增大厚度，即使是当作平板电脑用的时候，就从来没插过！

去掉这些东西后，这一台全新的~~平板~~**智能时钟，大约就卖 99 块，吧？反正再贵点，稣就不买。**

稣在 B 站扔了两个劣质视频，笑纳吧（不好笑的话，可以上去吐槽）：

- <https://www.bilibili.com/video/BV1FA4y1S7yA>

- <https://www.bilibili.com/video/BV1DL4y1c7Xw>

甚至还有开源项目：

- <https://github.com/UMU618/sddm-theme-clock>

- <https://gitee.com/umu618/sddm-theme-clock>

## 具体制作过程

### 1. 安装智能时钟操作系统——Debian

为什么？除了因为稣喜欢它，更重要的原因是：原装的 Windows 太大了，没啥剩余空间，而且定制「锁屏界面」真的难！Debian 小很多，也容易定制。其它 Linux 发行版不够爱国，被稣无视了。

注意：选择【不安装】桌面环境！

参考：[在华硕灵耀 X 纵横上装 Debian 桌面的经验](/2022/04/18/umutech-install-debian-on-asus-ux3000e/)

这步会遇到坑——这些 2G 内存的老平板很可能只支持 32bit 的 EFI 启动！

- 要么您就直接装 [32bit 的 Debian](https://cdimage.debian.org/debian-cd/current/i386/iso-cd/debian-11.3.0-i386-netinst.iso)，忍受可能应用不够用的困境；

- 要么您就在 [64bit 的安装盘](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-11.3.0-amd64-netinst.iso)下点功夫。上面的视频就有提到方法。

### 2. 安装驱动

iWork 8:

```sh
sudo apt install firmware-realtek firmware-intel-sound
```

Dell Venue 8 Pro 稍微麻烦点：

```sh
apt install firmware-atheros

git clone https://github.com/UMU618/ath6kl-firmware
sudo cp -r ath6kl-firmware/ath6k/AR6004/hw3.0/ /lib/firmware/ath6k/AR6004/
```

### 3. 配置无线网络

建议采用 NetworkManager 的方式：

```sh
sudo apt install network-manager
```

参考：<https://wiki.debian.org/WiFi/HowToUse#NetworkManager>

话说这里有个矛盾——要是一开始没网络，怎么 `apt`？当然是先插个 USB 有线网卡了……如果没有的话，需要离线把无线网卡驱动准备好，U 盘和 `dpkg` 大法。

### 4. 安装 SDDM 和 i3

```sh
sudo apt install xserver-xorg-video-intel
sudo apt install xserver-xorg-input-evdev xserver-xorg-input-kbd xserver-xorg-input-mouse
sudo apt install sddm i3 sakura fonts-wqy-zenhei
```

为什么是[反人类的 i3](/2022/04/25/umutech-linux-desktop/)？

- 因为它很小，能够获得更多剩余空间！

- 只跑个迅雷您还要什么大型桌面？

- Samba 服务也不要什么桌面呀！

- i3 是反人类，但稣很喜欢 i3 呢……

### 5. 安装智能时钟主题

参见开源项目：

<https://github.com/UMU618/sddm-theme-clock>

<https://gitee.com/umu618/sddm-theme-clock>

没错，这两个链接在本文出现了两次！

### 6. 安装远程桌面服务

时钟挂在墙上，主要的使用方式当然是 SSH 或远程桌面（RDP）过去。使用迅雷这样有界面的程序，最好就是通过远程桌面。

```sh
sudo apt install xrdp
```

### 7. 安装迅雷

```sh
wget http://archive.kylinos.cn/kylin/partner/pool/com.xunlei.download_1.0.0.1_amd64.deb
sudo dpkg -i com.xunlei.download_1.0.0.1_amd64.deb
sudo apt install libgtk2.0-0 libnss3 libdbus-glib-1-2
```

迅雷的启动命令是：`/opt/apps/com.xunlei.download/files/start.sh`

### 8. 安装做共享用的 Samba

```sh
sudo apt install samba
```

来个配置例子：

```sh
# cat /etc/samba/smb.conf
[movie]
   comment = Movie
   browseable = yes
   path = /mnt/sd/movie
   read only = yes
   guest ok = yes
```

### 9. 全家出动录音做闹钟声

直接在平板电脑上用 `arecord` 录有点麻烦，建议在 PC 上录再用 `scp` 或 `rz` 上传，wav 格式的就行。

调节音量可以用：

```sh
sudo apt install alsa-utils
alsamixer
```

然后在 `crontab` 脚本里用 `aplay` 播放。

参考稣家里的：

```sh
root@uclock:/opt/clock# ll
总用量 3228
-rw-r--r-- 1 root root 179300 May 12 22:37 0730.wav
-rw-r--r-- 1 root root 271548 May  5 21:57 0750.wav
-rw-r--r-- 1 root root 452796 May  5 22:03 1345.wav
-rw-r--r-- 1 root root 432312 May  5 21:46 2200.wav
-rwxr-xr-x 1 root root    168 May  6 22:23 alarm.sh
-rwxr-xr-x 1 root root    184 May  6 22:28 chime.sh
-rw-r--r-- 1 root root 208088 May  5 02:57 chime.wav
-rw-r--r-- 1 root root 171206 May  5 22:09 near_0750.wav
-rw-r--r-- 1 root root 429254 May  6 00:21 near_1345.wav
-rw-r--r-- 1 root root 500934 May  6 00:24 near_1345+.wav
-rw-r--r-- 1 root root 624838 May  5 22:12 near_2200.wav
-rwxr-xr-x 1 root root    199 May  6 00:29 near_alarm.sh
-rwxr-xr-x 1 root root     53 May  5 23:21 screen_off.sh
-rwxr-xr-x 1 root root    186 May  6 03:13 screen_on.sh

root@uclock:/opt/clock# crontab -l
0 * * * * /opt/clock/chime.sh
0 2 * * * /opt/clock/screen_off.sh
0 7 * * * /opt/clock/screen_on.sh
30 7 * * 1-5 /opt/clock/alarm.sh
47 7 * * 1-5 /opt/clock/near_alarm.sh 0750
50 7 * * 1-5 /opt/clock/alarm.sh
42 13 * * 1-5 /opt/clock/near_alarm.sh 1345+
43 13 * * 1-5 /opt/clock/near_alarm.sh 1345
45 13 * * 1-5 /opt/clock/alarm.sh
45 13 * * 1-5 /opt/clock/alarm.sh
57 21 * * * /opt/clock/near_alarm.sh 2200
```

alarm.sh 脚本：

```sh
#!/bin/sh

CD="$(cd $(dirname $0) && pwd)"
FILE="${CD}/$(date +%H%M).wav"
if [ -f $FILE ]; then
	/usr/bin/aplay -D plughw:1,0 ${FILE}
else
	echo "$FILE not found!"
fi
```

chime.sh 脚本：

```sh
#!/bin/sh

CD="$(cd $(dirname $0) && pwd)"
/usr/bin/aplay -D plughw:1,0 ${CD}/chime.wav

FILE="${CD}/$(date +%H%M).wav"
if [ -f $FILE ]; then
	/usr/bin/aplay -D plughw:1,0 ${FILE}
fi
```

near_alarm.sh 脚本：

```sh
#!/bin/sh

if [ ${#1} -ge 4 ]; then
	CD="$(cd $(dirname $0) && pwd)"
	FILE="${CD}/near_${1}.wav"
	if [ -f $FILE ]; then
		/usr/bin/aplay -D plughw:1,0 ${FILE}
	fi
else
	echo "Invalid parameter!"
fi
```

### 10. 高级功能

- 旋转屏幕：`xrandr`

- 防止休眠、定时息屏和亮屏：`dpms`

- 调屏幕亮度：`ls /sys/class/backlight`

- 模拟时钟：`xclock`

这就得好好学习 Linux 了……前面只是抛砖引玉，还有许多好玩的哦！

## 后话

折腾旧设备总有意外的收获！比如，Surface RT 在 Windows 下无法识别 5.8GHz WiFi5，在 Raspbian 下却可以。祝大家玩得愉快！
