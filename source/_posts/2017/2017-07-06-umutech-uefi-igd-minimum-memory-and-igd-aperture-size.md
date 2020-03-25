---
layout: post
title: UEFI 里的 IGD Minimum Memory 和 IGD Aperture Size
date: 2017-07-06 23:48:26
categories: UMUTech
tags:
- ops
- windows
- 调研
---
今天进 UEFI 看到集显的两个设置选项：IGD Minimum Memory 和 IGD Aperture Size，想着 UMU 的 NUC 有 32G 内存，要不要改大点？然后搜一下他们的作用，结果发现**最好不要改……**

## 知识

1. Adjusting the minimum memory can impact graphics performance in legacy operating systems (Windows 7/8/8/1).

   The default value (64 MB) is recommended for Windows 10. Windows 10 will allocate graphics memory dynamically when it loads, so setting the IGD minimal memory to higher value may not improve performance.

2. Keep the default BIOS setting for IGD Aperture Size and IGD Min Memory. This values are used only during POST and to boot of the Windows. 

   Window 10 assigns automatically the maximum available graphics memory and it depends off how much RAM you have. Usually it assigns about half of available RAM.

## 参考

<https://communities.intel.com/thread/106880>

<https://communities.intel.com/thread/106428>
