---
layout: post
title: 云游戏开发指南【1】
date: 2019-04-14 22:27:45
categories: UMUTech
tags:
- 作品
- 调研
- streaming-media
---
## 实用参考

1. [云游戏的架构设计和技术实现](https://blog.csdn.net/RA681t58CJxsgCkJ31/article/details/79226317)

    <https://blog.csdn.net/RA681t58CJxsgCkJ31/article/details/79226317>

2. [GamingAnywhere](http://www.gaminganywhere.org)

    <http://www.gaminganywhere.org>

3. [OBS Studio](https://github.com/obsproject/obs-studio)

    <https://github.com/obsproject/obs-studio>

## 服务器架构

- NVIDIA Grid 显卡

- Windows 8 - 10，最好别用 7。

- Sandbox 方案

一份授权 Windows 系统大约可以运行 40~50 个游戏实例。

## 音频

- 服务器上设置 48000Hz 采样率；

- 编码器采用 Opus，48000Hz，128kpbs

    1. 音频编码的延迟：需要一个 frame 才能编码，而等待一个 frame 需要时间。比如，48000Hz 的 samplerate 编码到 1024 samples/frame 的 AAC，一个 frame 需要的采集时间是 1000 * 1024 / 48000 = 21.3ms，这个只是采集的理论时间，应用程序的采集间隔还可能增大这个采集延迟，然后还有编码、传输、解码、播放延迟。

        Opus 的 frame size 默认是 960，相比 1024 的 AAC 理论上可以减少一点点延迟，1000 * 960 / 48000 = 20ms。

        Opus 的 frame size 还可以更小，比如 120，但不建议使用，因为 Windows 一次采集 480 samples，frame_size = 120 时，一次采集包含 4 frames，所以延迟和 frame_size = 480 其实是一样的。另外，[live555][live555] 最小发送单位是 packet，而不是 frame，并且实现上 packet 要等待 960 samples 时长。

        128kpbs 的 Opus 音质几乎已经顶天了，人耳很难分辨，比 mp3 好太多了，网络上流行的高音质 mp3 都是 320kpbs 的。

    2. Opus 是开源并免费的，AAC 好的编解码器都是收费的。

## 流传输协议

- RTSP

- [live555][live555]

    <http://www.live555.com/>

    UMU 之前用 live555 2017.07.18 版本时，发现它对 Opus 支持不好，需要改进，当前版本未测。

[live555]: http://www.live555.com/
