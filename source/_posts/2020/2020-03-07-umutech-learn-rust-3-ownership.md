---
layout: post
title: 学习 Rust【3】所有权
date: 2020-03-07 17:07:26
categories: UMUTech
tags:
- dev
- rust
---
## 摘要

Rust 三大设计宗旨：内存安全、零成本抽象、实用。本文从所有权角度来学习~~师兄妹的爱恨情仇~~“内存安全”。

## 所有权规则

- 值归变量所有。

- 当变量超出使用范围时，变量值所占用的内存将被释放。这是类似于 C++ 的 RAII 概念。

- 变量值可以由其他变量使用，但需遵守由编译器强制要求的若干规则。

前两条是其它语言也有的，没啥好说，重点放在第三条。

## 四种使用方法和规则

- **克隆（clone）**：此处将值复制到新的变量。新变量拥有新的复制值的所有权，而原始变量保留其原始值的所有权。

  > 你有一本书，稣按照你那本书，买了一样的书。你的书是你的书，稣的书是稣的书。

- **移动（move）**：所有权被转移到另一个要使用该值的变量，原始变量不再拥有所有权。

  > 学姐含情脉脉地把她的书**送给**稣。

- **不可变借用（immutable borrow）**：没有发生所有权转移，但是可以通过另一个变量读取该值。当借用变量超出范围，内存不会被回收，因为借用变量没有所有权。

  > 学长不太情愿地把书借给稣，并交代：“书借你，只能看，绝壁不要在上面做笔记，被我发现会砍死你的哦！我偶尔会找你查查。”

- **可变借用（mutable borrow）**：可以通过另一个变量对该值进行读取和写入操作。当借用变量超出范围，内存也不会回收，因为借用变量没有所有权。

  > 稣把书借给学妹时说：“这书你随便用，把稣的书当做你自己的书，等到用不上时再还。”

重点在于一个“借”字：书的所有权属于其主人，但主人将书借出之后，自己是无法再在书上做笔记啦！

### 不可变借用（immutable borrow）规则

- 借出期间，借方不能写：学长说过，在书上乱画，要砍死稣！

- 借出期间，所有者不能写：这书学长已经学完，偶尔要复习，但已经不需要写笔记，借出去之后就更不会写了。

```rust
fn main() {
    let mut owner = format!("学长的书");

    // 学长大笔一挥，在书上记了一点心得！
    owner.push('.');

    // 稣不可变借用学长的书
    let borrower = &owner;

    // 学长随时可以来找稣翻翻他自己的书
    println!("学长读{}", owner);
    println!("学长再读{}", owner);

    // 没问题，稣可以读学长的书
    println!("稣读{}", borrower);

    // error: 书还在稣手里，学长不能写
    //owner.push('.');

    println!("稣再读{}", borrower);

    // error: 学长说过，在书上乱画，要砍死稣！
    //borrower.push('.');

    // 稣已经归还，学长可以做笔记了
    owner.push('.');
}
```

### 可变借用（mutable borrow）规则

- 不能多次可变借用：只能有一个独占的学妹（active borrow），稣不能同时承诺给多个学妹“随便用”，不然学妹们可能打起来……

- 所有者不能再读写：书在学妹手里随便蹂躏，稣虽然心疼，但不能说！等她爽（huan）了再说吧。


```rust
fn main() {
    let mut owner = format!("稣的书");

    // 学妹可变借用稣的书
    let mutable_borrower = &mut owner;

    // error[E0499]: cannot borrow `owner` as mutable more than once at a time
    // 学姐也要借，稣表示：要书没有，要命一条！
    //let mutable_borrower_b = &mut owner;

    println!("学妹读{}", mutable_borrower);

    // error[E0502]: cannot borrow `owner` as immutable because it is also borrowed as mutable
    // 学妹暂时完全掌控稣的书
    //println!("稣读不鸟{}", owner);

    // 学妹大笔一挥，在书上记了一点心得！
    mutable_borrower.push('.');
    println!("学妹再读{}", mutable_borrower);

    // 书已经归还给稣，稣可以做笔记了
    owner.push('!');
    println!("稣读{}", owner);
}
```
