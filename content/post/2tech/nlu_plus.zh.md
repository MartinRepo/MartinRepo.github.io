---
title: "从RNN到Transformers"
date: 2025-02-01T10:39:49Z
draft: false
author: ["Martin"]
tags: 
- NLP
description: ""
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: "nluplus"
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示当前路径
cover:
    image: "/img/nluplus/whatisrnn.jpg"
    caption: ""
    alt: ""
    relative: true
mermaid: true
---
# 递归神经网络
简单的神经网络由三部分组成：输入，隐藏层，输出

# RNN变体
## LSTM
## GRU

# Transformers
## Self-Attention
> The fundamental operation of any transformer architecture is the self-attention operation [^1].

自注意力机制是一个从序列到序列的操作，它的输入是一组向量 $x_1, x_2, ..., x_t$，对应的输出序列是 $y_1, y_2, ..., y_t$。（每个向量都是k维）

每一个 $y_i$ 是怎么计算的？$y_i$是根据当前位置的$x_i$和其他所有位置的$x_j$的相关性计算出来的，具体公式如下
$$y_{\color{red}{i}} = \sum_{\color{blue}{j}} w_{\color{red}{i}\color{blue}{j}} x_{\color{blue}{j}}$$

对于$w_{\color{red}{i}\color{blue}{j}}$, 它是由$x_{\color{red}{i}}, x_{\color{blue}{j}}$的点积函数推导出来的。
$$w_{\color{red}{i}\color{blue}{j}} = \text{softmax}(w^{\prime}_{\color{red}{i}\color{blue}{j}})$$

其中
$$w^{\prime}_{\textcolor{red}{i} \textcolor{blue}{j}} = x^{T}_{\textcolor{red}{i}} x_{\textcolor{blue}{j}}$$

原理可视化如下图所示，值得注意的是，这是整个体系结构中唯一在向量之间传播信息的操作。
![self-attention](/img/nluplus/self-attention.png)

为什么self-attention机制如此有效？

# References
[^1]: ‘Transformers from scratch | peterbloem.nl’. Accessed: Feb. 01, 2025. [Online]. Available: https://peterbloem.nl/blog/transformers
