---
layout: post
title: Boost【9】Boost.Beast
date: 2023-11-12 23:51:31
description: 稣教 ChatGPT 用 Boost！
categories: UMUTech
tags:
- boost
- cpp
- dev
---
## 介绍

Boost.Beast 是一个 HTTP/WebSocket 库。本文只讨论 WebSocket。

## 相关经验

UMU 先是用过 WebSocket++（websocketpp），又用过 libwebsockets，再用的 Boost.Beast，之后就一直使用 Boost.Beast。

- 2018 年，参与 EOS 开发时，它是用 WebSocket++ 的，跟着学习了一阵子。

- 2020 年，在金山云时，内部版云游戏用 libwebsockets，跟着学习了一阵子。

- 做开源版云游戏——[鎏光云游戏引擎](https://github.com/ksyun-kenc/liuguang)时，特地学习并使用 Boost.Beast。因为公司其实是要求和内部版有一些差异的，正好之前一直用 Boost.Asio，对它的熟悉可以快速套用在 Boost.Beast，于是果断切到 Boost.Beast。

## 什么场景应该使用 WebSocket？

如果您原本使用裸 TCP，您应该知道 TCP 的流式传输，导致您需要自己分包，即界定一个“消息”的边界。鎏光本来设计为同时支持裸 TCP 和 WebSocket 的，所以还保留着处理分包的代码：

<https://github.com/UMU618/liuguang/blob/97f558275571b9be14893e4b55703b4b65cdbda5/src/cge/cge/game_session.cpp#L232>

这对 WebSocket 其实并不需要，它的发送和接受已经是都是“消息”，带着长度的。所以如果您原来用 Asio 写 C/S 程序，把它们改为 Beast，是很容易的，而且代码量会缩减不少。

对于工具性的 C/S 程序，建议下次直接用 Beast 写更省事。

另一种适用场景是需要支持浏览器，即同时支持 C/S 和 B/S 模型。

## Boost.Beast 经验

1. text 和 binary 模式需要区分清楚，如果用于发送音视频，显然应该使用 binary 模式。

2. stream 的默认接收长度是 16MiB，如果不够可以改长点，0 表示最大的 std::uint64_t。

参考：boost/beast/websocket/stream.hpp

```cpp
    /** Set the maximum incoming message size option.

        Sets the largest permissible incoming message size. Message
        frame fields indicating a size that would bring the total
        message size over this limit will cause a protocol failure.

        The default setting is 16 megabytes. A value of zero indicates
        a limit of the maximum value of a `std::uint64_t`.

        @par Example
        Setting the maximum read message size.
        @code
            ws.read_message_max(65536);
        @endcode

        @param amount The limit on the size of incoming messages.
    */
    void
    read_message_max(std::size_t amount);

    /// Returns the maximum incoming message size setting.
    std::size_t
    read_message_max() const;
```

3. 默认开了 deflate 压缩。UMU 在开发 [ClipboardSync](https://github.com/UMU618/ClipboardSync) 时，曾经考虑给剪切板数据压缩，对比了 lz4 和 zstd，很是犹豫，后来发现 Beast 默认开了 deflate 压缩，于是放弃自己用 lz4 或 zstd 压缩。
