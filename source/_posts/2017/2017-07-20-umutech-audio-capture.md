---
layout: post
title: 云录音
date: 2017-07-20 17:42:00
categories: UMUTech
tags:
- dev
- windows
- 调研
- streaming-media
- ffmpeg
- 作品
---
在云游戏和云桌面项目中，总结了几类声音采集技术，把录音做到极致。

## 从外设录音

最典型的就是麦克风，内置麦克风、外置麦克风，其实还有一种通过 LineIn 插入的其它播放器设备，比如 CD、DVD 等。

采集这种音频的方法可以只用 ffmpeg 搞定：av_find_input_format("dshow")...，也可以用 CoreAudio 搞定：


```
enumerator->EnumAudioEndpoints(eCapture, DEVICE_STATE_ACTIVE, ...
audio_client->Initialize(AUDCLNT_SHAREMODE_SHARED, AUDCLNT_STREAMFLAGS_EVENTCALLBACK ...
```

## 从播放设备回放录音

采集方式是用 CoreAudio：

```
enumerator->(eRender, DEVICE_STATE_ACTIVE, ...
audio_client->Initialize(AUDCLNT_SHAREMODE_SHARED, AUDCLNT_STREAMFLAGS_LOOPBACK, ...
```

这种方式会混音，比如说您开个 foobar 播歌，再开个 QQ 影音看电影，则会录到这两个应用程序的混音，嗯，如果 QQ 再嘀嘀嘀，也是会混进去的……

## 虚拟声卡采集

有个叫 Virtual Audio Cable 的虚拟声卡，能虚拟多张声卡，并且可以把声音转发到对应的虚拟 LineIn 设备，供应用程序采集。

## 只录制某个应用程序的音

比前一种更先进一些，多个播放器同时播歌，我们可以只录其中一个。

采集方法是：Hook CoreAudio。

另一个思路是：Hook 到这个应用，给它单独指定一个输出设备，其它应用不能用，否则还是混音了，然后用前面的回放录音技术录制这个独占的输出设备。您可能要说，哪有那么多输出设备？这个问题可以用前面提到的虚拟声卡解决，分分秒虚拟出 64 个是没问题的。而且用 VAC 的好处是，可以在这 64 个对应的 LineIn 通道直接录制，不需要用 CoreAudio，兼容性会更好。
