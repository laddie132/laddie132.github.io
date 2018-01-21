---
title: "pytorch笔记"
date: "2018-01-21 00:00:00"
categories:
- python

tags: 
- pytorch
- python
---

对于初学者，在使用pytorch时，有一些值得注意的地方，记录如下：

## tensor和variable

- tensor表示张量：一般情况下，一维tensor叫向量，二维tensor叫矩阵，多维tensor叫张量
- variable表示变量：实际上，是一个tensor的高级封装，同时包含了梯度值，且同一个变量的梯度值是累计更新的，需要手动清零。默认设置`requires_grad=False`
- parameter表示参数：其实是variable 的一种，一般情况下用于模型参数变量，训练过程中，可以使用`parameters`统一收集模型的所有参数进行更新。默认设置`requires_grad=True`

## resize和view

使用中，可以发现resize和view均可以实现tensor的形状变换。但它们仍然有区别：resize调用中存在拷贝内存操作，即不改变原变量，在新内存空间创建新变量；而view直接在原内存空间进行变换。

## requires_grad和viloate

上文中提到了变量可以计算梯度，当然，对于某些预训练的参数，也可以手动设置不计算。`requires_grad`和`viloate`均可以实现。区别如下：

- requires_grad只限制当前节点梯度计算。该值取False时不需要计算梯度，不同变量运算后，该值取或
- viloate可以限制当前节点及子节点梯度计算。该值取True时不需要计算梯度，不同变量运算后，该取或