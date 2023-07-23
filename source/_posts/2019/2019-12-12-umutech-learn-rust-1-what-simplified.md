---
layout: post
title: 学习 Rust【1】简化掉什么？
date: 2019-12-12 18:43:02
categories: UMUTech
tags:
- dev
- rust
- optimization
---
结论先行：**从语法上说，Rust 基本无敌。**

## 1. ++ 和 \-\-

| 语言 | 有无 ++、\-\- 语法 |
| :- | :- |
| C/C++/C#/Java | 有 |
| Go | 只支持放变量后，不支持放变量前 |
| Python/Rust/Scala | 无 |

++、\-\- 一般是 +=、-= 的特例（除了 C++ 的迭代器），没有必要单独支持，新语言倾向于语法的单一性。

Python 的情况比较有意思，放后面是语法错误，放前面其实就是正负号，+ 写两次还是原来的数，\- 写两次是负负得正，也还是原来的数。

## 2. 三目运算符（?:）

| 语言 | 有无 ?: 语法 |
| :- | :- |
| C/C++/C#/Java/Swift | 有 |
| Go/Python/Rust/Scala | 无 |

Rust 的 `let = if else` 就有 C 语言 `?:` 的功能，即判断语句的子语句块可以有返回值。

## 3. 条件无需括号

| 语言 | 条件需不需要括号 |
| :- | :- |
| C/C++/Java/Scala | 需要 |
| Go/Python/Rust/Swift | 不需要 |

字符是能少打一个是一个，有效预防鼠标手。另外，Go 和 Rust 的语句块必须包含于 {}。

## 4. 异常处理

| 语言 | 异常处理机制 |
| :- | :- |
| C/C++ | 编译器扩展 __try...__except...__finally, __leave |
| C++/C#/Java/Scala/Swift | throw, try...catch...finally |
| Python | raise, try...except...else, try...finally |
| Go/Rust | 无 |

## 5. 换行符（;）

| 语言 | 换行符 |
| :- | :- |
| C/C++/C#/Java | 必须 |
| JavaScript/Scala/Swift | 可选，有少数必须的情况 |
| Python/Go | 无 |
| Rust | 有是有，无是无（return），两者含义不同 |

Rust 有分号的是语句（statement），返回值是 ()，即没有返回值。而没分号的是表达式（expression），返回值就是自身的值。

其实想说的是：有的 return 被简化掉了。省略 ; 就是省略 return，真香。但是，由于隐含 return，所以只能用于语句块的最后一行。

## 6. case 隐含 break

| 语言 | case 是否隐含 break |
| :- | :- |
| C/C++/C#/Java | 必须显式 break |
| Go/Rust/Swift | 隐含 break |

Rust 优秀在用 match 代替 switch，明确告诉大家这是新语法，而 Go/Swift 用 switch，却改变 case 行为，还多出一个 fallthrough 关键字，容易引起[鲸神魂裂](/tags/鲸神魂裂/)。
