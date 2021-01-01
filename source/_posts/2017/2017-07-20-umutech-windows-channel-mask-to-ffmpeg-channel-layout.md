---
layout: post
title: Windows 的 ChannelMask 转 ffmpeg 的 ChannelLayout
date: 2017-07-20 16:02:47
categories: UMUTech
tags:
- dev
- windows
- streaming-media
- ffmpeg
---
## 前情

最近写录音程序，发现 MBP 的扬声器是 4 频道的，然后在抓音频保存时，Opus 编码器居然不支持 4 个频道，avcodec_open2() 会返回错误码 -22，Invalid argument。解决方法就是 resample 成 AV_CH_LAYOUT_STEREO。搞定后就顺便细研了这个 ChannelLayout，UMU 的代码里需要把微软 CoreAudio 的一些参数转成 ffmpeg 的，比如之前写的《[Windows 的 WAVEFORMATEX 转 ffmpeg 的 AVSampleFormat 类型](/2017/07/04/umutech-windows-waveformatex-to-ffmpeg-avsampleformat/)》，这次写 ChannelLayout 的转换。

## 分析

ffmpeg 的 channel layouts 定义：

```c
/**
 * @}
 * @defgroup channel_mask_c Audio channel layouts
 * @{
 * */
#define AV_CH_LAYOUT_MONO              (AV_CH_FRONT_CENTER)
#define AV_CH_LAYOUT_STEREO            (AV_CH_FRONT_LEFT|AV_CH_FRONT_RIGHT)
#define AV_CH_LAYOUT_2POINT1           (AV_CH_LAYOUT_STEREO|AV_CH_LOW_FREQUENCY)
#define AV_CH_LAYOUT_2_1               (AV_CH_LAYOUT_STEREO|AV_CH_BACK_CENTER)
#define AV_CH_LAYOUT_SURROUND          (AV_CH_LAYOUT_STEREO|AV_CH_FRONT_CENTER)
#define AV_CH_LAYOUT_3POINT1           (AV_CH_LAYOUT_SURROUND|AV_CH_LOW_FREQUENCY)
#define AV_CH_LAYOUT_4POINT0           (AV_CH_LAYOUT_SURROUND|AV_CH_BACK_CENTER)
#define AV_CH_LAYOUT_4POINT1           (AV_CH_LAYOUT_4POINT0|AV_CH_LOW_FREQUENCY)
#define AV_CH_LAYOUT_2_2               (AV_CH_LAYOUT_STEREO|AV_CH_SIDE_LEFT|AV_CH_SIDE_RIGHT)
#define AV_CH_LAYOUT_QUAD              (AV_CH_LAYOUT_STEREO|AV_CH_BACK_LEFT|AV_CH_BACK_RIGHT)
#define AV_CH_LAYOUT_5POINT0           (AV_CH_LAYOUT_SURROUND|AV_CH_SIDE_LEFT|AV_CH_SIDE_RIGHT)
#define AV_CH_LAYOUT_5POINT1           (AV_CH_LAYOUT_5POINT0|AV_CH_LOW_FREQUENCY)
#define AV_CH_LAYOUT_5POINT0_BACK      (AV_CH_LAYOUT_SURROUND|AV_CH_BACK_LEFT|AV_CH_BACK_RIGHT)
#define AV_CH_LAYOUT_5POINT1_BACK      (AV_CH_LAYOUT_5POINT0_BACK|AV_CH_LOW_FREQUENCY)
#define AV_CH_LAYOUT_6POINT0           (AV_CH_LAYOUT_5POINT0|AV_CH_BACK_CENTER)
#define AV_CH_LAYOUT_6POINT0_FRONT     (AV_CH_LAYOUT_2_2|AV_CH_FRONT_LEFT_OF_CENTER|AV_CH_FRONT_RIGHT_OF_CENTER)
#define AV_CH_LAYOUT_HEXAGONAL         (AV_CH_LAYOUT_5POINT0_BACK|AV_CH_BACK_CENTER)
#define AV_CH_LAYOUT_6POINT1           (AV_CH_LAYOUT_5POINT1|AV_CH_BACK_CENTER)
#define AV_CH_LAYOUT_6POINT1_BACK      (AV_CH_LAYOUT_5POINT1_BACK|AV_CH_BACK_CENTER)
#define AV_CH_LAYOUT_6POINT1_FRONT     (AV_CH_LAYOUT_6POINT0_FRONT|AV_CH_LOW_FREQUENCY)
#define AV_CH_LAYOUT_7POINT0           (AV_CH_LAYOUT_5POINT0|AV_CH_BACK_LEFT|AV_CH_BACK_RIGHT)
#define AV_CH_LAYOUT_7POINT0_FRONT     (AV_CH_LAYOUT_5POINT0|AV_CH_FRONT_LEFT_OF_CENTER|AV_CH_FRONT_RIGHT_OF_CENTER)
#define AV_CH_LAYOUT_7POINT1           (AV_CH_LAYOUT_5POINT1|AV_CH_BACK_LEFT|AV_CH_BACK_RIGHT)
#define AV_CH_LAYOUT_7POINT1_WIDE      (AV_CH_LAYOUT_5POINT1|AV_CH_FRONT_LEFT_OF_CENTER|AV_CH_FRONT_RIGHT_OF_CENTER)
#define AV_CH_LAYOUT_7POINT1_WIDE_BACK (AV_CH_LAYOUT_5POINT1_BACK|AV_CH_FRONT_LEFT_OF_CENTER|AV_CH_FRONT_RIGHT_OF_CENTER)
#define AV_CH_LAYOUT_OCTAGONAL         (AV_CH_LAYOUT_5POINT0|AV_CH_BACK_LEFT|AV_CH_BACK_CENTER|AV_CH_BACK_RIGHT)
#define AV_CH_LAYOUT_HEXADECAGONAL     (AV_CH_LAYOUT_OCTAGONAL|AV_CH_WIDE_LEFT|AV_CH_WIDE_RIGHT|AV_CH_TOP_BACK_LEFT|AV_CH_TOP_BACK_RIGHT|AV_CH_TOP_BACK_CENTER|AV_CH_TOP_FRONT_CENTER|AV_CH_TOP_FRONT_LEFT|AV_CH_TOP_FRONT_RIGHT)
#define AV_CH_LAYOUT_STEREO_DOWNMIX    (AV_CH_STEREO_LEFT|AV_CH_STEREO_RIGHT)
```

ffmpeg 的频道位置信息：

```c
#define AV_CH_FRONT_LEFT             0x00000001
#define AV_CH_FRONT_RIGHT            0x00000002
#define AV_CH_FRONT_CENTER           0x00000004
#define AV_CH_LOW_FREQUENCY          0x00000008
#define AV_CH_BACK_LEFT              0x00000010
#define AV_CH_BACK_RIGHT             0x00000020
#define AV_CH_FRONT_LEFT_OF_CENTER   0x00000040
#define AV_CH_FRONT_RIGHT_OF_CENTER  0x00000080
#define AV_CH_BACK_CENTER            0x00000100
#define AV_CH_SIDE_LEFT              0x00000200
#define AV_CH_SIDE_RIGHT             0x00000400
#define AV_CH_TOP_CENTER             0x00000800
#define AV_CH_TOP_FRONT_LEFT         0x00001000
#define AV_CH_TOP_FRONT_CENTER       0x00002000
#define AV_CH_TOP_FRONT_RIGHT        0x00004000
#define AV_CH_TOP_BACK_LEFT          0x00008000
#define AV_CH_TOP_BACK_CENTER        0x00010000
#define AV_CH_TOP_BACK_RIGHT         0x00020000
#define AV_CH_STEREO_LEFT            0x20000000  ///< Stereo downmix.
#define AV_CH_STEREO_RIGHT           0x40000000  ///< See AV_CH_STEREO_LEFT.
#define AV_CH_WIDE_LEFT              0x0000000080000000ULL
#define AV_CH_WIDE_RIGHT             0x0000000100000000ULL
#define AV_CH_SURROUND_DIRECT_LEFT   0x0000000200000000ULL
#define AV_CH_SURROUND_DIRECT_RIGHT  0x0000000400000000ULL
#define AV_CH_LOW_FREQUENCY_2        0x0000000800000000ULL
```

微软的频道位置信息：

```c
// Speaker Positions for dwChannelMask in WAVEFORMATEXTENSIBLE:
#define SPEAKER_FRONT_LEFT              0x1
#define SPEAKER_FRONT_RIGHT             0x2
#define SPEAKER_FRONT_CENTER            0x4
#define SPEAKER_LOW_FREQUENCY           0x8
#define SPEAKER_BACK_LEFT               0x10
#define SPEAKER_BACK_RIGHT              0x20
#define SPEAKER_FRONT_LEFT_OF_CENTER    0x40
#define SPEAKER_FRONT_RIGHT_OF_CENTER   0x80
#define SPEAKER_BACK_CENTER             0x100
#define SPEAKER_SIDE_LEFT               0x200
#define SPEAKER_SIDE_RIGHT              0x400
#define SPEAKER_TOP_CENTER              0x800
#define SPEAKER_TOP_FRONT_LEFT          0x1000
#define SPEAKER_TOP_FRONT_CENTER        0x2000
#define SPEAKER_TOP_FRONT_RIGHT         0x4000
#define SPEAKER_TOP_BACK_LEFT           0x8000
#define SPEAKER_TOP_BACK_CENTER         0x10000
#define SPEAKER_TOP_BACK_RIGHT          0x20000
```

## 结论

经过对比可以发现两者是一致的，只是微软的 dwChannelMask 是 DWORD，ffmpeg 用的是 int64_t。所以我们可以写个函数来转换他们：

```c
// 微软的 ChannelMask 值转为 ffmpeg 的 channel_layout
inline int64_t GetChannelLayout(const WAVEFORMATEX *wave_format)
{
    if (WAVE_FORMAT_EXTENSIBLE == wave_format->wFormatTag) {
        return reinterpret_cast<const WAVEFORMATEXTENSIBLE *>(wave_format)->dwChannelMask;
    }
    return av_get_default_channel_layout(wave_format->nChannels);
}
```

## 相关书籍

京东联盟购买链接：

[FFmpeg从入门到精通](https://union-click.jd.com/jdc?e=&p=AyIGZRprFQEQAl0eWRIyVlgNRQQlW1dCFFlQCxxKQgFHRE5XDVULR0UVARACXR5ZEh1LQglGaxFVZWEceFlrYkcEKlocdgVSZAtzPFMOHjdQG1oUARUAUxJTJQITBVAZWRYBFDdlG1olVHwHVBpaFAMXBlEYaxcDEwVWE10TAhI3VRxaHQcbB1AYUhUBEzdSG1IlZm5jUhtSJTISBFceUxAAFTdWK2slAiIEZVk1QQpCBgBMWhYFFVJVHl9ACxoDB0hbRQITV1xPDEBVFAZlGVoUBhs%3D) 出版时间：2018-04-01 用纸：胶版纸