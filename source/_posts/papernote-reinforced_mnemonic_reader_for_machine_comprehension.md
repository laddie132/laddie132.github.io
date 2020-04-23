---
title: "论文分享 - Reinforced Mnemonic Reader for Machine Comprehension"
date: 2018-01-09 18:13:00
categories:
- Machine Comprehension

tags: 
- 论文分享
- 问答系统
- Machine Comprehension
- LSTM
- Attention
- NLP
- DL

mathjax: true
---

这篇论文发表时间比较近，比较全面地总结了match-LSTM、R-Net等众多前人模型的优缺点，并做了很好的改进，如：增加编码层能力，解决长距离上下文信息，提炼预测答案片段，直接优化评价函数等，在SQuAD数据库上取得了State-Of-Art的效果。
<!-- more -->

## 简介
前人的很多模型都具有一个共同的网络框架，即“encoder-interaction-pointer”。首先是将问题和段落的单词序列利用RNN网络编码为向量，接着利用Attention机制捕捉问题和段落之间的关系，最后利用指针网络预测答案的界限。

但是，这种框架具有很明显的缺点：
1. 编码层，可能缺失部分词汇、句法信息，如词性、命名实体等都十分有用
2. 交互层，使用LSTM或者GRU仍然无法全部捕捉长距离的上下文信息
3. 指针层，a.多个候选答案无法处理，b.答案过长，过复杂无法清晰定义边界

该文提出以下解决方案：
1. 编码层融入词汇、句法特征：是否完全匹配、词性POS、实体标签NER、问题类别
2. 交互层增加passage的自对齐，即passage和passage做Attention，获得fully-aware context representation（*这个有点类似R-NET的self-matching机制*）
3. 设计了一种基于记忆的答案抽取网络，可以持续增加阅读知识的同时，不断提取答案片段
4. 训练时，结合极大似然估计的交叉熵损失函数和强化学习中的奖励机制


## 模型

<img src="/images/mnemonic-model.png" style="zoom:40%" />

这里使用（Q，C，A）表示一个样例，即问题为$Q=\{w_i^q\}_{i=1}^n$，段落为$C=\{w_j^c\}_{j=1}^m$，答案为A

### 富特征编码层

一个特征向量主要包括三部分：word-level、character-level和额外特征

#### 混合编码
使用BiLSTM分别对段落和问题做word-level和character-level编码，得到向量$x_w$和$x_c$，拼接为$x=[x_w;x_c]$

#### 额外特征

1. 完全匹配（EM）特征：描述问题中的词语是否存在于段落中，或者段落中词语是否存在于问题中
2. POS、NER：可以用查字典的方式得到每一个词语的标签，并追加到问题和段落向量中
3. 问题类型：主要有“what, how, who, when, which, where, why, be, other”等，将类别标签表示为一个向量，只追加到问题向量中

处理完，获得了一个问题向量$\{\tilde{x}_i^q\}_{i=1}^n$，和段落向量$\{\tilde{x}_j^c\}_{j=1}^m$

#### 编码

使用两个BiLSTM网络编码，如下：
<img src="/images/mnemonic-encoder.png" style="zoom:40%" />

这样一来，可以得到问题描述：$Q^{'}=\{q_i\}_{i=1}^n$，段落描述：$C^{'}=\{c_j\}_{j=1}^m$

### 迭代对齐层

之所以成为迭代，是因为采用了多跃点机制，相当于将一个aligner封装为一个unit，将该unit水平维度上依次连接，起到迭代的效果，迭代次数为T

#### 对齐

在第t个hop中，问题和段落联合attention向量$B_{ij}^t=q_i^T \cdot \check{c}_j^{t-1}$，其中，$\check{c}_j^{t-1}$表示上一个hop的输出中第j个段落单词表示，初始值$\check{c}_j^0=c_j$。

从该公式中，也能看出若干个hop依次连接。变量$B_{ij}^t$即表示了问题中第i个单词与段落中第j个单词的关联程度。

用$b_j^t$表示段落中第j个单词在问题上attention分布的归一化向量，$\{\tilde{q}_j^t\}_{j=1}^m$，表示每一个段落单词的问题attention向量，如下所示：

<img src="/images/mnemonic-aligner.png" style="zoom:40%" />

一般情况下，论文计算到这块就已经完成了问题和段落的关联表示，即quare-aware context representation，但该文将上一个hop输出的段落表示$\check{c}_j^{t-1}$和这个attention向量结合起来，具体方法使用了一种语义融合单元（semantic fusion unit）。

<img src="/images/mnemonic-aligner-sfu.png" style="zoom:40%" />

#### 语义融合单元（Semantic Fustion Unit）

这是该文的一个创新点，提出了一种新的向量融合方法，下面我们剖析公式可以发现，实质上还是一种门机制的引入。

单元接受一个输入向量$r$和一个待融合向量集合$\{f_i\}_{i=1}^k$，输出一个向量$o$，如下：

<img src="/images/mnemonic-sfu.png" style="zoom:40%" />

#### 自对齐机制

前面是将段落与问题利用attention机制进行对齐关联，这里为了获取到段落中更长距离的依存信息，将段落与段落自身进行对齐关联，

> 该方法在MSRA论文R-NET中也有使用，读者可参考。

与上文相似，首先计算一个联合Attention向量$\check{B}_{ij}^t$，如下：

<img src="/images/mnemonic-self-aligner.png" style="zoom:40%" />

这里，如果$i==j$，即相同位置，则关联度设置为0；之后，仍然按照上述方法计算注意力段落向量$\{\tilde{c}_j^t\}_{j=1}^m$；然后，还会包括一个SFU单元与之前的quare-aware context representation$\bar{c}_j^t$进行向量融合，得到一个self-aware context representation$\hat{c}_j^t$

#### 收集

每一个hop的最后，都会有一个收集单元，主要用于捕捉段落中的短距离信息。结构可以直接套用之前的编码层结构，即一个BiLSTM层。输入为之前自对齐产生的self-aware context representation向量，输出为fully-aware context representation$\check{c}_j^t$。

### 基于记忆的答案指针层

由于使用SQuAD数据库，答案只需要指定起始位置和终止位置即可。

这个层仍然使用multi-hop机制，包含L个hops。在第l个hop，输入fully-aware context representation$\check{c}_i^t$和起始位置记忆向量$z_s^l$，生成起始位置的概率分布$p_s^l$，如下：

<img src="/images/mnemonic-ptr-net.png" style="zoom:40%" />

其中，FN指前向神经网络。记忆向量$z_s^l$初始化使用一个问题相关的向量，实验中发现，直接使用问题编码层最后一个隐含层状态向量效果最好，即$\vec{q}$。之后会利用上一个hop进行更新。

有了起始位置的概率分布$p_s^l$，可以得到一个evidence向量$u_s^l=\hat{C}^T \odot p_s^l$描述，包含了当前起始位置预测的信息，将它和之前的起始位置记忆向量$z_s^l$做融合，得到终止位置记忆向量$z_e^l=SFU(z_s^l,u_s^l)$。之后，和得到起始位置概率分布的方法相同：

<img src="/images/mnemonic-ptr-net2.png" style="zoom:40%" />

这里得到$p_e^l$即代表终止位置概率分布，仍然按照相同方法得到一个evidence向量$u_e^l=\hat{C}^T \odot p_e^l$，这个向量最后会输入到下一个hop中，当做起始位置记忆向量$z_s^{l+1}$。

如此一次迭代若干个hop。

## 训练步骤

### 极大似然估计

之前诸如match-LSTM等论文中都采用了对数损失函数，即极大似然估计的方法，公式如下：

<img src="/images/mnemonic-mle.png" style="zoom:40%" />

局限性：
1. 这种函数以优化完全匹配（EM）为目标，无法直接提升F1分数
2. 另外，算法不稳定，在问题答案较长，边界模糊时候，无法正确划分。因此，很难解决“为什么”类型问题

### 强化学习

将原评价指标中的F1分数作为强化学习的奖励函数，设计损失函数如下：

<img src="/images/mnemonic-rl.png" style="zoom:40%" />

为了保证训练稳定性以及防止早期过拟合，该文将之前的MLE方法和RL方法结合起来，即同时优化EM和F1值，有了如下的损失函数：

<img src="/images/mnemonic-loss.png" style="zoom:40%" />

## 实验结果

在SQuAD中取得了State-Of-Art的效果，如下：

<img src="/images/mnemonic-result.png" style="zoom:40%" />
