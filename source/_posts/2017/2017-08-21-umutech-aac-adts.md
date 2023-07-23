---
layout: post
title: AAC 编码之 ADTS 头相关分析
date: 2017-08-21 15:15:00
categories: UMUTech
tags:
- debug
- dev
- ffmpeg
- streaming-media
---
之前在《[Opus 编解码遇到的怪事](/2017/07/01/umutech-opus-debugging/)》说过一个因为编码器不同而导致的怪事的解决过程，最近又出现一例类似情况了。

UMU 的任务是把从麦克风采集到的音频数据，直接编码成 AAC，然后用 live555 流化为 RTSP 协议，做服务端。其中涉及到一个 ADTS 头部的问题，理论上有没有 ADTS 都是可以的，各有可行的解决方案。但在阅读其他同事代码的时候，惊讶地发现，他特地把 ADTS 头给去掉了。而 UMU 调试时，发现 AVPacket 的数据里根本没有 ADTS 头，何来去掉之说？

有了上次的经验，UMU 很快推测，我们俩用的编码器可能不同。后来验证，确实如此：ffmpeg 3.1 有两个 AAC 编码器，一个内置的，名字是 aac，另一个第三方的 libfdk_aac，商业使用 non-free。（以前还有其它两个第三方的，因为质量不行，已经被移除，ffmpeg 官网上有说明）默认的编译方式只有前者，后者需要使用 non-free 参数编译，基于后期的版权问题考虑，UMU 使用的是内置的 aac。但为了调查这个问题，UMU 特地编译并使用了 libfdk_aac，发现确实有不同。

1. aac 编码出来的 AVPacket 是没有 ADTS 头的； libfdk_aac 则有。

2. aac 不需要设置 profile，因为它默认使用 LC，而 libfdk_aac 支持很多中 profile，所以需要设置一个合适的。

3. libfdk_aac 设置合适的 profile 字段，编码出来的 AVPacket 有 ADTS 头，VLC 可以播放，特地去掉 ADTS 头，VLC 也可以播放。

4. 如果不设置 profile，默认是 FF_PROFILE_UNKNOWN，这时有 ADTS 头，但由于这个 ADTS 头里的 adts_buffer_fullness 不对，所以 VLC 无法播放，去掉反而可以。
