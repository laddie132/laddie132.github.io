---
title: pytorch中损失函数
date: 2018-03-08 13:13:50

categories:
- Deep Learning

tags:
- loss
- DL
- pytorch
- python
---

常见损失函数有：均方误差损失函数（MSE）、交叉熵损失函数、对数似然损失函数等。一般情况下，对于回归问题，采用均方误差损失函数；对于分类问题，采用交叉熵或对数似然损失函数。

后面这两种损失函数在大部分情况下并无差别，只是因为推导方式不同，而叫法不同。对数似然损失函数的思想是极大化似然函数，而交叉熵则是从信息熵的角度考虑。
<!-- more -->

> 具体介绍可以参考logistic回归和softmax回归原理。

但在pytorch中，提供的`CrossEntropyLoss`和`NLLoss`两个损失函数用法有些特殊。

1. `NLLoss`

<img src="/images/loss_nll.png" style="zoom:40%" />

需要使用者手动输入一个对数概率值，即该函数不提供对数运算。

2. `CrossEntropyLoss`

<img src="/images/loss_cel.png" style="zoom:40%" />

这里的交叉熵相当于`LogSoftMax`和`NLLLoss`的组合，即在传统交叉熵的基础上，进一步引入了softmax回归