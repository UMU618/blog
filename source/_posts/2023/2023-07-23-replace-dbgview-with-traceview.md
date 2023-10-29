---
layout: post
title: 用 TraceView 取代 DbgView
date: 2023-07-23 20:05:23
description: 稣在教 ChatGPT 写代码
categories: UMUTech
tags:
- debug
- dev
- windows
---
[tv]: https://learn.microsoft.com/en-us/windows-hardware/drivers/devtest/traceview
[tl]: https://learn.microsoft.com/en-us/windows/win32/tracelogging/trace-logging-portal
[tp]: https://learn.microsoft.com/en-us/windows-hardware/drivers/devtest/trace-provider

## 1. 故事

- 开发虚拟显示器驱动时，打太多日志到 DbgView，结果导致驱动被底层主动杀死。

- 有一天，Linux 和 Windows 驱动都精通的钧叔，突然和稣吐槽，WPP 太擸𢶍。

## 2. 问题

- OutputDebugStringA 太慢了！

- WPP 太乱了！

## 3. 相关知识

- ETW（Event Tracing for Windows）是 Windows 操作系统中的一种事件跟踪技术，可以用于记录系统和应用程序生成的事件。ETW 的优点就是性能好，并且同时具备内核态和用户态 API。

- WPP（Windows Software Trace Preprocessor）是一种用于 Windows 软件跟踪的预处理器，可以帮助开发人员在代码中插入跟踪语句，并生成可用于 ETW 的跟踪消息。即，WPP 基于 ETW，只是做了层封装。

- [TraceView][tv] is a trace controller and a trace consumer. 类似于用 DbgView 看 OutputDebugStringA 产生的消息。ETW 产生的消息用 [TraceView][tv] 来看。

## 4. 解决

慢的，不用就行。乱的，换个用法。

既然已经有 [TraceView][tv]，那么我们只需要把 OutputDebugStringA 替换为 ETW 的 API 不就完事了吗？说干就干，先写个简单的 [Trace Provider][tp] 代码：

```cpp
#include <Windows.h>

#include <evntprov.h>

#include <cassert>
#include <iostream>
#include <string_view>

class EtwLog {
 public:
  EtwLog() = default;
  ~EtwLog() {
    if (0 != event_handle_) {
      EventUnregister(event_handle_);
    }
  }

  ULONG Initialize(const GUID& guid) noexcept {
    ULONG ec = EventRegister(&guid, nullptr, nullptr, &event_handle_);
    assert(ERROR_SUCCESS == ec);
    return ec;
  }

  ULONG Log(std::wstring_view message,
            std::uint8_t level = 1,
            std::uint64_t keyword = 1) noexcept {
    assert(0 != event_handle_);
    EVENT_DESCRIPTOR event_descriptor;
    EVENT_DATA_DESCRIPTOR data_descriptor;
    EventDescCreate(&event_descriptor, 0, 0, 0, level, 0, 0, keyword);
    EventDataDescCreate(
        &data_descriptor, message.data(),
        static_cast<ULONG>((message.size() + 1) *
                           sizeof(std::wstring_view::value_type)));
    return EventWrite(event_handle_, &event_descriptor, 1, &data_descriptor);
  }

 private:
  REGHANDLE event_handle_{};
};

int main() {
  std::cout << "Hello ETW!\n";

  EtwLog etw;
  // {aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa}
  static const GUID guid = {0xaaaaaaaa,
                            0xaaaa,
                            0xaaaa,
                            {0xaa, 0xaa, 0xaa, 0xaa, 0xaa, 0xaa, 0xaa, 0xaa}};
  ULONG ec = etw.Initialize(guid);
  if (ERROR_SUCCESS != ec) {
    std::cerr << "Failed to Initialize ETW!\n";
    return ec;
  }
  etw.Log(L"Hello ETW!");   // View via TraceView
}
```

打开 [TraceView][tv]，做好基本配置：

![采用 GUID 方式](/images/2023/20230723-traceview-guid.png)

![没有 TMF，先 Auto，等八哥](/images/2023/20230723-traceview-tmf.png)

然后，运行以上 C++ 代码，回到 [TraceView][tv] 界面，能看到捕获到信息，但并没有“Hello ETW!”，而是写着“解码错误 1168”。

![解码错误](/images/2023/20230723-traceview-1168.png)

到此，恍然大悟，原来 ETW 太底层，所以才有 WPP 定义一系列规范来使用 ETW，只不过 WPP 太老，不好用了。有没有一种不需要 pdb/man/tmf 的使用 ETW 的方式？

有的。它就是 Windows 10 新增的 [TraceLogging][tl]。

> Windows 10 introduces TraceLogging which builds on ETW and provides a simplified way to instrument code for native, .NET and WinRT developers.

> TraceLogging is a system for logging events that can be decoded without a manifest. On Windows, TraceLogging is used in user-mode and kernel-mode to generate Event Tracing for Windows (ETW) events. TraceLogging builds on Event Tracing for Windows (ETW) and provides a simplified way to instrument code.

参考微软给的例子就很容易理解并上手：[C/C++ TraceLogging Examples](https://learn.microsoft.com/en-us/windows-hardware/drivers/devtest/tracelogging-examples)
