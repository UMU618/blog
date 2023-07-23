---
layout: post
title: 人工神经网络训练方法——后向传播
date: 2018-07-10 21:00:23
categories: UMUTech
tags:
- cpp
- machine-learning
---

《[人工神经网络训练方法——随机查找](/2018/07/05/umutech-train-ann-using-random-search/)》介绍的随机查找方法，有点盲人摸象，所以继续介绍主流的后向传播(BackPropagation)算法。

## 填坑

先给随机查找做个优化！上篇中的激活函数统一使用 ReLU，其实这是不好的，输出层可以改为 Sigmoid 或 Tanh：

```
inline double ActivationFunction_ReLU(double x) {
  return std::max(0.0, x);
}
inline double ActivationFunction_Sigmoid(double x) {
  return 1.0 / (1 + exp(-x));
}
inline double ActivationFunction_Tanh(double x) {
  return (tanh(x) + 1.0) / 2;
}

double AnnRun(const double x[2], double* w) {
  double f = ActivationFunction_ReLU(x[0] * w[0] + x[1] * w[1] - w[2]);
  double g = ActivationFunction_ReLU(x[0] * w[3] + x[1] * w[4] - w[5]);
  return ActivationFunction_Sigmoid(f * w[6] + g * w[7] - w[8]);
}
```

原因很简单，我们已经知道 Xor 的结果不是 0 就是 1，用 ReLU 是可能大于 1 的，而 Sigmoid 和 Tanh 不会大于 1。

## 后向传播

理论学习：[《如何直观地解释 back propagation 算法？》](https://www.zhihu.com/question/27239198)

原理：求导

训练时，x 和 y 都是固定的，要求的是 a 和 b，所以问题是：当 y 偏离了 delta_y，求 a 和 b 应该修正多少？

分别对 a 和 b 求偏导，则:

~~~
dy/da = x
dy/db = 1
~~~

所以

~~~
delta_a = delta_y / x
delta_b = delta_y
~~~

代码不会骗人，来一个简化的例子：

```
// BackPropagation.cpp
//

#include <iostream>

void Train(double& a,
           double& b,
           double input,
           double expect_output,
           double learning_rate) {
  double delta_y = expect_output - (input * a + b);
  if (input != 0) {
    a += (delta_y / input) * learning_rate;
  }
  b += delta_y * learning_rate;
}

int main() {
  // 要求的函数是：y = 2 * x + 3
  const double input[4] = {0, 1, 2, 3};
  const double expect_output[4] = {3, 5, 7, 9};

  // 初始化状态是：y = 1 * x + 4
  double a = 1.0;
  double b = 4.0;

  std::cout << "Initial: y = " << a << " * x + " << b << "\n";

  // 两轮就搞定了
  for (int t = 0; t < 2; ++t) {
    for (int i = 0; i < 4; ++i) {
      Train(a, b, input[i], expect_output[i], 1);
    }
  }
  std::cout << "Trained: y = " << a << " * x + " << b << "\n";

  return 0;
}
```
