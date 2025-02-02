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

深度神经网络（DNN）一般是增加隐藏层的数量。

递归神经网络（RNN）更加关注的是隐藏层中每个神经元在时间上的成长与进步。

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
$$w^{\prime}_{\color{red}{i}\color{blue}{j}} = x^T_ix_j$$

原理可视化如下图所示，值得注意的是，这是整个体系结构中唯一在向量之间传播信息的操作。
![self-attention](/img/nluplus/self-attention.png)

为什么self-attention机制如此有效？我们拿电影推荐系统来举例子。假设你经营一家电影租赁公司，你有一些电影和一些用户，你想向你的用户推荐他们可能会喜欢的电影。一种方法是为电影手动创建特征，例如电影中有多少浪漫情节，有多少动作场面，然后为用户设计相应的特征：他们有多喜欢浪漫电影，有多喜欢动作片。如果这样做，两个特征向量之间的点积就会为电影属性与用户喜好的匹配程度打分。如果用户和电影的某个特征的符号相匹配--电影很浪漫，用户喜欢浪漫，或者电影不浪漫，用户讨厌浪漫--那么得到的点积就会得到该特征的正值。如果符号不匹配--电影浪漫而用户讨厌浪漫，则相应项为负值。
![movie_recommendation](/img/nluplus/movie_recommendation.png)

当然这样做并不现实，因为这涉及到大量的数据标注工作。一种解决办法是我们将电影特征和用户特征作为模型的参数，让用户选择一些喜欢的电影，然后通过优化算法调整用户特征和电影特征，使它们的点积能够匹配用户的已知喜好。即使我们没有告诉模型这些特征的具体含义，但在训练后，这些特征往往会自发地映射到有意义的电影内容信息。 例如，某个维度可能对应“浪漫”，另一个维度可能对应“动作”或“喜剧”，即使我们事先并未明确这些维度的含义。

这就是self-attention机制。它本质上是一种计算序列内部关系的方法。在处理文本时，我们首先为每个单词分配一个可学习的**嵌入向量（embedding vector）**，例如句子 "the cat walks on the street" 会被映射为向量序列 $ v_{\text{the}}, v_{\text{cat}}, v_{\text{walks}}, v_{\text{on}}, v_{\text{the}}, v_{\text{street}} $。然后，这些向量进入self-attention layer，输出新的向量序列 $ y_{\text{the}}, y_{\text{cat}}, y_{\text{walks}}, y_{\text{on}}, y_{\text{the}}, y_{\text{street}} $。其中，每个 $ y_t $ 都是输入向量的加权和，而权重是根据**归一化 (softmax) 的点积（dot product）** 计算得到的。点积衡量了单词之间的相关性，如果两个单词在上下文中紧密相关，它们的点积就会较高，从而获得更高的注意力权重。例如，在句子中，“walks” 这个动词需要与“cat”建立联系，所以 $ v_{\text{walks}} $ 和 $ v_{\text{cat}} $ 的点积会较高，而“the” 这样的冠词由于对整体语义影响较小，通常不会与其他词有很大的点积值。

有趣的是，基础的self-attention并不关心输入的顺序，它只关注单词之间的相对关系。如果我们对输入序列进行重新排列，输出的向量序列也会以相同的方式被调整，而不会改变内容本身。这意味着自注意力本质上是一种**集合操作（Set Operation）**，它不会依赖词的前后顺序，而是让模型自己学习哪些单词之间应该有更强的联系。此外，基础的自注意力层没有可训练参数，它的行为完全取决于输入向量，而这些向量来自于嵌入层，而嵌入层是可学习的。在完整的 Transformer 结构中，我们会通过**位置编码（Positional Encoding）** 让模型感知单词的顺序信息，但在纯自注意力层中，它是**完全顺序不敏感**的。

## Query, Key, Value
当代transformer中使用的self-attention还依赖三个额外的技巧 - Queries, Values, Keys。在 **自注意力** 计算中，每个输入向量 $ x_i $ 需要在三种不同的角色之间切换：
1. **查询（Query，$ q_i $）**：每个词向量都需要与其他词向量进行比较，以确定它对自己的输出 $ y_i $ 的影响权重。
2. **键（Key，$ k_i $）**：每个词向量也被其他单词用来计算权重，即它对其他单词输出 $ y_j $ 的影响。
3. **值（Value，$ v_i $）**：在计算完权重后，每个单词的最终表示是所有词向量的加权和，其中加权方式由查询和键计算得到的注意力权重决定。

为了更方便计算，我们不会直接使用原始输入向量 $ x_i $，而是对其进行三次不同的 **线性变换**，得到：
$$
q_i = W_q x_i, \quad k_i = W_k x_i, \quad v_i = W_v x_i
$$
其中 $ W_q, W_k, W_v $ 是可训练的 **权重矩阵**，用于学习适合任务的注意力表示。

接下来，计算两个词之间的相似性（相关性）:
$$
w' = q_i^T k_j
$$
然后进行 **softmax 归一化** 以获得最终的注意力权重：
$$
w_{ij} = \text{softmax}(w')
$$
最后，每个单词的输出向量$ y_i $是所有值向量的加权和：
$$
y_i = \sum_j w_{ij} v_j
$$
这个过程让模型能够动态调整单词之间的相互影响，从而更好地理解序列中的语义关系。

![movie_recommendation](/img/nluplus/qvk.png)
## 点积缩放
刚才我们使用 **dot product** 计算查询（Query）和键（Key）之间的相似性，并将其输入 **Softmax** 计算注意力权重。然而，当**嵌入维度 $ k $ 较大时**，点积的值会变得很大，导致 **Softmax 的梯度变得极小，甚至造成梯度消失，影响模型训练**。

为了解决这个问题，我们对点积进行缩放 **（scaling dot product）**，即除以 $ \sqrt{k} $: 
$$
w'_{ij} = \frac{q_i^T k_j}{\sqrt{k}} 
$$
从而防止输入值过大，使 softmax 函数输出的梯度保持稳定，提高训练效率。这样缩放的原因在于 **高维向量的欧几里得长度随着维度 $ k $ 增大而增大**，通过除以 $\sqrt{k}$，可以消除这种放大效应，使不同维度的计算结果保持在合理范围内，从而更稳定地学习注意力权重。

## Multi-head Attention

# References
[^1]: ‘Transformers from scratch | peterbloem.nl’. Accessed: Feb. 01, 2025. [Online]. Available: https://peterbloem.nl/blog/transformers
