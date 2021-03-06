---
layout: post
title: 留一法交叉验证
date: 2018-05-26 23:27:16
categories: 宇督观
tags:
- 数学
- 做题
- debug
---
## 题目

假设有如下一组输入并输出一个实数的数据，则线性回归（Y = bX + c）的留一法交叉验证均方差为？

| X | Y |
| -: | -: |
| 0 | 2 |
| 2 | 2 |
| 3 | 1 |

A. 10/27
B. 20/27
C. 50/27
D. 49/27

## 概念

### 1. 交叉验证（Cross Validation）

也称作循环估计（Rotation Estimation），是一种统计学上将数据样本切割成较小子集的实用方法。

在模式识别（Pattern Recognition）和机器学习（Machine Learning）的相关研究中，经常会将整个数据集合分成两个部分，分别是训练集合和测试集合。在一个 n 个元素的集合，选择 r 个元素做训练集（非空集，r > 0），剩下的 n - r 个做测试集，这可以用“组合”计算有多少种可能。把每种组合都做过一遍就是交叉验证。

### 2. 组合（Combination）

nCr 表示由 n 个不同元素中，每次取出 r 个不重复之元素的组合，用符号 C n（下标）r（上标）表示。

### 3. 留一法交叉验证（Leave-one-out Cross Validation）

只留一个元素做测试集，即：r = n - 1。

### 4. 均方差

标准差（Standard Deviation），别名：标准偏差、实验标准差、均方差，是离均差平方的算术平均数的平方根，用 σ 表示。标准差是方差的算术平方根。标准差能反映一个数据集的离散程度。平均数相同的两组数据，标准差未必相同。

## 解题

三个元素的集合留一，一共有 3C1 = 3 种组合，画 3 个点：

* A = (0, 2)
* B = (2, 2)
* C = (3, 1)

1. 连接 A 和 B，得到直线 Y = 2，C 点的偏差 = 2 - 1 = 1
2. 连接 A 和 C，得到直线 Y = (6 - X) / 3，B 点的偏差 = 4/3 - 2 = -2/3
3. 连接 B 和 C，得到直线 Y = 4 - X，A 点的偏差 = 4 - 2 = 2

所以方差为：(1^2 + (2/3)^2 + 2^2) / 3 = (9 + 4 + 4 * 9) / 27 = 49/27

题目说的是“均方差”，根据百度百科[标准差](https://baike.baidu.com/item/标准差)词条的说法，“均方差”==标准差，要开平方……所以题目中的答案没有一个是对的。出题者想让我们选 D，稣偏要选 F，你懂的 ck……