---
layout: post
title: 为什么 EOS 私钥有不同长度？
date: 2020-04-22 16:27:30
categories: UMUTech
tags:
- cryptology
- debug
- dev
- blockchain
---
## 问题

这里有两个 EOS 私钥，它们长度居然不一样？

- PVT_K1_1EEr5aW5162skbocDSMDgoWn9jna6HPSr1TwEMR6PNXbPtRky

- PVT_K1_2bfGi9rYsXQSXXTvJbDAPhHLQUojjaNLomdm3cEJ1XTzMqUt3V

为什么私钥有不同长度？而公钥就都是固定长度呢？

## 知识点

- 私钥是一个大型随机数，而公钥则是私钥乘以椭圆曲线上的基点后对应的点。对于 secp256k1 来说，是 256 位，并且 < n 的整数。

- n 须为质数，Order of G，是使得 `n * G = 0` 的最⼩正整数，n 是安全性最⼤的决定因素。对于 secp256k1 来说，n
 = 0xfffffffffffffffffffffffffffffffebaaedce6af48a03bbfd25e8cd0364141。

- 不是每个数都安全，比如小的数肯定是不安全的，黑客可以从 1 开始枚举，不够大的数很快就被找到对应关系，也可以从 n 倒着枚举，所以太大的也不安全。（PS：临近一些特别数的数也不安全……）一般来说，私钥的安全范围是 [0x0080000000000000000000000000000000000000000000000000000000000000, 0x7fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff]。

## 工具

<https://github.com/UMU618/secp256k1-tools>

使用 private-2-public.js 可以把私钥转为它代表的数字：

```sh
DEBUG=* node private-2-public.js PVT_K1_1EEr5aW5162skbocDSMDgoWn9jna6HPSr1TwEMR6PNXbPtRky
  secp256k1-tools:key-util pvt = 84ade57e2b35cca8972562fcc6d1f6f2fbf078c4f2cfb532eb4d740767c5a8 +0ms
  secp256k1-tools:key-util x = 2110b8d675240f5d548d166cc06b22f44c671d762711a3a67027b74cd166ab76 +9ms
  secp256k1-tools:key-util y = 20ac68b75ad8b0e4bc3ec5705ebaf57c69d2d8268504d6aa95fdebfd0b7db831 +0ms
PVT_K1_1EEr5aW5162skbocDSMDgoWn9jna6HPSr1TwEMR6PNXbPtRky
PUB_K1_75o92oRgWSgx3XzTDYPj1e3hFSRhMnKaUdW8ZZpxJXkhfiGBHS

DEBUG=* node private-2-public.js PVT_K1_2bfGi9rYsXQSXXTvJbDAPhHLQUojjaNLomdm3cEJ1XTzMqUt3V
  secp256k1-tools:key-util pvt = d2653ff7cbb2d8ff129ac27ef5781ce68b2558c41a74af1f2ddca635cbeef07d +0ms
  secp256k1-tools:key-util x = c0ded2bc1f1305fb0faac5e6c03ee3a1924234985427b6167ca569d13df435cf +8ms
  secp256k1-tools:key-util y = eeceff7130fd352c698d2279967e2397f045479940bb4e7fb178fd9212fca8c0 +1ms
PVT_K1_2bfGi9rYsXQSXXTvJbDAPhHLQUojjaNLomdm3cEJ1XTzMqUt3V
PUB_K1_6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5BoDq63
```

## 分析

### 1. 数值比较

- PVT_K1_1EEr5aW5162skbocDSMDgoWn9jna6HPSr1TwEMR6PNXbPtRky，有 56 个字符，去掉前缀和校验码后有 45 个字符，它代表 0x84ade57e2b35cca8972562fcc6d1f6f2fbf078c4f2cfb532eb4d740767c5a8；

- PVT_K1_2bfGi9rYsXQSXXTvJbDAPhHLQUojjaNLomdm3cEJ1XTzMqUt3V，有 57 个字符，去掉前缀和校验码后有 46 个字符，它代表 d2653ff7cbb2d8ff129ac27ef5781ce68b2558c41a74af1f2ddca635cbeef07d。

可以清楚地看出前者短一个字符，数值也相应比较小。

### 2. BASE58 编码的原理

BASE58 的字符集：`123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz`，其中 '1' 代表 0，'z' 代表 57。把待编码的数字不断除以 58，并将余数用对应的字符表示。举个小点的数字做例子：618

```
618 / 58 = 10 .. 38 -> f

10 / 58  = 0  .. 10 -> B
```

拼接余数得 fB，再反转得 Bf。

### 3. BASE58 编码位数关系

一个数编码后，应该长于或等于比它小的数。我们可以通过简单的数学计算得出 45 个字符的 BASE58 编码可以表示的最大数：

zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz = 0xc33ed2d1fbdd3bfe9c22b96164d38cf0d640e1c0ee8b61c39c57899fffffffffff

所以 <= 0xc33ed2d1fbdd3bfe9c22b96164d38cf0d640e1c0ee8b61c39c57899fffffffffff 的私钥编码后是 56 个字符；大于者 59 个字符。

### 4. 旧格式私钥

- 旧格式私钥：5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3

- 对应新格式：PVT_K1_2bfGi9rYsXQSXXTvJbDAPhHLQUojjaNLomdm3cEJ1XTzMqUt3V

同理，只是格式不同罢了。不再展开。

### 5. 为什么公钥是固定长度呢？

因为公钥有个表示奇偶性的前缀，0x02 或者 0x03，所以它的大小范围被限定，没能相差一个 BASE58 字符。

## 相关文章

[基于 ECC 的私钥转为公钥的过程](/2019/08/15/umutech-eosio-private-key-to-public-key/)

[ECC Node.js](/2019/08/16/umutech-ecc-nodejs/)
