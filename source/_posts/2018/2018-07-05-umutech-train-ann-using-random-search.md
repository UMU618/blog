---
layout: post
title: 人工神经网络训练方法——随机查找
date: 2018-07-05 23:45:52
categories: UMUTech
tags:
- cpp
- machine-learning
---

《[人工神经网络究竟是什么鬼？](/2018/06/05/ann/)》中没有讲到如何训练神经网络，本篇延续用 XOR 运算为例，介绍一种随机查找的训练方式，主要原理是：随机初始化 w，计算错误率，在循环中，保存错误率小的 w，直到错误率小于等于 0.01 为止。

代码不会骗人，简单的实现如下：

```
// TrainXor_RandomSearch.cpp
// UMUTech @ 2018-07-05 23:45:52
// Be aware that I'm only a novice to ANN. My apologies for any wrong info.
//
#include <algorithm>
#include <iostream>
#include <random>

std::default_random_engine random_engine;

void RandomizeW(double* w, size_t size) {
  std::uniform_real_distribution<double> r(0, 1);
  for (size_t i = 0; i < size; ++i) {
    w[i] = r(random_engine);
  }
}

void PrintW(double* w, size_t size) {
  for (size_t i = 0; i < size; ++i) {
    std::cout << i << "\t" << w[i] << "\n";
  }
}

double ActivationFunction(double x) {
  // ReLU
  return std::max(0.0, x);
}

double AnnRun(const double x[2], double* w) {
  // bias 乘了 -1，让结果更好地收敛到 [0, 1]
  double f = ActivationFunction(x[0] * w[0] + x[1] * w[1] - w[2]);
  double g = ActivationFunction(x[0] * w[3] + x[1] * w[4] - w[5]);
  return ActivationFunction(f * w[6] + g * w[7] - w[8]);
}

int main() {
  const double input[4][2] = {{0, 0}, {0, 1}, {1, 0}, {1, 1}};
  const double expect_output[4] = {0, 1, 1, 0};

  double last_error = 1000;

  double w[3 * 3];
  double w_copy[3 * 3];

  std::random_device rd;
  random_engine.seed(rd());

  int train_count = 0;
  for (; last_error > 0.01; ++train_count) {
    if (train_count % 10000 == 0) {
      std::cout << "Randomize\n";
      RandomizeW(w, _countof(w));
    }

    memcpy(w_copy, w, sizeof(w));

    // 随机改变 w
    std::uniform_real_distribution<double> r(-0.5, 0.5);
    for (int i = 0; i < 3 * 3; ++i) {
      w[i] += r(random_engine);
    }

    double error = pow(AnnRun(input[0], w) - expect_output[0], 2.0);
    error += pow(AnnRun(input[1], w) - expect_output[1], 2.0);
    error += pow(AnnRun(input[2], w) - expect_output[2], 2.0);
    error += pow(AnnRun(input[3], w) - expect_output[3], 2.0);

    if (error < last_error) {
      // 错误率更小，保存
      last_error = error;
    } else {
      // 恢复 w
      memcpy(w, w_copy, sizeof(w));
    }
  }

  printf("Finished in %d loops.\n", train_count);

  PrintW(w, _countof(w));

  /* Run the network and see what it predicts. */
  printf("Output for [%1.f, %1.f] is %1.f.\n", input[0][0], input[0][1],
         AnnRun(input[0], w));
  printf("Output for [%1.f, %1.f] is %1.f.\n", input[1][0], input[1][1],
         AnnRun(input[1], w));
  printf("Output for [%1.f, %1.f] is %1.f.\n", input[2][0], input[2][1],
         AnnRun(input[2], w));
  printf("Output for [%1.f, %1.f] is %1.f.\n", input[3][0], input[3][1],
         AnnRun(input[3], w));

  return 0;
}
```

效果主要看人品，可能跑个不停，也可能几乎立刻完成。一次运行结果：

```
Randomize
Finished in 344 loops.
0       -1.18943
1       -1.60685
2       -0.848489
3       1.28751
4       1.21697
5       0.532657
6       -2.27322
7       -0.77646
8       -1.57966
Output for [0, 0] is 0.
Output for [0, 1] is 1.
Output for [1, 0] is 1.
Output for [1, 1] is 0.
```

另一次：

```
Randomize
Finished in 444 loops.
0       1.6138
1       1.4345
2       1.33925
3       1.50895
4       1.09461
5       -0.283878
6       -2.37528
7       1.08117
8       0.239626
Output for [0, 0] is 0.
Output for [0, 1] is 1.
Output for [1, 0] is 1.
Output for [1, 1] is 0.
```
