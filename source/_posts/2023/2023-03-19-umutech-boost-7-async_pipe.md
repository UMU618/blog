---
layout: post
title: Boost【7】async_pipe
date: 2023-03-19 17:13:36
description: 稣是 Windows 用户，ChatGPT 让稣去搬砖！
categories: UMUTech
tags:
- boost
- cpp
- dev
- windows
---
## 1. 问题

- Windows 的管道好奇怪哦！

- 嘶，它看起来好像条沟！

- 噗。

## 2. 概念和分类

按命名来分：

- named，命名（或具名）

- anonymous，匿名

匿名管道的开销低于命名管道，但提供有限的服务。不过匿名管道实际上是由唯一名字的命名管道实现的，所以匿名管道的句柄可以传递给大部分需要命名管道句柄的 APIs。从实现上看，匿名管道是命名管道的特例，“匿名管道的开销低”这个说法，是使用的参数限制了功能导致的，不能从概念去理解这点。

按通信方式来分：

- two-way/duplex，双向（或双工）

- one-way，单向（或单工）

从概念上讲，管道有两端。 单向管道允许一端的进程写入管道，并允许另一端的进程从管道读取。 双向 (或双工) 管道允许进程从它的那端读取和写入。

## 3. 用途和注意事项

总体上看，Windows 的管道设计和其它类 Unix 系统差别比较大。

### 3.1 匿名管道

匿名管道主要用于父进程于子进程之间的通信。

匿名管道是一个未命名的**单向管道**，通常在父进程和子进程之间传输数据。匿名管道始终是本地管道；它们不能用于通过网络进行通信。

CreatePipe 函数创建匿名管道并返回两个句柄：管道的读取句柄和管道的写入句柄。读取句柄对管道具有只读访问权限，写入句柄对管道具有仅写访问权限。若要使用管道进行通信，管道服务器必须将管道句柄传递给另一个进程。通常，这是通过继承完成的；也就是说，进程允许子进程继承句柄。此过程还可以使用 DuplicateHandle 函数复制管道句柄，并使用某种形式的进程间通信（例如 DDE 或共享内存）将其发送到不相关的进程。

匿名管道**不支持异步 (重叠) 读取和写入操作**。 这意味着不能对匿名管道使用 ReadFileEx 和 WriteFileEx 函数。 此外，当这些函数与匿名管道一起使用时，将忽略 ReadFile 和 WriteFile 的 lpOverlapped 参数。

**注意**：类 Unix 系统的匿名管道支持异步 IO，Windows 上需要拿命名管道来模拟匿名管道，以便支持支持异步 IO。

### 3.2 命名管道

命名管道可以用于 IPC，两端进程可以是任意关系，父子关系或者对等关系，典型应用是 LPC（本地的 C/S 模型）的实现。

命名管道可以在局域网里通信，可以用 PIPE_REJECT_REMOTE_CLIENTS 禁止。

管道名称**不区分**大小写。

可以用 PIPE_ACCESS_DUPLEX 指定为双向（双工）。

## 4. boost::process::async_pipe

设计理念：The pipes here are mainly meant for parent-child I/O. 如果您想拿 async_pipe 来写对等关系的 LPC，需要使用 asio，并手动调用一些 Windows APIs。

> If you want to to use async-pipe servers and stuff, you can do that with boost.asio, by using the normal winapi functions and then assign the open pipe to a stream_handle.

async_pipe 的构造函数为：

```cpp
async_pipe::async_pipe(boost::asio::io_context & ios_source,
                       boost::asio::io_context & ios_sink,
                       const std::string & name, bool private_)
```

阅读其代码可知，boost::process::async_pipe 只使用命名管道，并给内部**两个管道句柄**起了名字：source 和 sink。其中：

- source 是用 CreateNamedPipe 创建 PIPE_ACCESS_INBOUND 的管道，作为服务端用来读取客户端发来的数据。

- sink 是用 CreateFile 打开现存的管道，用于写入。sink 字面意思是下沉，可以理解为灌入，例如：“把水灌入下水道”。

```
this end          that end
        +--------+
 source | <--    | sink
        +--------+

        +--------+
   sink |    --> | source
        +--------+
```

如果不传名字，会自动指定，名字生成的代码如下：

```cpp
inline std::string make_pipe_name()
{
    std::string name = "\\\\.\\pipe\\boost_process_auto_pipe_";

    auto pid = ::boost::winapi::GetCurrentProcessId();

    static std::atomic_size_t cnt{0};
    name += std::to_string(pid);
    name += "_";
    name += std::to_string(cnt++);

    return name;
}
```

鉴于其设计用途，**建议不要自己命名**，省事，还不容易冲突。

其它平台的管道大多是单向（单工或半双工）的，为了跨平台，Boost 没有封装 PIPE_ACCESS_DUPLEX 属性的双向（双工）管道。

private_ 参数为 true 时，管道只有一个实例，而 false 则可以有 PIPE_UNLIMITED_INSTANCES 个实例，即最多 255 个。

### async_pipe 使用范例

父子进程之间的通信：

```cpp
#include <string>

#include <boost/process.hpp>

namespace bp = boost::process;

int main() {
    boost::asio::io_context io_context;
    bp::async_pipe p1(io_context);
    bp::async_pipe p2(io_context);
    bp::system(
        "test.exe",
        bp::std_out > p2,
        bp::std_in < p1,
        io_context,
        bp::on_exit([&](int exit, const std::error_code& ec_in)
            {
                p1.async_close();
                p2.async_close();
            })
    );
    std::vector<char> in_buf;
    std::string value = "my_string";
    boost::asio::async_write(p1, boost::asio::buffer(value),  []( const boost::system::error_code&, std::size_t){});
    boost::asio::async_read (p2, boost::asio::buffer(in_buf), []( const boost::system::error_code&, std::size_t){});
}
```

## 5. 参考

[Pipes (Interprocess Communications)](https://learn.microsoft.com/en-us/windows/win32/ipc/pipes) / [管道 (进程间通信)](https://learn.microsoft.com/zh-cn/windows/win32/ipc/pipes)

[[Windows][Pipes] Can't open named pipe in Windows: error 231 (All pipe instances are busy.) #83](https://github.com/boostorg/process/issues/83)