---
layout: post
title: mif2png（QQGame 专用 mif 格式转 png 格式）
date: 2011-11-26 00:27:00
categories: UMUTech
tags:
- dev
- dotnet
---
2011-11-26 00:27 发布于百度空间，由于百度空间停运，搬到此处。

大学时代的作品《UMU 游戏之争上游》的副产品，mif2bmp 改进版，用 GdiPlus 来产生 png 格式图片。

mif2png.exe 下载：<http://download.csdn.net/detail/umu/3843545>

以前 UMU 有写过文章分析 mif 格式，不过很早了，懒得找，直接上代码吧，先看头部结构体：

```cpp
#pragma pack(1)
struct MifHeader
{
    DWORD version;
    DWORD width;
    DWORD height;
    DWORD type;
    DWORD frame_count;
};
#pragma pack()
```

以下代码是 C# 写的 Paint.NET 文件类型插件 MifFileType.cs

```csharp
using System;
using System.Collections.Generic;
using System.Text;
using PaintDotNet;
using PaintDotNet.Data;
using System.IO;
using System.Drawing;
using System.Windows.Forms;
using System.Drawing.Imaging;

namespace MifFileType
{
	public class MifFileType : FileType
	{
		public MifFileType()
			: base("MIF Files", FileTypeFlags.SupportsLoading | FileTypeFlags.SupportsLayers, new String[] { ".mif" })
		{
		}

		protected override Document OnLoad(Stream input)
		{
			if (input.Length < 20)
			{
				MessageBox.Show("Invalid MIF File", "UMU Corporation - MifFileTypePlugIn", MessageBoxButtons.OK, MessageBoxIcon.Error);

				Bitmap b = new Bitmap(800, 600);
				return Document.FromImage(b);
			}

			try
			{
				BinaryReader br = new BinaryReader(input);

				int MifVersion = br.ReadInt32();
				int FrameWidth = br.ReadInt32();
				int FrameHeight = br.ReadInt32();
				int MifType = br.ReadInt32();
				int FrameCount = br.ReadInt32();

				int ImageWidth = FrameWidth;
				int ImageHeight = FrameHeight * FrameCount;

				bool Valid = true;
				long Prefix;

				if (MifType == 3)
				{
					Prefix = 20;
				}
				else if (MifType == 7)
				{
					Prefix = 20 + 4 * FrameCount;
				}
				else
				{
					MessageBox.Show("Invalid MIF File", "UMU Corporation - MifFileTypePlugIn", MessageBoxButtons.OK, MessageBoxIcon.Error);

					Bitmap b = new Bitmap(800, 600);
					return Document.FromImage(b);
				}

				if (MifVersion == 0)
				{
					if (Prefix + ImageWidth * ImageHeight * 3 != input.Length)
					{
						Valid = false;
					}
				}
				else if (MifVersion == 1)
				{
					if (Prefix + ImageWidth * ImageHeight * 3 > input.Length)
					{
						Valid = false;
					}
				}

				if (!Valid)
				{
					MessageBox.Show("Invalid MIF File", "UMU Corporation - MifFileTypePlugIn", MessageBoxButtons.OK, MessageBoxIcon.Error);

					Bitmap b = new Bitmap(800, 600);
					return Document.FromImage(b);
				}

				Bitmap bmp = new Bitmap(ImageWidth, ImageHeight);

				for (int CurrentFrame = 0; CurrentFrame < FrameCount; ++CurrentFrame)
				{
					if (MifType == 7)
					{
						input.Seek(4, SeekOrigin.Current);
					}

					UInt16[] rgb16 = new UInt16[FrameWidth * FrameHeight];

					for (int i = 0; i < FrameHeight * FrameWidth; ++i)
					{
						rgb16[i] = br.ReadUInt16();
					}

					Byte[] a8 = new Byte[FrameWidth * FrameHeight];

					for (int i = 0; i < FrameHeight * FrameWidth; ++i)
					{
						a8[i] = br.ReadByte();
					}

					for (int y = 0; y < FrameHeight; ++y)
					{
						for (int x = 0; x < FrameWidth; ++x)
						{
							int a = a8[x + y * FrameWidth];
							int r = (rgb16[x + y * FrameWidth] & 0xF800) >> 8;
							int g = (rgb16[x + y * FrameWidth] & 0x07E0) >> 3;
							int b = (rgb16[x + y * FrameWidth] & 0x001F) << 3;

							if (a == 32)
							{
								a = 255;
							}
							else if (a > 0)
							{
								a <<= 3;
							}

							bmp.SetPixel(x, FrameHeight * CurrentFrame + y, Color.FromArgb(a, r, g, b));
						}
					}
				}

				br.Close();
				return Document.FromImage(bmp);
			}
			catch (Exception ex)
			{
				MessageBox.Show(ex.Message, "UMU Corporation - MifFileTypePlugIn", MessageBoxButtons.OK, MessageBoxIcon.Warning);

				Bitmap bmp = new Bitmap(800, 600);
				//Document doc = Document.FromImage(bmp);
				//doc.Tag = "UMU Corporation - MifFileTypePlugIn";
				//return doc;
				return Document.FromImage(bmp);
			}
		}
	}

	public class MifFileTypeFactory : IFileTypeFactory
	{
		public FileType[] GetFileTypeInstances()
		{
			return new FileType[] { new MifFileType() };
		}
	}
}
```
