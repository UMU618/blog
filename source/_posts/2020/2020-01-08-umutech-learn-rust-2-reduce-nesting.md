---
layout: post
title: 学习 Rust【2】减少代码嵌套
date: 2020-01-08 17:59:01
categories: UMUTech
tags:
- dev
- rust
- optimization
---
结论先行：**减少代码嵌套就是降低复杂度。**

资源管理一向是编程中的重要任务。当一个函数要管理多个资源时，很容易出现代码嵌套层级太深的问题，尤其是调用系统或第三方 API 时。

以 C 语言代码为例，这里简化为两个资源，请您自行脑补多个资源：

```c
int error_code = 0;
resource1 *p1 = new_resource1();
// UMU: with C++ SHOULD be `p1 != nullptr`
if (p1) {
  resource2 *p2 = new_resource2();
  if (p2) {
    if (!deal_resources(p1, p2)) {
      error_code = 3;
    }
    free_new_resource2(p2);
  } else {
    error_code = 2;
  }
  free_new_resource1(p1);
} else {}
  error_code = 1;
}

return error_code;
```

上面代码最深嵌套是三层，为了减少嵌套，可以把代码改为平坦结构，降低到一层：

```c
resource1 *p1 = new_resource1();
if (!p1) {
  free_new_resource1(p1);
  return 1;
}

resource2 *p2 = new_resource2();
if (!p2) {
  free_new_resource1(p1);
  free_new_resource2(p2);
  return 2;
}

if (!deal_resources(p1, p2)) {
  free_new_resource1(p1);
  free_new_resource2(p2);
  return 3;
}

free_new_resource1(p1);
free_new_resource2(p2);
```

但这么改在资源释放时，**更容易遗漏**。也有人为使代码层级平坦化，会使用 `goto` 到函数末尾统一释放，或者更优雅点的 C++ 方式：用 `try...throw...catch...finally` 将所有资源包含起来管理。

Node.js 的异步回调函数也存在嵌套层级过深的问题，可以用 Promise 来平坦化，参考：

```js
setTimeout(() => {
  console.log('step1')
  setTimeout(() => {
    console.log('step2')
    setTimeout(function() {
      console.log('step3')
      console.log('done!')
    }, 1000)
  }, 1000)
}, 1000)

// flatten
let timer = (text) => {
  let promise = new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log(text)
      resolve()
    }, 1000)
  })

  return promise
}

timer("step1")
  .then(() => {
    return timer("step2")
  })
  .then(() => {
    return timer("step3")
  })
  .then(() => {
    console.log("done!")
  })
```

C++ 建议使用 RAII 思想来管理资源，获得资源后立刻放到管理对象里。如果有些资源使用得不频繁，想偷懒不去封装，则可以使用 [scope_exit](/2020/09/22/umutech-boost-2-scope-exit/)。go 语言更是用内置关键字 `defer` 来提供 scope_exit 机制。

Rust 用 scopeguard 提供 scope_exit 机制，`defer!` 宏和 go 的 `defer` 功能类似。

另外，Rust 还有 ? 操作符，也有减少嵌套的作用。比如这个任务：打开文件，如果失败就返回错误。go 是这样写的：

```go
package main

import (
	"os"
)

func main() {
	file, error := os.Open("file.txt")
	if error != nil {
		panic(error)
	}
	defer file.Close()
}
```

同样功能，Rust 代码少一层：

```rust
use std::fs::File;

fn main() -> std::io::Result<()> {
  let _f = File::open("file.txt")?;
  Ok(())
}
```
