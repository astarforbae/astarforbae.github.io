---
layout: post
title: "记录一次DEUBG经历 | 解决：Expected all tensors to be on the same device, but found at least two devices"
subtitle: ""
date: 2024-08-02
mathjax:  true
author: "Yi"
header-img: "img/post-bg-2015.jpg"
tags: [DEBUG,RL]
---

## 问题描述

这次报错出现在DQN基础上加入Noisy Network的时候，具体的报错如下：

```txt
RuntimeError: Expected all tensors to be on the same device, but found at least two devices, cuda:0 and cpu!
```

报错的代码是forward中计算weight的代码

```python
def forward(self, x):
    self.reset_noise()

    if self.training:
        epsilon_weight = self.epsilon_j.ger(self.epsilon_i)
        epsilon_bias = self.epsilon_j
        weight = self.mu_weight + self.sigma_weight.mul(
            torch.autograd.Variable(epsilon_weight)
        )
        bias = self.mu_bias + self.sigma_bias.mul(
            torch.autograd.Variable(epsilon_bias)
        )
    else:
        weight = self.mu_weight
        bias = self.mu_bias
    y = F.linear(x, weight, bias)
    return y
```

## 解决方案

经过调试，发现是在使用GPU进行训练时，`self.sigma_weight`是在GPU上的，而`epsilon_weight`是在CPU上，两者类型不一致才导致了报错。

那么我就很疑惑为什么`epsilon_weight`为什么是在CPU上的。

从上方的代码可以知道：

```python
epsilon_weight = self.epsilon_j.ger(self.epsilon_i)
```

这一步说明`epsilon_weight`是两个向量的外积，那理应类型是和这两个向量一致，所以再回去看这两个变量的定义。

在`__init__`函数中发现：

```python
self.register_buffer("epsilon_i", torch.FloatTensor(num_in))
self.register_buffer("epsilon_j", torch.FloatTensor(num_out))
```

首先我们来看`register_buffer`的作用是什么：
通常，如果有一些参数需要被保存，但是不需要被训练，那么就可以使用`register_buffer`来保存这些参数。

那`register_buffer`中的参数存放的位置是否和`model`一致呢？答案是肯定的。

我们可以通过一段代码进行测试：

```python
import torch
from torch import nn


class TestModule(torch.nn.Module):
    def __init__(self):
        super(TestModule, self).__init__()
        self.register_buffer("buffer", torch.FloatTensor(10))
        self.fc = nn.Linear(10, 10)

    def forward(self, x):
        return self.fc(x)


module = TestModule()
module.to("cuda")
print(module.buffer.device)
module.to("cpu")
print(module.buffer.device)
```

第一个输出是`cuda:0`，第二个输出是`cpu`，说明`register_buffer`中的参数是会随着`model`的迁移而迁移的。

`register_buffer`的问题排除了之后，我又去查看其他队`epsilon_i`进行了改动的地方，找到了：

```python
def reset_noise(self):
    eps_i = torch.randn(self.num_in)
    eps_j = torch.randn(self.num_out)
    self.epsilon_i = eps_i.sign() * (eps_i.abs()).sqrt()
    self.epsilon_j = eps_j.sign() * (eps_j.abs()).sqrt()
```

从这个地方发现了问题：`eps_i = torch.randn(self.num_in)`没有调用`to(device)`方法，导致`eps_i`和`eps_j`都是在CPU上的。进而导致了`epsilon_i`和`epsilon_j`也是在CPU上的。
那么一个简单的解决方案就是在先获得device，再进行`eps_i`和`eps_j`的计算

这里我是用模型中的一个参数的device来进行判断的，代码为：

```python
def reset_noise(self):
    device = self.sigma_weight.device
    eps_i = torch.randn(self.num_in).to(device)
    eps_j = torch.randn(self.num_out).to(device)
    self.epsilon_i = eps_i.sign() * (eps_i.abs()).sqrt()
    self.epsilon_j = eps_j.sign() * (eps_j.abs()).sqrt()
```

至此，问题解决。

## 总结

这次Bug的出现反映出了我对`PyTorch`代码的不熟悉，以及对代码的不仔细。
Debug的过程中也有比较多收获，至少对`register_buffer`和`device`有了更深刻的理解。
