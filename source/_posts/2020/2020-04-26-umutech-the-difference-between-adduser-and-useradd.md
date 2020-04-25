---
layout: post
title: adduser 和 useradd 的区别
date: 2020-04-26 00:40:03
categories: UMUTech
tags:
- ops
- linux
---
## 需求

在 armbian 系统里新建个账号。

这当然是个简单的任务，问题是发现居然同时存在 adduser 和 useradd 两个命令。

## 解决选择恐惧症

1. adduser 不是可执行程序。

```sh
which adduser
/usr/sbin/adduser

ldd /usr/sbin/adduser
        not a dynamic executable
```

2. useradd 是可执行程序。

```sh
which useradd
/usr/sbin/useradd

ldd /usr/sbin/useradd
        linux-vdso.so.1 (0xbea59000)
        libaudit.so.1 => /lib/arm-linux-gnueabihf/libaudit.so.1 (0xb6ea1000)
        libselinux.so.1 => /lib/arm-linux-gnueabihf/libselinux.so.1 (0xb6e77000)
        libsemanage.so.1 => /usr/lib/arm-linux-gnueabihf/libsemanage.so.1 (0xb6e3f000)
        libc.so.6 => /lib/arm-linux-gnueabihf/libc.so.6 (0xb6d45000)
        /lib/ld-linux-armhf.so.3 (0xb6efb000)
        libcap-ng.so.0 => /lib/arm-linux-gnueabihf/libcap-ng.so.0 (0xb6d31000)
        libpcre.so.3 => /lib/arm-linux-gnueabihf/libpcre.so.3 (0xb6cd4000)
        libdl.so.2 => /lib/arm-linux-gnueabihf/libdl.so.2 (0xb6cc1000)
        libsepol.so.1 => /lib/arm-linux-gnueabihf/libsepol.so.1 (0xb6c44000)
        libbz2.so.1.0 => /lib/arm-linux-gnueabihf/libbz2.so.1.0 (0xb6c28000)
        libpthread.so.0 => /lib/arm-linux-gnueabihf/libpthread.so.0 (0xb6c03000)
```

3. 推测 adduser 是脚本，内部调用 useradd。求证之！

```sh
head -1 /usr/sbin/adduser
#!/usr/bin/perl

grep useradd /usr/sbin/adduser
    my $useradd = &which('useradd');
    &systemcall($useradd, '-d', $home_dir, '-g', $ingroup_name, '-s',
    my $useradd = &which('useradd');
    &systemcall($useradd, '-d', $home_dir, '-g', $ingroup_name, '-s',
    # useradd without -p has left the account disabled (password string is '!')
```

这说明 adduser 是 perl 脚本，内部确实调用 useradd。

4. 直觉告诉 UMU，应该用 adduser，如果 useradd 很好用，不会有 adduser 存在的必要。
