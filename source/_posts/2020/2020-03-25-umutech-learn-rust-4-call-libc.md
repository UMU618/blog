---
layout: post
title: 学习 Rust【4】调用 libc
date: 2020-03-25 13:04:28
categories: UMUTech
tags:
- dev
- linux
- rust
---
## 问题

[Rust中如何使用linux的原生api？ - 知乎](https://www.zhihu.com/question/381987281)

## 概述

很多语言调用 C 语言写的模块来弥补自己某些不足。Rust 当然也可以调用 C 语言开发的模块，不过这是不安全的。

## 代码

1. 在 Cargo.toml 中加入依赖库：

```toml
[dependencies]
libc = "0.2.68"
```

2. Rust 示例代码：

```rust
use libc;

fn main() {
    let pid = unsafe { libc::fork() };
    if pid < 0 {
        eprintln!("错误！");
    } else if pid == 0 {
        println!("子进程空间");
    } else {
        println!("父进程空间, 子进程 pid 为 {}", pid);
    }
}
```

3. macOS **也**测试通过。
