---
layout: post
title: 把 ffmpeg AVAudioFifo/AVFrame 数据读到共享内存
date: 2017-07-20 16:45:38
categories: UMUTech
tags:
- dev
- windows
- streaming-media
- ffmpeg
---
一般情况下操作 AVAudioFifo/AVFrame 都是用全套 ffmpeg API，内部自己管理内存，不需要了解它们内部怎么组织内存。比如：

```cpp
inline int InitFrame(AVFrame *&frame, int frame_size = kTargetSamplesPerFrame)
{
    frame = av_frame_alloc();
    if (nullptr == frame) {
        return AVERROR(ENOMEM);
    }

    frame->nb_samples = frame_size;
    frame->channel_layout = av_get_default_channel_layout(kTargetChannels);
    frame->format = kTargetSampleFormat;
    frame->sample_rate = kTargetSampleRate;

    int error = av_frame_get_buffer(frame, 0);
    if (error < 0) {
        av_frame_free(&frame);
        ATLTRACE2(atlTraceException, 0, "!av_frame_get_buffer(), #%d, %s\n", error, GetAvErrorText(error));
    }
    return error;
}

{
    ...
    AVFrame *frame;
    error_code = InitFrame(frame);
    if (error_code < 0) {
        ATLTRACE2(atlTraceException, 0, __FUNCTION__ ": !InitFrame(), #%d\n", error_code);
        return error_code;
    }
    ON_SCOPE_EXIT([&] {
        av_frame_free(&frame);
    });
    
    int read_size = av_audio_fifo_read(fifo_, (void **)frame->data, kTargetSamplesPerFrame);
    ...
}
```

这里读了一个 AVFrame 出来，并不需要知道具体的内存布局，**但如果要写入 FileMapping 对象里，就得知道了！** 参考以下函数：

```cpp
int av_audio_fifo_read(AVAudioFifo *af, void **data, int nb_samples)
{
    int i, size;

    if (nb_samples < 0)
        return AVERROR(EINVAL);
    nb_samples = FFMIN(nb_samples, af->nb_samples);
    if (!nb_samples)
        return 0;

    size = nb_samples * af->sample_size;
    for (i = 0; i < af->nb_buffers; i++) {
        if (av_fifo_generic_read(af->buf[i], data[i], size, NULL) < 0)
            return AVERROR_BUG;
    }
    af->nb_samples -= nb_samples;

    return nb_samples;
}
```

和 AVFrame 定义：

```c
typedef struct AVFrame {
#define AV_NUM_DATA_POINTERS 8
    /**
     * pointer to the picture/channel planes.
     * This might be different from the first allocated byte
     *
     * Some decoders access areas outside 0,0 - width,height, please
     * see avcodec_align_dimensions2(). Some filters and swscale can read
     * up to 16 bytes beyond the planes, if these filters are to be used,
     * then 16 extra bytes must be allocated.
     *
     * NOTE: Except for hwaccel formats, pointers not needed by the format
     * MUST be set to NULL.
     */
    uint8_t *data[AV_NUM_DATA_POINTERS];

    /**
     * For video, size in bytes of each picture line.
     * For audio, size in bytes of each plane.
     *
     * For audio, only linesize[0] may be set. For planar audio, each channel
     * plane must be the same size.
     *
     * For video the linesizes should be multiples of the CPUs alignment
     * preference, this is 16 or 32 for modern desktop CPUs.
     * Some code requires such alignment other code can be slower without
     * correct alignment, for yet other it makes no difference.
     *
     * @note The linesize may be larger than the size of usable data -- there
     * may be extra padding present for performance reasons.
     */
    int linesize[AV_NUM_DATA_POINTERS];
...
};
```

以 AV_SAMPLE_FMT_S16 为例，发现 InitFrame() 里的 av_frame_get_buffer() 之后只有 linesize[0] 是非 0，即 data[0] 的分配长度，其它 7 个都是 0，即 data[1] -> data[7] 都没有分配，于是猜测就是读 data[0]，长度 linesize[0]，尝试把它写到 FileMapping 里，果然是对的。如果 SampleFormat 是带 P 的，就不是只有 data[0] 了，有几个 channel 就有几个 data，要相应改变。

## 相关书籍

京东联盟购买链接：

[FFmpeg从入门到精通](https://union-click.jd.com/jdc?e=&p=AyIGZRprFQEQAl0eWRIyVlgNRQQlW1dCFFlQCxxKQgFHRE5XDVULR0UVARACXR5ZEh1LQglGaxFVZWEceFlrYkcEKlocdgVSZAtzPFMOHjdQG1oUARUAUxJTJQITBVAZWRYBFDdlG1olVHwHVBpaFAMXBlEYaxcDEwVWE10TAhI3VRxaHQcbB1AYUhUBEzdSG1IlZm5jUhtSJTISBFceUxAAFTdWK2slAiIEZVk1QQpCBgBMWhYFFVJVHl9ACxoDB0hbRQITV1xPDEBVFAZlGVoUBhs%3D) 出版时间：2018-04-01 用纸：胶版纸
