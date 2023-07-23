---
layout: post
title: macOS 研究经验【1】：关于 APFS Container
date: 2020-03-15 23:25:53
categories: UMUTech
tags:
- debug
- macos
- ops
---
## 起因

最近 MBP15 的硬盘空间告急，打开“磁盘工具”查看，却发现居然有两个“宗卷”！从没认真研究过 macOS 磁盘管理的稣疑惑了。

“磁盘工具”默认“仅显示宗卷”，“显示所有设备”后是这样的：

![disk0](/images/20200315-0.png)

![disk1](/images/20200315-1.png)

![disk1s5](/images/20200315-2.png)

![disk1s1](/images/20200315-3.png)

## 分析

肉眼观测 disk1 是放在 disk0 里的……嗯，想起容器！但 GUI 有时候是会骗人的，用 diskutil 来检查一下：

```
$ diskutil list
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *251.0 GB   disk0
   1:                        EFI EFI                     314.6 MB   disk0s1
   2:                 Apple_APFS Container disk1         250.7 GB   disk0s2

/dev/disk1 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +250.7 GB   disk1
                                 Physical Store disk0s2
   1:                APFS Volume Macintosh HD - 数据     209.2 GB   disk1s1
   2:                APFS Volume Preboot                 79.9 MB    disk1s2
   3:                APFS Volume Recovery                526.6 MB   disk1s3
   4:                APFS Volume VM                      8.6 GB     disk1s4
   5:                APFS Volume Macintosh HD            11.2 GB    disk1s5
```

以上可知，只有 disk0 是物理的，disk1 是由 disk0s2 这个分区虚拟出来的。

下面查看两者详细信息对比：

```
$ diskutil info /dev/disk0
   Device Identifier:         disk0
   Device Node:               /dev/disk0
   Whole:                     Yes
   Part of Whole:             disk0
   Device / Media Name:       APPLE SSD AP0256M

   Volume Name:               Not applicable (no file system)
   Mounted:                   Not applicable (no file system)
   File System:               None

   Content (IOContent):       GUID_partition_scheme
   OS Can Be Installed:       No
   Media Type:                Generic
   Protocol:                  PCI-Express
   SMART Status:              Verified

   Disk Size:                 251.0 GB (251000193024 Bytes) (exactly 490234752 512-Byte-Units)
   Device Block Size:         4096 Bytes

   Read-Only Media:           No
   Read-Only Volume:          Not applicable (no file system)

   Device Location:           Internal
   Removable Media:           Fixed

   Solid State:               Yes
   Virtual:                   No
   Hardware AES Support:      Yes

$ diskutil info /dev/disk1
   Device Identifier:         disk1
   Device Node:               /dev/disk1
   Whole:                     Yes
   Part of Whole:             disk1
   Device / Media Name:       APPLE SSD AP0256M

   Volume Name:               Not applicable (no file system)
   Mounted:                   Not applicable (no file system)
   File System:               None

   Content (IOContent):       EF57347C-0000-11AA-AA11-00306543ECAC
   OS Can Be Installed:       No
   Media Type:                Generic
   Protocol:                  PCI-Express
   SMART Status:              Verified
   Disk / Partition UUID:     1DDCC569-2632-4CD5-88E7-66E2BBE745C9

   Disk Size:                 250.7 GB (250685575168 Bytes) (exactly 489620264 512-Byte-Units)
   Device Block Size:         4096 Bytes

   Read-Only Media:           No
   Read-Only Volume:          Not applicable (no file system)

   Device Location:           Internal
   Removable Media:           Fixed

   Solid State:               Yes
   Virtual:                   Yes
   Hardware AES Support:      Yes

   This disk is an APFS Container.  APFS Information:
   APFS Physical Store:       disk0s2
   Fusion Drive:              No
```

两者 Virtual 属性的不同，说明前面的猜测是对的。

## 理论求证

搜到如下参考：

- [当 Mac 升级到 Catalina 时，苹果在硬盘里施了点魔法](https://sspai.com/post/57052)

推理正确！
