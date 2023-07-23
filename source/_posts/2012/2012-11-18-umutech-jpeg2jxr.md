---
layout: post
title: 图像格式转换之 Jpeg2Jxr
date: 2012-11-18 15:40:25
categories: UMUTech
tags:
- dev
- windows
- dotnet
---
为什么转？因为 JXR 格式在同等质量的情况下，存储空间比 JPEG 节约了 45-50%。

之前在《[从 Windows 8 新功能推理某产品的八哥](/2012/10/13/umutech-mato-orientation-bug/)》提到过现在手机上的省流量 App，其原理就是压缩图片，但为了提高效果，这个压缩基本都是有损的，流量减少了，但是图片质量下降了，有的下降可以忍受，有的则令人发指！比如，长微博，文字转图片，这种图片线条分明，相邻像素值对比可能很大（黑白分明），这类图片采用高压缩比的 JPEG 压缩后，图片质量往往很差。

再举个例子：QR 码图片，您可以做一下试验，为了说明 JPEG 不适合存储线条型图片，哥采用一张蛋疼的 1290*1290 像素的 QR 码图片，保存为 JPEG 大小是 4.76MB，但保存为 PNG 格式时只有 52.4KB，请注意单位，前者是后者大小的将近 100 倍！！

大家可能比较少关注 WP，也许您没听过 DataSense，简单地说，它就是微软做的节省流量的 App。号称可以节约 45% 的流量，这么大的压缩率，除了优化 HTML 相关的文本之外，对图片的压缩肯定是必须的！推测 DataSense 可能使用了 JPEG XR 格式来转化其他格式的图片。

JPEG XR 虽然已经成为一种标准，但目前依然只有微软支持，所以，如果您想把这个技术应用到 iOS、Android 的节省流量 App 中，那很抱歉，此路暂时还不通。

根据实测，IE9@PC、IE10@PC、IE10@WP8 都是支持 JXR 格式的。下面是用 C++/CLI 写的很简单的一个格式转化程序：

```cpp
using namespace System;
using namespace System::IO;
using namespace System::Windows::Media;
using namespace System::Windows::Media::Imaging;

bool ConvertToJxr(System::String^ source_name)
{
    //try {
        Stream^ stream = gcnew FileStream(source_name, FileMode::Open, FileAccess::Read, FileShare::Read);
        BitmapDecoder^ jpeg_decoder =  BitmapDecoder::Create(stream, BitmapCreateOptions::PreservePixelFormat, BitmapCacheOption::None);
        //JpegBitmapDecoder^ jpeg_decoder = gcnew JpegBitmapDecoder(gcnew Uri(source_name, UriKind::RelativeOrAbsolute), BitmapCreateOptions::PreservePixelFormat, BitmapCacheOption::None);
        //Console::WriteLine(L"Author: `{0}'", jpeg_decoder->Metadata->Title);
        FileStream^ jxr_file_stream = gcnew FileStream(source_name + L".jxr", FileMode::Create);
        WmpBitmapEncoder^ jxr_encoder = gcnew WmpBitmapEncoder;
        //BitmapMetadata^ metadata = gcnew BitmapMetadata(L"wmphoto");
        
        for each (BitmapFrame ^ frame in jpeg_decoder->Frames) {
            jxr_encoder->Frames->Add(BitmapFrame::Create(frame, jpeg_decoder->Thumbnail, (BitmapMetadata^)frame->Metadata, jpeg_decoder->ColorContexts));
        }
        //jxr_encoder->Metadata = metadata;
        jxr_encoder->Save(jxr_file_stream);
    //} catch (...) {
    //    return false;
    //}
    return true;
}

int main(array<System::String ^> ^args)
{
    for each (auto arg in args) {
        if (File::Exists(arg)) {
            if (ConvertToJxr(arg)) {
                Console::WriteLine(L"Converted: `{0}'", arg);
            }
        } else {
            Console::WriteLine(L"NOT Exists: `{0}'", arg);
        }
    }
    return 0;
}
```

文末是一些搜索到的关于 JPEG XR 的资料，可供参考： 

<http://jpeg.org/newsrel26.html>

> JPEG XR (ISO/IEC 29199-2) is now an International Standard and also an ITU-T Recommendation (T.832).
>
> JPEG XR（旧称 HD Photo 及 Windows Media Photo）是一种连续色调静止图像压缩算法和文件格式，由Microsoft开发，属于Windows Media家族的一部分。它支持有损数据压缩以及无损数据压缩，并且是微软的XPS文档的首选图像格式。目前支持的软件包括.NET Framework（3.0 or newer），Windows Vista/Windows 7、Internet Explorer 9，Flashplayer 11等。
>
> JPEG XR（微软HD Photo格式）2009 年，成为 ITU-T 推荐的国际标准（ISO/IEC 29199-2）。JPEG XR 的标准化确保数码相机、打印机、显示器和软件公司能够在开发其新产品的时候兼容互通。其核心技术由微软核心媒体开发团队开发完成，针对当前和将来的数字图像发展需求以提供了许多新的优势和特点。
>
> 在 Vista 操作系统中已经支持了这种新的文件格式，JPEG XR 相比其它技术更有优势，其中包括更好的压缩技术，以一半的文件大小保存与 JPEG 相同质量的图像，或以相同大小的文件保存质量相当于 JPEG 两倍的图像。JPEG 组织还对微软开放与 JPEG XR 相关的专利的决策表示了赞扬，称微软免许可费政策将有助于JPEG推动 JPEG XR 普及，有助于确保它能够被更多的用户所采用。JPEG 组织还鼓励其它公司向微软学习。
