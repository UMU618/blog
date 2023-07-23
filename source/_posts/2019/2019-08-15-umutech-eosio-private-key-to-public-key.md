---
layout: post
title: 基于 ECC 的私钥转为公钥的过程
date: 2019-08-15 20:53:45
categories: UMUTech
tags:
- nodejs
- cryptology
- blockchain
---
## 基本知识

ECC 体系中，私钥是一个大型随机数，而公钥则是私钥乘以椭圆曲线上的基点后对应的点。

[meet-one](https://github.com/meet-one)/[private-to-public](https://github.com/meet-one/private-to-public) 是 MEET.ONE 开发的私钥转公钥工具。

EOS 支持的 EC 有两种：secp256k1（以下简称 k1）、secp256r1，下面以 k1 为例，结合 Node.js 和 Python 代码介绍转换过程。

## 涉及算法

- **BASE58**：编解码私钥、公钥。
- **SHA-256**：校验私钥，本文忽略此步。
- **ECC, secp256k1**：计算公钥。
- **RIPEMD-160**：校验公钥。

## 转换过程

### 1. 解码私钥

假设私钥为：`5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3`，这是 base58 编码，用 bs58 库解码：

```js
const bs = require('bs58')

bs.decode('5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3')
```

得到 `<Buffer 80 d2 65 3f f7 cb b2 d8 ff 12 9a c2 7e f5 78 1c e6 8b 25 58 c4 1a 74 af 1f 2d dc a6 35 cb ee f0 7d aa 08 64 4a>`，其中第 1 个字节 `0x80` 是类型，末尾的 4 字节是校验码。

这里我们不关心校验码，也可以直接用 `bs58check` 解码：

```js
const bsc = require('bs58check')

bsc.decode('5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3')
```

得到 `<Buffer 80 d2 65 3f f7 cb b2 d8 ff 12 9a c2 7e f5 78 1c e6 8b 25 58 c4 1a 74 af 1f 2d dc a6 35 cb ee f0 7d>`，去掉首字节后为：

`d2 65 3f f7 cb b2 d8 ff 12 9a c2 7e f5 78 1c e6 8b 25 58 c4 1a 74 af 1f 2d dc a6 35 cb ee f0 7d`

这是 256bit 整数的 Big endian 字节流表示，转为 16 进制整形为 `0xd2653ff7cbb2d8ff129ac27ef5781ce68b2558c41a74af1f2ddca635cbeef07d`，记为 `pk`。

### 2. 构造椭圆曲线，求公钥的坐标

根据 [SECG](http://secg.org/) 规定的 k1 的参数，我们用基于 Python 的 [SAGE](https://sagecell.sagemath.org/) 构造 k1 对应的椭圆曲线，然后计算 `pk * G`：

```python
a = 0
b = 7
p = 0xfffffffffffffffffffffffffffffffffffffffffffffffffffffffefffffc2f
Gx = 0x79be667ef9dcbbac55a06295ce870b07029bfcdb2dce28d959f2815b16f81798
Gy = 0x483ada7726a3c4655da4fbfc0e1108a8fd17b448a68554199c47d08ffb10d4b8
E = EllipticCurve (GF(p), [a, b])
pk = 0xd2653ff7cbb2d8ff129ac27ef5781ce68b2558c41a74af1f2ddca635cbeef07d
G = E(Gx, Gy)
pk * G
```

结果为：`(87237761414843254130560834629777710286905276524352264071298714336416392033743 : 108016191455113306196371645921919775466659772908675410052799661524790827329728 : 1)`

**注：这里的 (x : y : z) 是射影坐标，一般采用笛卡尔坐标系表示，为 (x/z, y/z)。**

### 3. 编码公钥

取 x 值：`87237761414843254130560834629777710286905276524352264071298714336416392033743`

16 进制为：`0xc0ded2bc1f1305fb0faac5e6c03ee3a1924234985427b6167ca569d13df435cf`

Big endian 字节流表示为：`[0xc0, 0xde, 0xd2, 0xbc, 0x1f, 0x13, 0x05, 0xfb, 0x0f, 0xaa, 0xc5, 0xe6, 0xc0, 0x3e, 0xe3, 0xa1, 0x92, 0x42, 0x34, 0x98, 0x54, 0x27, 0xb6, 0x16, 0x7c, 0xa5, 0x69, 0xd1, 0x3d, 0xf4, 0x35, 0xcf]`

由于 y 值是偶数，所以添加一个前缀 2，得到：`[2, 0xc0, 0xde, 0xd2, 0xbc, 0x1f, 0x13, 0x05, 0xfb, 0x0f, 0xaa, 0xc5, 0xe6, 0xc0, 0x3e, 0xe3, 0xa1, 0x92, 0x42, 0x34, 0x98, 0x54, 0x27, 0xb6, 0x16, 0x7c, 0xa5, 0x69, 0xd1, 0x3d, 0xf4, 0x35, 0xcf]`

**注：若 y 为奇数，则前缀为 3。**

**注：为什么这么规定？只用 x 值和 y 值的奇偶性表示一个点，这叫公钥的压缩格式。因为只要有 x，就可以通过 k1 的椭圆曲线方程式 $y^2 = x^3 + 7 \mod p$ 求出 y，但此时 y 会有两个解，又由于 p 是一个大质数，必定为奇数，故两个 y 解的和 mod p 一定等于 0，即一奇一偶，所以用 x 和 y 的奇偶性标志即可代表这个点。**

接着求校验码：

```js
const ripemd160 = require('ripemd160')

new ripemd160().update(Buffer.from([2, 0xc0, 0xde, 0xd2, 0xbc, 0x1f, 0x13, 0x05, 0xfb, 0x0f, 0xaa, 0xc5, 0xe6, 0xc0, 0x3e, 0xe3, 0xa1, 0x92, 0x42, 0x34, 0x98, 0x54, 0x27, 0xb6, 0x16, 0x7c, 0xa5, 0x69, 0xd1, 0x3d, 0xf4, 0x35, 0xcf])).digest()
```

得到：`<Buffer eb 05 f9 d2 c6 dd 62 f7 f2 a0 f7 61 ea 1d 8c 0b 84 4a 3b 52>`，取前 4 字节，添加到末尾，得到：`[2, 0xc0, 0xde, 0xd2, 0xbc, 0x1f, 0x13, 0x05, 0xfb, 0x0f, 0xaa, 0xc5, 0xe6, 0xc0, 0x3e, 0xe3, 0xa1, 0x92, 0x42, 0x34, 0x98, 0x54, 0x27, 0xb6, 0x16, 0x7c, 0xa5, 0x69, 0xd1, 0x3d, 0xf4, 0x35, 0xcf, 0xeb, 0x05, 0xf9, 0xd2]`

然后，base58 编码：

```js
const bs = require('bs58')

bs.encode(Buffer.from([2, 0xc0, 0xde, 0xd2, 0xbc, 0x1f, 0x13, 0x05, 0xfb, 0x0f, 0xaa, 0xc5, 0xe6, 0xc0, 0x3e, 0xe3, 0xa1, 0x92, 0x42, 0x34, 0x98, 0x54, 0x27, 0xb6, 0x16, 0x7c, 0xa5, 0x69, 0xd1, 0x3d, 0xf4, 0x35, 0xcf, 0xeb, 0x05, 0xf9, 0xd2]))
```

得到：`6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV`，加上前缀 `EOS`，即为公钥。
