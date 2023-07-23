---
layout: post
title: Opus 编解码遇到的怪事
date: 2017-07-01 17:23:52
categories: UMUTech
tags:
- dev
- debug
- streaming-media
- ffmpeg
---
## 前情

最近参考 ffmpeg 的 transcoding_aac 示例代码，写了一个 transcoding_opus，并拿 MP3 测试转码，结果发现转完的 opus 文件的 SampleFormat 和指定的并不一样。UMU 的代码是把源文件解码出来的 sample 先 resample 成 AV_SAMPLE_FMT_S16 格式，然后再交给 opus encoder 去编码的，但是编完用 ffprobe 查看，发现 SampleFormat 变成 AV_SAMPLE_FMT_FLTP。

**那么第一个问题来了，为什么会这样？**

## 分析

开始研究，首先 UMU 把 opus encoder 支持的 sample_fmt 打印出来，发现只有两种：AV_SAMPLE_FMT_S16、AV_SAMPLE_FMT_FLT，压根就没有 AV_SAMPLE_FMT_FLTP，强行指定 AV_SAMPLE_FMT_FLTP 之后，直接报错，不支持这种 sample_fmt。

推测，真的被编码为 AV_SAMPLE_FMT_S16 了，是 ffprobe 的问题，于是自己写了个简化版的 ffprobe，流程几乎是一样的，出来的结果——果然一模一样……打印出 AV_SAMPLE_FMT_FLTP。

接着怀疑 ffprobe 用的 decoder，于是去看了 avcodec_find_decoder 返回的 AVCodec，打印一下 name 和 long_name，和 transcoding_opus 的 avcodec_find_encoder 返回的一比，果然不一样……

选用的编码器是这样的：

```c
AVCodec ff_libopus_encoder = {
    .name            = "libopus",
    .long_name       = NULL_IF_CONFIG_SMALL("libopus Opus"),
    .type            = AVMEDIA_TYPE_AUDIO,
    .id              = AV_CODEC_ID_OPUS,
    .priv_data_size  = sizeof(LibopusEncContext),
    .init            = libopus_encode_init,
    .encode2         = libopus_encode,
    .close           = libopus_encode_close,
    .capabilities    = AV_CODEC_CAP_DELAY | AV_CODEC_CAP_SMALL_LAST_FRAME,
    .sample_fmts     = (const enum AVSampleFormat[]){ AV_SAMPLE_FMT_S16,
                                                      AV_SAMPLE_FMT_FLT,
                                                      AV_SAMPLE_FMT_NONE },
    .supported_samplerates = libopus_sample_rates,
    .priv_class      = &libopus_class,
    .defaults        = libopus_defaults,
};
```

而选用的解码器是这样的：

```c
AVCodec ff_opus_encoder = {
    .name           = "opus",
    .long_name      = NULL_IF_CONFIG_SMALL("Opus"),
    .type           = AVMEDIA_TYPE_AUDIO,
    .id             = AV_CODEC_ID_OPUS,
    .defaults       = opusenc_defaults,
    .priv_class     = &opusenc_class,
    .priv_data_size = sizeof(OpusEncContext),
    .init           = opus_encode_init,
    .encode2        = opus_encode_frame,
    .close          = opus_encode_end,
    .caps_internal  = FF_CODEC_CAP_INIT_THREADSAFE | FF_CODEC_CAP_INIT_CLEANUP,
    .capabilities   = AV_CODEC_CAP_EXPERIMENTAL | AV_CODEC_CAP_SMALL_LAST_FRAME | AV_CODEC_CAP_DELAY,
    .supported_samplerates = (const int []){ 48000, 0 },
    .channel_layouts = (const uint64_t []){ AV_CH_LAYOUT_MONO,
                                            AV_CH_LAYOUT_STEREO, 0 },
    .sample_fmts    = (const enum AVSampleFormat[]){ AV_SAMPLE_FMT_FLTP,
                                                     AV_SAMPLE_FMT_NONE },
};
```

问题清楚了，看来用 ID 查找编解码器并不靠谱，因为这个 ID 是 Type ID，不是 Item ID，还是改为用 name 来找：

```c
//AVCodec *output_codec = avcodec_find_encoder(AV_CODEC_ID_OPUS);
AVCodec *output_codec = avcodec_find_encoder_by_name("opus");
```

**那么，第二个问题顺势而来——哪个比较牛？**

## 结论

用 AV_SAMPLE_FMT_FLTP 后 frame_size 是 120，用其它是 960，frame_size 小有小的好处，比如在做实时编码直播时，理论延迟会更小。

经过测试，用 AV_SAMPLE_FMT_FLTP 的 opus 比 libopus 压缩率普遍略高一些，但它只支持 48000Hz 一种 sample_rate，libopus 支持的更多：48000, 24000, 16000, 12000, 8000。

## 相关书籍

京东联盟购买链接：

[FFmpeg从入门到精通](https://union-click.jd.com/jdc?e=&p=AyIGZRprFQEQAl0eWRIyVlgNRQQlW1dCFFlQCxxKQgFHRE5XDVULR0UVARACXR5ZEh1LQglGaxFVZWEceFlrYkcEKlocdgVSZAtzPFMOHjdQG1oUARUAUxJTJQITBVAZWRYBFDdlG1olVHwHVBpaFAMXBlEYaxcDEwVWE10TAhI3VRxaHQcbB1AYUhUBEzdSG1IlZm5jUhtSJTISBFceUxAAFTdWK2slAiIEZVk1QQpCBgBMWhYFFVJVHl9ACxoDB0hbRQITV1xPDEBVFAZlGVoUBhs%3D) 出版时间：2018-04-01 用纸：胶版纸
