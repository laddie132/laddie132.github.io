---
title: "论文分享 - R-Net: Machine Reading Comprehension with Self-Matching Networks"
date: 2018-01-09 12:09:00
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

## 介绍

该文由MSRA发表，在SQuAD数据库上目前成绩最好。模型借鉴了Wang&Jiang最早的match-LSTM方法，做了一些改进，网络结构分为以下四部分：

1. RNN网络分别对question和passage单独编码
2. 基于门限的注意力循环神经网络（gated-attention based recurrent network）匹配question和passage，获取问题的相关段落表示（question-aware passage representation）
3. 基于自匹配注意力机制的循环神经网络（self-matching attention network），将passage和它自己匹配，从而实现整个段落的高效编码
4. 基于指针网络（pointer-network）定位答案所在位置

> 部分方法可参考Wang&Jiang论文《Machine Comprehension Using Match-LSTM and Answer Pointer》

## 问题介绍

在SQuAD数据库中，一个case为可以用一个三元组表示（P，Q，A）：P表示段落（passage），Q表示问题（Question），A表示答案（Answer）。另外，答案通常是在段落中连续出现，且答案较多样化，包含非命名实体以及长短语等特殊情况。

## R-Net模型

<img src="/images/r-net-model.png" style="zoom:35%" />

### passage和question编码层

输入问题$Q={w_t^Q}_{t=1}^m$和段落$P={w_t^P}_{t=1}^n$，分别进行word-level编码和character-level编码，得到向量$e$和$c$。这里character-level编码主要是为了应对OOV的影响，以往OOV词向量直接就是0，这里可以缓和OOV的影响。之后，利用两个双向RNN网络分别对question和passage再编码。另外，作者在这里选用了GRU单元，而不是LSTM，原因在于GRU计算量更小。

<img src="/images/r-net-encoder.png" style="zoom:35%" />

### 门限注意力RNN层

首先，仍然是match-LSTM模型中的attention关联层，$c_t=att(u^Q,[u_t^P,v_{t-1}^P])$是question的attention向量，如下：

<img src="/images/r-net-attention1.png" style="zoom:35%" />

<img src="/images/r-net-attention2.png" style="zoom:35%" />

之后是门机制的引入，主要用于控制passage表示的不同部分重要性，如下：

<img src="/images/r-net-gated.png" style="zoom:35%" />

之后，$[u_t^P,c_t]^*$将代替$[u_t^P,c_t]$继续传播。

### 自匹配注意力RNN层

上面的问题注意力段落表示（question-aware passage representation）有一些局限性：只能获取到有限的段落上下文信息。很多时候，段落和问题存在句法和词汇上的区别，段落的上下文信息将更能有效帮助模型找到正确答案。

这一层与上面的门限注意力层大体结构相同，只不过将输入变为$v^p$和$v^p$，$c_t=att(v^P,v_t^P)$是passage（$v^p$）的attention向量，如下：

<img src="/images/r-net-self-matching1.png" style="zoom:35%" />
<img src="/images/r-net-self-matching2.png" style="zoom:35%" />

之后和gated-attention层同样引入门机制，自适应控制不同部分的重要性。

### 答案输出层

这部分与之前的match-LSTM等论文相同，使用Pointer-Network网络，即直接将attention向量当做候选位置的概率，共输出两个位置：答案起始和结束。

<img src="/images/r-net-ptr-net1.png" style="zoom:35%" />
<img src="/images/r-net-ptr-net2.png" style="zoom:35%" />

唯一一点区别是初始化，即在预测答案起始位置的时候，$h_{t-1}^a$从何处来？一般应该是随机初始化的，这里使用passage attention机制生成一个question向量$r^Q=att(u^Q,V_r^Q)$，其中，$V_r^Q$是一个可变参数，如下：

<img src="/images/r-net-ptr-net3.png" style="zoom:35%" />

至于损失函数，与match-LSTM论文中一样，使用对数损失函数。

## 实验

无论是single model，还是ensemble model，R-NET都取得了至今为止最好的效果。

<img src="/images/r-net-result.png" style="zoom:30%" />

### 讨论

1. 语句排名（Sentence Ranking）。实验发现，段落总是包含多个句子，并且答案往往在一个句子中，故而考虑能否引入句子排名机制，先初步筛选出可能性最大的句子，再从对应句子中进一步定位答案。作者尝试了两种方法：1. 单独训练一个句子排名模型；2. 将句子排名模型和片段抽取模型关联起来，训练一个联合模型。但这两种方法效果都较差，并未达到预期。这或许说明额外的句子信息仍然对答案抽取有帮助。

2. 句法信息（Syntax Information）。实验尝试引入句法信息到编码层，如：增加词性、命名实体、依赖关系等特征到编码层；之后，引入新的tree-LSTM模型，自顶向下和自底向上构建网络。实验证明，这种方法并未对当前模型有任何增益。

3. 多跃点推理（Multi-hop Inference）。尝试在答案指针层引入multi-hop推理机制，但无效果提升。作者认为可能是模型过于复杂，问题无法有效地学习。
> 注：在论文《Reinforced Mnemonic Reader for Machine Comprehension》中，整个模型都引入了multi-hop机制，可以参考

4. 问题生成（Question Generation）。深度神经网络是一种数据驱动的方法，数据质量和数据大小对于最后的结果具有至关重要的作用，作者尝试手动生成一些假的问题和答案对，与SQuAD数据库共同训练，但最后并未有任何效果提升。分析证明，这些自动生成的数据质量仍然有待提高。
