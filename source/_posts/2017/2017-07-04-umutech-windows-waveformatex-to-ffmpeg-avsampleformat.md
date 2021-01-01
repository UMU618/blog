---
layout: post
title: Windows 的 ChannelMask 转 ffmpeg 的 ChannelLayout
date: 2017-07-04 20:46:31
categories: UMUTech
tags:
- dev
- windows
- cpp
- streaming-media
- ffmpeg
---
## 需求

Windows 平台，录音。

## 任务

用 Windows 的 IAudioCaptureClient 对象采集音频，然后用 ffmpeg 编码。

## 困难

一些类型定义不一样，比如 SampleFormat。

## 解决方案

```cpp
inline AVSampleFormat GetSampleFormat(const WAVEFORMATEX *wave_format)
{
    switch (wave_format->wFormatTag) {
    case WAVE_FORMAT_PCM:
        if (16 == wave_format->wBitsPerSample) {
            return AV_SAMPLE_FMT_S16;
        }
        if (32 == wave_format->wBitsPerSample) {
            return AV_SAMPLE_FMT_S32;
        }
        break;
    case WAVE_FORMAT_IEEE_FLOAT:
        return AV_SAMPLE_FMT_FLT;
    case WAVE_FORMAT_ALAW:
    case WAVE_FORMAT_MULAW:
        return AV_SAMPLE_FMT_U8;
    case WAVE_FORMAT_EXTENSIBLE:
    {
        const WAVEFORMATEXTENSIBLE *wfe = reinterpret_cast<const WAVEFORMATEXTENSIBLE *>(wave_format);
        if (KSDATAFORMAT_SUBTYPE_IEEE_FLOAT == wfe->SubFormat) {
            return AV_SAMPLE_FMT_FLT;
        }
        if (KSDATAFORMAT_SUBTYPE_PCM == wfe->SubFormat) {
            if (16 == wave_format->wBitsPerSample) {
                return AV_SAMPLE_FMT_S16;
            }
            if (32 == wave_format->wBitsPerSample) {
                return AV_SAMPLE_FMT_S32;
            }
        }
        break;
    }
    default:
        break;
    }
    return AV_SAMPLE_FMT_NONE;
}
```
## 相关书籍

京东联盟购买链接：

[FFmpeg从入门到精通](https://union-click.jd.com/jdc?e=&p=AyIGZRprFQEQAl0eWRIyVlgNRQQlW1dCFFlQCxxKQgFHRE5XDVULR0UVARACXR5ZEh1LQglGaxFVZWEceFlrYkcEKlocdgVSZAtzPFMOHjdQG1oUARUAUxJTJQITBVAZWRYBFDdlG1olVHwHVBpaFAMXBlEYaxcDEwVWE10TAhI3VRxaHQcbB1AYUhUBEzdSG1IlZm5jUhtSJTISBFceUxAAFTdWK2slAiIEZVk1QQpCBgBMWhYFFVJVHl9ACxoDB0hbRQITV1xPDEBVFAZlGVoUBhs%3D) 出版时间：2018-04-01 用纸：胶版纸