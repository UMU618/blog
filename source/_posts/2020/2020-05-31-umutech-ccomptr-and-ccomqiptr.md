---
layout: post
title: CComPtr 和 CComQIPtr
date: 2020-05-31 16:22:06
description:
categories: UMUTech
tags:
- atl
- cpp
- debug
- dev
- windows
---
## 问题

CComPtr 和 CComQIPtr 长得这么像，有啥关系和区别？

## 分析

1. 看代码 CComQIPtr 继承自 CComPtr，CComPtr\<IUnknown\> 没问题，但 CComQIPtr\<IUnknown\> 报错，应该使用 CComQIPtr\<IUnknown, &IID_IUnknown\>。

2. 不同类型 CComPtr\<\> 不能直接互相构造；CComQIPtr\<\> 则可以，因为 CComQIPtr 会进行目标类型的 QueryInterface。

```cpp
CComPtr<IUnknown> u;
// ...
CComPtr<IDispatch> d(u);    // error

CComQIPtr<IDispatch> d(u);    // right, will call QueryInterface
```

3. 两者构造/赋值时，都会进行 AddRef，如果不想 AddRef，可以使用裸指针（必须十分清楚自己在干嘛！）。

```cpp
CComPtr<IUnknown> u;
// ...
CComPtr<IUnknown> u1(u);    // will call AddRef

CComQIPtr<IDispatch> d(u);    // will call QueryInterface(call AddRef impliedly)

auto raw = static_cast<IDispatch*>(u.p); // won't call AddRef
```

4. 两者赋值时，小部分行为不同。以下模板使得，当等号两边类型不同时，CComPtr 为左值和 CComQIPtr 为左值，表现不同。

```cpp
// CComPtr
template <typename Q>
T* operator=(_Inout_ const CComPtr<Q>& lp) throw()
{
    if(!this->IsEqualObject(lp) )
    {
        AtlComQIPtrAssign2((IUnknown**)&this->p, lp, __uuidof(T));
    }
    return *this;
}
```

## 总结

- 您可以忘记 CComPtr，只使用 CComQIPtr；

- 或者，尽量使用 CComPtr，只在必要时使用 CComQIPtr。

## 测试代码

```cpp
#include <iostream>

#include <atlbase.h>
#include <atlcom.h>
#include <atlstr.h>

MIDL_INTERFACE("00554d55-0000-0000-C000-000000000041")
IA : public IUnknown {
 public:
  virtual HRESULT STDMETHODCALLTYPE FuncA() = 0;
};

MIDL_INTERFACE("00554d55-0000-0000-C000-000000000042")
IB : public IA {
 public:
  virtual HRESULT STDMETHODCALLTYPE FuncB() = 0;
};

class A : public IA {
 public:
  ~A() { std::cout << __FUNCTION__ << "\n"; }

  HRESULT STDMETHODCALLTYPE QueryInterface(
      /* [in] */ REFIID riid,
      /* [iid_is][out] */ _COM_Outptr_ void __RPC_FAR* __RPC_FAR* ppvObject) {
    if (__uuidof(IA) == riid || __uuidof(IUnknown) == riid) {
      *ppvObject = this;
      std::cout << __FUNCTION__ << "\n";
      AddRef();
      return S_OK;
    }
    return E_NOINTERFACE;
  }

  ULONG STDMETHODCALLTYPE AddRef(void) {
    ++ref_;
    std::cout << __FUNCTION__ << ": " << ref_ << "\n";
    return ref_;
  }

  ULONG STDMETHODCALLTYPE Release(void) {
    --ref_;
    std::cout << __FUNCTION__ << ": " << ref_ << "\n";
    if (0 == ref_) {
      delete this;
    }
    return ref_;
  }

  HRESULT STDMETHODCALLTYPE FuncA() {
    std::cout << __FUNCTION__ << "\n";
    return S_OK;
  }

 private:
  int ref_ = 0;
};

class B : public IB {
 public:
  ~B() { std::cout << __FUNCTION__ << "\n"; }

  HRESULT STDMETHODCALLTYPE QueryInterface(
      /* [in] */ REFIID riid,
      /* [iid_is][out] */ _COM_Outptr_ void __RPC_FAR* __RPC_FAR* ppvObject) {
    if (__uuidof(IB) == riid || __uuidof(IUnknown) == riid) {
      *ppvObject = this;
      std::cout << __FUNCTION__ << "\n";
      AddRef();
      return S_OK;
    }
    return E_NOINTERFACE;
  }

  ULONG STDMETHODCALLTYPE AddRef(void) {
    ++ref_;
    std::cout << __FUNCTION__ << ": " << ref_ << "\n";
    return ref_;
  }

  ULONG STDMETHODCALLTYPE Release(void) {
    --ref_;
    std::cout << __FUNCTION__ << ": " << ref_ << "\n";
    if (0 == ref_) {
      delete this;
    }
    return ref_;
  }

  HRESULT STDMETHODCALLTYPE FuncA() {
    std::cout << __FUNCTION__ << "\n";
    return S_OK;
  }

  HRESULT STDMETHODCALLTYPE FuncB() {
    std::cout << __FUNCTION__ << "\n";
    return S_OK;
  }

 private:
  int ref_ = 0;
};

HRESULT CreateObject(REFIID riid,
                     _COM_Outptr_ void __RPC_FAR* __RPC_FAR* ppvObject) {
  if (__uuidof(IA) == riid || __uuidof(IUnknown) == riid) {
    auto p = new A;
    p->QueryInterface(riid, ppvObject);
    return S_OK;
  } else if (__uuidof(IB) == riid) {
    auto p = new B;
    p->QueryInterface(riid, ppvObject);
    return S_OK;
  }
  return E_NOINTERFACE;
}

int main() {
  std::cout << "---- Raw pointer:\n";
  {
    IUnknown* u;
    HRESULT hr = CreateObject(__uuidof(IA), reinterpret_cast<void**>(&u));
    std::cout << hr << ", " << u << "\n";
    auto a = static_cast<IA*>(u);
    a->FuncA();
    a->Release();
  }
  std::cout << "---- ctor\n";
  {
    CComPtr<IUnknown> u;
    HRESULT hr = CreateObject(__uuidof(IUnknown), reinterpret_cast<void**>(&u));
    std::cout << hr << ", " << u << "\n";
    CComPtr<IUnknown> u2(u);  // will call AddRef
    CComQIPtr<IA> a(u);       // will call QueryInterface(call AddRef impliedly)
    CComQIPtr<IB> b(a);       // will call QueryInterface(call AddRef impliedly)
    a->FuncA();
  }
  std::cout << "---- CComPtr A=B:\n";
  {
    CComPtr<IUnknown> u;
    HRESULT hr = CreateObject(__uuidof(IB), reinterpret_cast<void**>(&u));
    std::cout << hr << ", " << u << "\n";
    CComPtr<IA> a;
    a = u;  // will call QueryInterface(call AddRef impliedly)
    if (a) {
      a->FuncA();
    }

    CComPtr<IB> b;
    b = u;
    if (b) {
      b->FuncA();
      b->FuncB();
    }

    // template<T, Q>
    a = b;  // failed
    if (a) {
      std::cout << "failed!\n";
      a->FuncA();
    }
  }
  std::cout << "---- CComQIPtr A=CComPtr<B>:\n";
  {
    CComPtr<IUnknown> u;
    HRESULT hr = CreateObject(__uuidof(IB), reinterpret_cast<void**>(&u));
    std::cout << hr << ", " << u << "\n";
    CComQIPtr<IA> a;
    a = u;  // will call QueryInterface(call AddRef impliedly)
    if (a) {
      a->FuncA();
    }

    CComPtr<IB> b;
    b = u;  // will call QueryInterface(call AddRef impliedly)
    if (b) {
      b->FuncA();
      b->FuncB();
    }

    a = b;  // will call AddRef
    if (a) {
      std::cout << "OK! B is-a A\n";
      a->FuncA();
    }
  }
  std::cout << "---- CComQIPtr A=CComQIPtr<B>:\n";
  {
    CComPtr<IUnknown> u;
    HRESULT hr = CreateObject(__uuidof(IB), reinterpret_cast<void**>(&u));
    std::cout << hr << ", " << u << "\n";
    CComQIPtr<IA> a;
    a = u;  // will call QueryInterface(call AddRef impliedly)
    if (a) {
      a->FuncA();
    }

    CComQIPtr<IB> b;
    b = u;  // will call QueryInterface(call AddRef impliedly)
    if (b) {
      b->FuncA();
      b->FuncB();
    }

    a = b;  // will call AddRef
    if (a) {
      std::cout << "OK! B is-a A\n";
      a->FuncA();
    }
  }
  std::cout << "---- CComPtr B=A:\n";
  {
    CComPtr<IUnknown> u;
    HRESULT hr = CreateObject(__uuidof(IUnknown), reinterpret_cast<void**>(&u));
    std::cout << hr << ", " << u << "\n";
    CComPtr<IA> a;
    a = u;  // will call QueryInterface(call AddRef impliedly)
    if (a) {
      a->FuncA();
    }

    CComPtr<IB> b;
    b = u;  // failed
    if (b) {
      b->FuncA();
      b->FuncB();
    }
  }
  std::cout << "---- CComQIPtr B=A:\n";
  {
    CComPtr<IUnknown> u;
    HRESULT hr = CreateObject(__uuidof(IUnknown), reinterpret_cast<void**>(&u));
    std::cout << hr << ", " << u << "\n";
    CComQIPtr<IA> a;
    a = u;  // will call QueryInterface(call AddRef impliedly)
    if (a) {
      a->FuncA();
    }

    CComQIPtr<IB> b;
    b = u;  // failed
    if (b) {
      b->FuncA();
      b->FuncB();
    }
  }
  std::cout << "---- CComQIPtr<IB>:\n";
  {
    CComQIPtr<IB> b;
    HRESULT hr = CreateObject(__uuidof(IB), reinterpret_cast<void**>(&b));
    std::cout << hr << ", " << b << "\n";
    b->FuncA();
    b->FuncB();
  }
  std::cout << "---- CComPtr<IA>:\n";
  {
    CComPtr<IA> a;
    HRESULT hr = CreateObject(__uuidof(IB), reinterpret_cast<void**>(&a));
    std::cout << hr << ", " << a << "\n";
    a->FuncA();
  }
  std::cout << "---- CComPtr<IUnknown>:\n";
  {
    CComPtr<IUnknown> u;
    HRESULT hr = CreateObject(__uuidof(IA), reinterpret_cast<void**>(&u));
    std::cout << hr << ", " << u << "\n";
    auto a = static_cast<IA*>(u.p);
    a->FuncA();
  }

  return 0;
}
```