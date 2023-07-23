---
layout: post
title: 跟 UMU 一起玩 OpenWRT（入门篇16）：Python3
date: 2020-02-23 18:17:39
categories: UMUTech
tags:
- embedded
- linux
- openwrt
- python
---
## 为什么 Python3？

1. Shell 不适合某些复杂运算，尤其是 OpenWRT 用的 ash。

2. Lua 缺乏某些 SDK，比如说阿里云 SDK 就没有 Lua 版。

3. Python2 已经过时。

4. Node.js 在小型设备上不如 Python3 高效。

5. C、C++ 之流太难了！Go、Rust 还得编译，麻烦。

6. Perl、Ruby 已没落。

## 一个例子

当 IPv6 地址变化时，将地址发送到钉钉：<https://github.com/UMU618/openwrt-ipv6-addresses>

## 安装与调试

### 1. 安装可执行程序

```sh
opkg install python3-base
```

安装 `python3-base` 之后，就可以运行 `python3` 了。

```sh
root@UMU:~# python3
Could not find platform dependent libraries <exec_prefix>
Consider setting $PYTHONHOME to <prefix>[:<exec_prefix>]
Python 3.7.6 (default, Feb 11 2020, 12:41:31)
[GCC 7.5.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```

用以下 Python 代码，打印目前已有的模块：

```python
import sys

## 打印的信息太长
i = 0
for m in sys.modules:
    i += 1
    print('{0:2d} {1:32s} {2}'.format(i, m, sys.modules[m]))

## 不精准
i = 0
for m in sys.modules.values():
    i += 1
    if m.__spec__:
        v = m.__spec__.origin
    elif m.__builtins__:
        v = '-'
    else:
        v = ''
    print('{0:2d} {1:32s} {2}'.format(i, m.__name__, v))

## 推荐使用
i = 0
for m in sys.modules.values():
    i += 1
    s = str(m)
    start = s.find("'")
    end = s.find("'", start+1)
    k = s[start+1:end]
    start = s.find('from', end+1)
    if start > -1:
        start += 4
        end = s.find('>', start+1)
    else:
        start = s.find('(', end+1)
        end = s.find(')', start+1)
    v = s[start+1:end]
    print('{0:2d} {1:28s} {2}'.format(i, k, v))
```

结果为：

```
 1 sys                          built-in
 2 builtins                     built-in
 3 _frozen_importlib            frozen
 4 _imp                         built-in
 5 _thread                      built-in
 6 _warnings                    built-in
 7 _weakref                     built-in
 8 zipimport                    built-in
 9 _frozen_importlib_external   frozen
10 io                           built-in
11 marshal                      built-in
12 posix                        built-in
13 encodings                    '/usr/lib/python3.7/encodings/__init__.pyc'
14 codecs                       '/usr/lib/python3.7/codecs.pyc'
15 _codecs                      built-in
16 encodings.aliases            '/usr/lib/python3.7/encodings/aliases.pyc'
17 encodings.utf_8              '/usr/lib/python3.7/encodings/utf_8.pyc'
18 _signal                      built-in
19 __main__                     built-in
20 encodings.latin_1            '/usr/lib/python3.7/encodings/latin_1.pyc'
21 io                           '/usr/lib/python3.7/io.pyc'
22 abc                          '/usr/lib/python3.7/abc.pyc'
23 _abc                         built-in
24 site                         '/usr/lib/python3.7/site.pyc'
25 os                           '/usr/lib/python3.7/os.pyc'
26 stat                         '/usr/lib/python3.7/stat.pyc'
27 _stat                        built-in
28 posixpath                    '/usr/lib/python3.7/posixpath.pyc'
29 genericpath                  '/usr/lib/python3.7/genericpath.pyc'
30 posixpath                    '/usr/lib/python3.7/posixpath.pyc'
31 _collections_abc             '/usr/lib/python3.7/_collections_abc.pyc'
32 _sitebuiltins                '/usr/lib/python3.7/_sitebuiltins.pyc'
33 atexit                       built-in
```

下面来实现获取 IPv6 地址的功能：

```python
def net_hex_to_ipv6(h):
    ipv6 = h[0:4]
    i = 4
    while i < len(h):
        ipv6 += ':' + h[i:i+4]
        i += 4
    return ipv6

with open('/proc/net/if_inet6') as f:
    for line in f:
        p = line.split()
        if p[3] == '00' and (int(p[4], 16) & 0x80) != 0x80:
            ip = net_hex_to_ipv6(p[0])
            print(ip)
    f.close()
```

以上代码有个“美中不足”：只能打印地址的“首选格式”，不支持“压缩格式”。下面改进！

### 2. 安装轻量库

UMU 打算使用 socket 模块的工具函数格式化 IPv6 地址，但目前已安装的 `python3-base` 不带 socket 模块：

```python
>>> import socket
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ModuleNotFoundError: No module named 'socket'
```

所以需要安装 `python3-light`：

```sh
opkg install python3-light
```

装完即可愉快地玩耍：

```python
import socket

ip = '0618:0618:0618:0618:0000:0000:0000:0618'
print(ip, '->',
    socket.inet_ntop(socket.AF_INET6, socket.inet_pton(socket.AF_INET6, ip)))
```

以上代码打印：`0618:0618:0618:0618:0000:0000:0000:0618 -> 618:618:618:618::618`。

### 3. 全量安装

如果 `python3-light` 还不能满足您，推荐来个全家桶：

```sh
opkg install python3
```

PS: 不要以为只要上面这句就全装上了，前面的 `opkg install python3-base` 是必要的！如果只装 `python3`，则 `/usr/bin/python3` 并不存在！

（完）
