---
layout: post
title: 跟 UMU 一起玩 OpenWRT（高级篇2）：不拆机刷不死 U-Boot
date: 2014-05-23 00:23:50
categories: UMUTech
tags:
- embedded
- linux
- openwrt
---
在上一篇《[跟 UMU 一起玩 OpenWRT（高级篇1）：编译不死 U-Boot](/2014/05/21/umutech-openwrt-advanced-1-compiling-uboot/)》介绍了如何编译不死 U-Boot，但是不死 U-Boot 的作者只介绍了用 TTL 线刷方法，UMU 可不想拆机，毕竟拆机感觉并不好……

第一个思路是刷上 DD-WRT 固件，但是找了一下 DD-WRT 木有支持 DIR-505，只好继续蛋疼地编译 OpenWRT。

第一遍在虚拟机从 12:20 编译到 23:56，花费将近 12 小时……刚开始时，有一个下载过程，不断失败，想想是因为公司的网络太烂，于是把下载脚本改了一下：

`<openwrt-svn-dir>/trunk/scripts/download.pl 中的 wget -t5 --timeout=20 --no-check-certificate` 改为 `wget -t5 --timeout=120 --no-check-certificate`

第二天来，刷上，没问题，于是开始改代码去掉 U-Boot 写保护，参考这篇《[Openwrt 中刷写 uboot ART](http://see.sl088.com/wiki/Openwrt_%E4%B8%AD%E5%88%B7%E5%86%99_uboot_art)》<http://see.sl088.com/wiki/Openwrt_%E4%B8%AD%E5%88%B7%E5%86%99_uboot_art>，但结果很不幸，型号不同嘛！

接下来，凭自己的编程水平了，尝试改 `<openwrt-svn-dir>/trunk/target/linux/ar71xx/files/arch/mips/ath79/mach-dir-505-a1.c`，加入下面两个结构体：

```c
static struct mtd_partition dir505_partitions[] = {
 {
  .name  = "u-boot",
  .offset  = 0,
  .size  = 0x010000,
  .mask_flags = 0,
 }, {
  .name  = "art",
  .offset  = 0x010000,
  .size  = 0x010000,
 }, {
  .name  = "mac",
  .offset  = 0x020000,
  .size  = 0x010000,
 }, {
  .name  = "nvram",
  .offset  = 0x030000,
  .size  = 0x010000,
 }, {
  .name  = "language",
  .offset  = 0x040000,
  .size  = 0x040000,
 }, {
  .name  = "firmware",
  .offset  = 0x080000,
  .size  = 0x780000,
  .mask_flags = 0,
 }
};

static struct flash_platform_data dir505_flash_data = {
 .parts  = dir505_partitions,
 .nr_parts       = ARRAY_SIZE(dir505_partitions),
};
```

并将 dir_505_a1_setup 函数里的 `ath79_register_m25p80(NULL);` 改为 `ath79_register_m25p80(&dir505_flash_data);`

测试还是无效……看来必须在源头上使 MTD_WRITEABLE 无效掉，`grep -r MTD_WRITEABLE <openwrt-svn-dir>/trunk/build_dir/target-mips_34kc_uClibc-0.9.33.2/linux-ar71xx_generic/linux-3.10.36/drivers/mtd`，看到几处关键的地方：

```c
if (!(ubi->mtd->flags & MTD_WRITEABLE)) {
```

和

```c
if (!mtd->_write || !(mtd->flags & MTD_WRITEABLE))
```

主要在 mtd_erase、mtd_write 等函数，很明显，C 语言不管在什么平台都是很好懂，看几眼就搞定了，原理是使 MTD_WRITEABLE 这个标志无用掉，您可以设置，但是我把判断这个标志的代码全干掉了，设了也是白设！

最后编译好的 `openwrt-ar71xx-generic-dir-505-a1-squashfs-sysupgrade.bin`，用 sysupgrade 刷一下，`reboot` 后再用 `mtd` 刷不死 U-Boot，一切顺利，成功刷上不死 U-Boot！
