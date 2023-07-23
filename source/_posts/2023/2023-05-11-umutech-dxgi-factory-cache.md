---
layout: post
title: DXGI 类工厂缓存
date: 2023-05-11 11:42:37
description: 稣是 Windows 开发，ChatGPT 让稣卷起来！
categories: UMUTech
tags:
- cpp
- dev
- directx
- windows
---
## 1. 问题

- CreateDXGIFactory1 太慢！

- CreateDXGIFactory2 太慢！

## 2. 验证

```cpp
int main() {
  // Do NOT use CreateDXGIFactory + IDXGIFactory
  // CreateDXGIFactory1 + IDXGIFactory1: Windows 7
  // CreateDXGIFactory1 + IDXGIFactory2: Windows 8 and Platform Update for Windows 7
  // CreateDXGIFactory2 + IDXGIFactory3: Windows 8.1
  // CreateDXGIFactory2 + IDXGIFactory4: Windows 10
  // CreateDXGIFactory2 + IDXGIFactory5: Windows 10
  // CreateDXGIFactory2 + IDXGIFactory6: Windows 10 1803
  // CreateDXGIFactory2 + IDXGIFactory7: Windows 10 1809
  auto now = std::chrono::steady_clock::now();
  CComPtr<IDXGIFactory7> dxgi_factory_;
  HRESULT hr = CreateDXGIFactory1(IID_PPV_ARGS(&dxgi_factory_));
  // HRESULT hr = CreateDXGIFactory2(0, IID_PPV_ARGS(&dxgi_factory_));
  if (FAILED(hr)) {
    std::cerr << std::format("!CreateDXGIFactory1(), #0x{:08X}\n", hr);
    return hr;
  }
  auto diff = std::chrono::steady_clock::now() - now;
  std::cout << std::format("CreateDXGIFactory1() costs {}\n", diff);
}
```

测试结果：

```
// CPU: Intel(R) N100
// GPU: Intel(R) UHD Graphics
CreateDXGIFactory1() costs 14795500ns
CreateDXGIFactory1() costs 12253500ns
CreateDXGIFactory1() costs 7063000ns

CreateDXGIFactory2() costs 13026300ns
CreateDXGIFactory2() costs 8201400ns
CreateDXGIFactory2() costs 8535500ns

// CPU: 11th Gen Intel(R) Core(TM) i7-11700F @ 2.50GHz
// GPU: NVIDIA GeForce RTX 3060
CreateDXGIFactory2() costs 11'770'500ns
CreateDXGIFactory2() costs 13'821'500ns
CreateDXGIFactory2() costs 11'671'900ns
```

居然平均 10ms 以上，确实太慢！

## 3. 解决

既然创建类工厂慢，就不要频繁创建，创建后把它缓存起来。

新问题：缓存失效怎么办？

> There are only two hard things in Computer Science: cache invalidation and naming things.
> -- Phil Karlton
>
> Phil Karlton：计算机科学领域有两个难题：一个是缓存失效，另一个就是命名。

答案：[IDXGIFactory1::IsCurrent](https://learn.microsoft.com/en-us/windows/win32/api/dxgi/nf-dxgi-idxgifactory1-iscurrent)

以下是 MSDN 说的：

> Informs an application of the possible need to re-create the factory and re-enumerate adapters.
>
>  FALSE, if a new adapter is becoming available or the current adapter is going away. TRUE, no adapter changes.
>
>  IsCurrent returns FALSE to inform the calling application to re-enumerate adapters.

这就很含糊了，IsCurrent 返回 FALSE 时，究竟该 re-enumerate adapters，还是 re-create the factory？

假设稣需要反复调用 EnumAdapterByLuid，如果只 re-enumerate adapters，那么稣每次直接用 EnumAdapterByLuid 即可，根本不需要理会 IsCurrent。所以这显然是错的，真正需要的是 re-create the factory，再 re-enumerate adapters。

这点可以在类工厂创建后，禁用和启用显示适配器来验证。根据稣的测试，类工厂创建后，禁用显示适配器，此时 IsCurrent 返回 FALSE，依然可以枚举出被禁用的显示适配器。只有重建类工厂，才能得到当前正确工作的显示适配器。