---
layout: post
title: 无人在意的八哥
date: 2018-11-25 13:59:53
categories: 宇督观
tags:
- debug
---
## 1. 主题曲《像屎》

> 实习生黑锅
> 是很遥远的事情
> 八哥算什么
> 早无人在意
> 前埔不夜城
> 处处烤鱼
> 酒杯中好一片男男风情
> 最肯忘却故人失
> 最不屑一顾是相思
> 养着老怕人笑
> 还怕人看轻
> 新又来看乱码呆
> 竟不见有心人去改
> 庸才占着茅坑前途不在

## 2. 百度分享不支持 HTTPS

[百度分享](http://share.baidu.com/code)不支持 HTTPS 的解决方案：[https://github.com/hrwhisper/baiduShare](https://github.com/hrwhisper/baiduShare)，最早是 2016-07-09 发布，说明百度分享不支持 HTTPS 已经两年以上。

### 结论

百度可能听不进用户的话，用户宁愿自己解决问题……稣也亲自反馈过，应该是被无视了，至今还没官方支持！

## 3. Linux 内核代码风格翻译错误

[Linux 内核代码风格 v4.19](https://www.kernel.org/doc/html/v4.19/translations/zh_CN/coding-style.html#id12)

> 分配一个零长数组的首选形式是这样的：
> `p = kcalloc(n, sizeof(...), ...);`

[原文](https://www.kernel.org/doc/html/latest/process/coding-style.html#allocating-memory
)是“The preferred form for allocating a zeroed array is the following:”，所以“零长”应该改为“填零”。

[kcalloc 的文档](https://www.kernel.org/doc/htmldocs/kernel-api/API-kcalloc.html)也说：“kcalloc — allocate memory for an array. The memory is set to zero. ”

kcalloc 的定义 [/include/linux/slab.h](https://github.com/torvalds/linux/blob/v4.19/include/linux/slab.h) 更能说明：

```
/**
 * kcalloc - allocate memory for an array. The memory is set to zero.
 * @n: number of elements.
 * @size: element size.
 * @flags: the type of memory to allocate (see kmalloc).
 */
static inline void *kcalloc(size_t n, size_t size, gfp_t flags)
{
	return kmalloc_array(n, size, flags | __GFP_ZERO);
}
```

“零长数组”应该是指：`char u[0];`

### 结论

疑智商太高，学习太快，中国的内核开发者都不屑看翻译的文档。
