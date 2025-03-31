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

## Back Propogation Through Time
在普通的前馈神经网络 (feed-forward nn) 中，我们可以直接应用反向传播来计算梯度并更新权重。但是在 RNN 这种具有时间依赖的网络中，当前的输出不仅依赖于当前输入，还依赖于过去的隐藏状态。这就导致了参数的梯度计算需要沿时间维度进行传播，而 BPTT 正是解决这一问题的方法。
数学推导如下：

**(1) 设定符号**
- 输入序列：$ X = ${$x_1, x_2, ..., x_T$}
- 输出序列：$ Y = ${$y_1, y_2, ..., y_T$}
- 隐藏状态：$ h_t $ 表示时间步 $ t $ 的隐藏状态
- 参数：
  - $ W_{xh} $：输入到隐藏层的权重
  - $ W_{hh} $：隐藏层到隐藏层的权重
  - $ W_{hy} $：隐藏层到输出的权重
- 损失函数：$ L(Y, \hat{Y}) $（如 MSE 或交叉熵）

**(2) RNN 的前向传播**
在 RNN 中，每个时间步的计算如下：
$$
h_t = f(W_{hh} h_{t-1} + W_{xh} x_t)
$$
$$
\hat{y} _ t = g(W_{hy} h_t)
$$
其中：
- $ f $ 是隐藏状态的激活函数（如 tanh 或 ReLU）
- $ g $ 是输出层的激活函数（如 softmax）

总损失为：
$
L = \sum_{t=1}^{T} L_t(y_t, \hat{y}_t)
$

**(3) 反向传播 Through Time**
1. **计算损失对输出的梯度**
   $
   \frac{\partial L}{\partial \hat{y} _ t} = \nabla_{\hat{y}_t} L_t
   $
   计算损失相对于输出的梯度，这部分和普通 BP 计算类似。

2. **计算损失对隐藏状态的梯度**
   $
   \frac{\partial L}{\partial h_t} = \frac{\partial L_t}{\partial h_t} + \frac{\partial L_{t+1}}{\partial h_t} + \frac{\partial L_{t+2}}{\partial h_t} + \dots
   $
   由于隐藏状态 $ h_t $ 会影响后续的所有时间步，因此梯度要沿时间方向**反向累积**。

3. **计算参数的梯度**
   - **对隐藏状态的权重 $ W_{hh} $**：
     $
     \frac{\partial L}{\partial W_{hh}} = \sum_{t=1}^{T} \frac{\partial L}{\partial h_t} \cdot \frac{\partial h_t}{\partial W_{hh}}
     $
   - **对输入权重 $ W_{xh} $**：
     $
     \frac{\partial L}{\partial W_{xh}} = \sum_{t=1}^{T} \frac{\partial L}{\partial h_t} \cdot \frac{\partial h_t}{\partial W_{xh}}
     $
   - **对输出权重 $ W_{hy} $**：
     $
     \frac{\partial L}{\partial W _ {hy}} = \sum _ {t=1}^{T} \frac{\partial L}{\partial \hat{y} _ t} \cdot \frac{\partial \hat{y} _ t}{\partial W_{hy}}
     $

4. **反向传播回传梯度**
   - 计算 **梯度传播路径**，从损失 $ L $ 反向通过时间步 $ T \to 1 $ 逐步更新参数。
   - 使用 **梯度下降（SGD, Adam）** 来更新权重。

显而易见，BPTT存在两个致命问题：
- 第一个是梯度消失或爆炸：当权重矩阵的范数$||W_{hh}||$小于1时，随着时间步的增加，梯度指数级衰减，导致早期时间步的梯度几乎为 0，无法有效训练；
    - 可能的解决方案：使用梯度裁剪（Gradient Clipping） 来限制梯度大小；使用长短时记忆网络（LSTM）或 门控循环单元（GRU） 代替 RNN。
- 第二个问题在于计算开销：BPTT 需要存储所有时间步的隐藏状态，因此当序列很长是，训练效率低。
    - 可能的解决方案：Truncated BPTT，仅在固定窗口大小内进行梯度传播，而不回溯整个序列。
## BPTT Variation
### Truncted BPTT
- **策略**：不展开整个时间序列，而是仅在 **固定窗口大小 $ k $** 内进行反向传播（如$ k=10 $）。
- **优点**：减少计算量，适用于长序列训练。
- **缺点**：可能导致长程依赖信息丢失。
### Online BPTT
- 计算一个时间步的梯度 **立即更新参数**（类似在线学习）。
- 适用于 **流式数据处理（如语音识别）**。

## RNN Variation
LSTM 和 GRU 是RNN的两种主要变体，它们被设计用于解决传统RNN的梯度消失和梯度爆炸问题，从而更好地捕捉长程依赖（long-term dependencies）。LSTM和GRU可以通过门控机制（Gates） 控制信息流动，能够选择性保留或遗忘信息，有效缓解梯度消失问题，从而更好地学习长程依赖。
### LSTM
**(1) 结构**

LSTM 主要引入了三个Gates和一个记忆单元（Cell State, $ C_t $）：
- **遗忘门（Forget Gate）**：决定**丢弃**多少过去的信息。
- **输入门（Input Gate）**：决定**更新**多少新信息到记忆单元。
- **输出门（Output Gate）**：决定**输出**多少隐藏状态。

**完整计算流程**：
$$
f_t = \sigma(W_f [h_{t-1}, x_t] + b_f)
$$
$$
i_t = \sigma(W_i [h_{t-1}, x_t] + b_i)
$$
$$
\tilde{C} _ t = \tanh(W_c [h_{t-1}, x_t] + b_c)
$$
$$
C _ t = f _ t \odot C _ {t-1} + i _ t \odot \tilde{C} _ t
$$
$$
o_t = \sigma(W_o [h_{t-1}, x_t] + b_o)
$$
$$
h_t = o_t \odot \tanh(C_t)
$$

**(2) 详细解释**
- **遗忘门 $ f_t $**：控制是否遗忘过去的记忆。
- **输入门 $ i_t $**：控制是否写入新的信息。
- **候选记忆 $ \tilde{C}_t $**：当前时间步的新信息。
- **记忆单元 $ C_t $**：更新后的长程记忆，**信息可以直接在时间轴上流动，不易丢失**。
- **输出门 $ o_t $**：决定隐藏状态 $ h_t $ 的输出。

> $ C_t $ 直接通过加法连接，使得梯度流动不会因多次乘法而消失。
### GRU
GRU 是 LSTM 的简化版本，去掉了独立的记忆单元 $ C_t $，仅使用隐藏状态 $ h_t $ 作为存储信息的介质。

**(1) 结构**
GRU 主要有**两个门**：
- **重置门（Reset Gate）**：控制如何结合新的输入和过去的隐藏状态。
- **更新门（Update Gate）**：控制多少过去的信息保留，多少新信息加入。

**完整计算流程**：
$$
r_t = \sigma(W_r [h_{t-1}, x_t] + b_r)
$$
$$
z_t = \sigma(W_z [h_{t-1}, x_t] + b_z)
$$
$$
\tilde{h} _ t = \tanh(W_h [r_t \odot h_{t-1}, x_t] + b_h)
$$
$$
h_t = (1 - z_t) \odot h_{t-1} + z_t \odot \tilde{h} _ t
$$

**(2) 详细解释**
- **重置门 $ r_t $**：决定**是否遗忘**过去的隐藏状态。
- **更新门 $ z_t $**：决定**新旧信息的混合比例**，类似 LSTM 的输入门和遗忘门的结合。
- **候选状态 $ \tilde{h}_t $**：新的信息。
- **最终隐藏状态 $ h_t $**：
  - 当 $ z_t $ 逼近 1：当前状态 **接近新信息**。
  - 当 $ z_t $ 逼近 0：当前状态 **保留过去的隐藏状态**。

> GRU 用更新门合并了 LSTM 的输入门和遗忘门，因此计算更简单。

# Transformers
虽然 LSTM 和 GRU 仍然被广泛使用，但现代NLP任务大多采用 **Transformer**（基于自注意力机制），因为：
- Transformer **并行计算** 性能更强，而 LSTM/GRU 依赖序列处理，无法并行。
- Transformer 通过 **Self-Attention** 直接建模长程依赖，而LSTM/GRU仍然有一定信息衰减问题。

但在语音处理、时间序列预测等任务中，LSTM/GRU仍然具有较好的表现。

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
在Self-Attention机制中，每个单词都可以与句子中的其他单词建立关系。然而，在标准的单头注意力（Single-Head Attention）机制下，一个单词只能用一个注意力分布来表示它与所有其他单词的关系。这就导致了一个问题：单个词可能在不同的上下文中承担不同的语义角色，但单头注意力无法区分这些角色。例如Mary gave roses to susan. 在 single-head attention 机制中，所有这些信息都会被简单地加权求和，而不会分别处理不同的语义关系。这意味着，"Mary" 和 "Susan" 都会影响 "gave" 的表示，但它们的影响方式是相同的，无法区分"施事者" 和 "受益者" 的不同语义。

多头注意力如何解决这个问题？
多头注意力（Multi-Head Attention） 通过使用多个独立的注意力头（attention heads） 来提供更高的表达能力。具体来说,每个注意力头（head）具有自己独立的查询、键和值变换矩阵：$W_q^r, W_k^r, W_v^r$，其中$r$表示不同的注意力头。对于输入$x_i$，每个注意力头都会产生一个输出向量$y_i^r$，把这些向量拼接起来，然后通过一个线性变换把维度降到k。

## 什么是Transformer?
> Any architecture designed to process a connected set of units—such as the tokens in a sequence or the pixels in an image—where the only interaction between units is through self-attention.

下面是一个比较基础的架构
![Transoformer](/img/nluplus/transformer_block.png)

## BERT
> BERT - Bidirectional Embedding Representation Transformer

BERT与GPT架构的区别

# Parsing
## Decoding with LLMs
> One naturally wonders if the problem of translation could conceivably be treated as a problem in cryptography. When I look at an article in Russian, I say: ’This is really written in English, but it has been coded in some strange symbols. I will now proceed to decode.

**Terminology: continuation - 补全**

我们如何生成the most probable continuation?
$$
\arg\max_{w_1, ..., w_n} p(w_1, ..., w_n | \text{prefix})
$$
如何求解？
- **遍历所有可能的续写**，计算它们的概率，然后选择概率最大的那个。  但在自然语言处理中，可能的序列数量是**指数级增长**的，所以这种方法通常**不可行**。
- 另一种策略是**逐步生成**，即每一步选择**当前最优**的下一个单词（或 top-k 选项）。这种方法更实用，比如**贪心解码（greedy decoding）** 

### Greedy Decoding
下面是greedy decoding的具体流程
![beam search](/img/nluplus/greedy_decoding.png)
这种解码方式的问题：
- 我们只做一次continuation，一旦发生错误无法回溯更改。
### Beam Searching Decoding
根据greedy decoding的教训，我们选择保留几个continuation，并且每次都扩展most promising的那个。流程如下图所示
![beam search](/img/nluplus/beam_search_smooth.gif)
Beam Searching如何实现？
- 用优先队列这样的数据结构来存储每个continuation
- 这个队列中保持最多k个解析
- 当要解码一个单词时，从队列中pop出一个解析，扩展完再push回队列
- 用priority queue这种结构来做，时间复杂度不会超过O(logk)，甚至能达到O(1)

队列规模k的影响
- 当k=1，就是greedy decoding。所以当k较小时，我们会出现类似的问题
- 当k很大时，一方面是计算昂贵，一方面是有工作表明较大的k往往导致worse translation [^2], [^3]。
- 而且当k很大时，往往生成的contination都很短，因为较短的continuation一般都是优选
- 因此，选择一个合适的k非常重要

解决方法：
### Sampling
与其每次选择概率最大的，不如从输出的概率分布中采样。例如Top-k Sampling和Nucleus sampling

Holtzman et al. 对比了几种方法的效果，如图所示[^4]
![sampling](/img/nluplus/neuclues-sampling.png)

除此以外，还有Locally Typical Sampling
**Top-k Sampling vs. Nucleus Sampling vs. Locally Typical Sampling**
这些都是**文本生成任务**中用于**解码（decoding）**的策略，旨在**平衡多样性和合理性**，特别是在语言模型（如 GPT、LLM）中应用。  
它们的核心目标是**避免单一确定性输出（如 Greedy Search）**，让生成文本更加自然和富有变化。

**核心思想**：
- 选择那些使得**模型生成的下一个单词在上下文中最符合“局部典型性”的单词**。
- 不只是看单词概率本身，而是看它是否在上下文中“听起来”合理（局部熵最小）。

**流程**：
1. 计算所有单词的概率 $ p(w \mid \text{context}) $。
2. 计算这些单词的 **信息熵（self-information）**：
   $$
   s(w) = -\log p(w \mid \text{context})
   $$
3. 选择**信息熵最接近平均信息熵（典型熵）** 的一部分单词。
4. 在这个子集中进行采样。

**优点**：
- 避免了 Top-k 和 Nucleus Sampling 的局限，使得输出更加**符合上下文**，而不是仅仅基于全局概率高低。
- 适用于**对局部语境要求高的任务**，比如对话生成、摘要等。

**缺点**：
- 计算复杂度略高，需要额外计算信息熵。

> 采样的目的就是平衡多样性和合理性。避免语言模型单一确定性的输出。
## Neural Parsing
先复习下Encoder-Decoder架构，如下图所示，Encoder作用是将输入序列（如德语句子）编码成一个固定长度的上下文向量（context vector）。
- 输入词（如 "natürlich", "hat", "john", "spaß"）首先被映射成词向量（$ x_1, x_2, x_3, x_4 $）。
- 这些词向量依次输入到一个RNN 编码器（通常是 LSTM 或 GRU），产生隐藏状态 $ h_1, h_2, h_3, h_4 $。
- 这些隐藏状态捕捉了句子的信息，最终最后一个隐藏状态（这里是 $ h_4 $）被用作整个输入句子的上下文向量（context vector）。

Decoder接收编码器提供的上下文信息，然后逐步生成目标语言的句子（如英文）。
- 编码器最后的隐藏状态被传入解码器的 RNN 作为初始状态。
- 解码器使用 RNN（LSTM/GRU）逐步生成目标语言的单词：
  - 初始状态 $ s_1 $ 依赖于编码器的最后一个隐藏状态。
  - 每个时间步，解码器的 RNN 生成一个新的隐藏状态 $ s_t $。
  - 通过一个 **softmax 层**，$ s_t $ 计算当前时间步的输出词 $ y_t $（如 "of", "course", "john", "has", "fun"）。
  - 解码器的输出作为输入传递到下一步，直到生成完整句子。


![Encoder Decoder](/img/nluplus/encoder_decoder.png)

> Parsing is the task of turning a sequence of words

问题来了，句子解析的输入是一个序列，但输出不是，那我们如何用encoder-decoder模型来进行parsing？

我们可以linearize the syntax tree. 这样我们就有一个序列来表示输出了。例如：I saw a man with a telescope的句子解析就是
`(S (NP (Pro You ) ) (VP (V saw ) (NP (Det a ) (N man ) (PP (P with ) (Det
a ) (N telescope ) ) ) ) )`

此外，还需要一些优化
- 添加EOS。使解码器知道何时停止生成，而不会一直无限制地预测单词。
- 反转输入字符串可以带来小幅的性能提升。
- 加深网络层数。例如对encoder和decoder都使用三层LSTM[^5]。
- 添加注意力机制
- 使用word2vec作为输入（预训练的词嵌入）
- 自动生成训练数据，而不需要人工标注。Vinyals et al. (2015) [^5] 采用 Berkeley Parser（一个已有的句法解析器）来解析大量文本，并用这些解析结果作为训练数据。

可能发生的问题（比例很小，无需担心）：比如我们如何确定opening and closing brackets match? 如何对应输入和输出？如何确保模型输出的是整个序列的最优parsing而不是仅仅预测每个time step上的best symbol?

**Parsing with Transformers**
Kitaev等人用transformers进行parsing。
- Use a transformer to encode the input
- This results in a “context aware summary vector” (embedding) for each input word.
- The embedding encodes word, PoS tag, and position information.
- The embedding layers are combined to obtain **span scores**
- But they also try **factored attention heads**, which separate position and content information.

什么是span scores? 在NLP中，尤其是命名实体识别（NER）、句法解析（Parsing） 和 信息抽取（IE） 等任务中用于评估模型性能的指标。
Span scores 通常基于 Precision（精确率）、Recall（召回率） 和 F1-score（F1 分数） 来评估模型在 span-level 的表现：

Span scores在Transformer解析任务中的计算方法：计算span相关的向量差值，经过线性变换、归一化（LayerNorm）、非线性激活（ReLU）和再次变换，得到 span scores。最终，这些得分用于 解码（decode the output），即找到最优的解析树。

## Unsupervised Parsing
Unsupervised Parsing[^7]主要是从未标记的文本中归纳出语法结构。

Unsupervised Parsing的SOTA的F-score只有60，对比supervised parsing来说很难，supervised parsing的F-score可以达到95以上


# LLM前沿研究
## Scaling Law
例如，给定固定的计算资源预算，训练Transformer语言模型的最佳模型大小和训练数据集大小是多少？

这类问题就可以用Scaling laws来回答。它提供了Compute Budget(C)，size of model(N)以及number of training tokens(D)之间的一种关系。有一个粗略的共识是，它们可以使用power laws或类似的定律（以及它们的组合）来建模。

### Power Law
Power Law描述的是变量x和它某些行为之间的关系，公式表示为$f(x) = \alpha x^\mathcal{-K}$

它有一个重要的性质：当$x$乘以一个因子时，$f(x)$也同样作为$κ$的函数乘以一个因子。即：
$$
f(cx) = \alpha (cx)^\mathcal{-K} = c^\mathcal{-K}f(x)
$$

### Kaplan et al. (2020)
Kaplan等人的研究[^8]总结
- 模型的性能很大程度上取决于scale（模型参数的数量，数据集大小，计算预算）而不是网络拓扑（层数，每层的宽度）。
- 数据集大小和模型大小的一起增长是很重要的。
- 性能可以用power law来建模。

还有一点，我们使用floating-point operations per second (FLOPs) 来衡量compute，而不是单纯依赖时间来衡量。

对他们的工作（Kaplan Scaling Law）的主要批判点在于，他们在训练中使用了相同的学习率。

### Hoffmann et al. (2020)
以前的scaling law认为，增加计算预算（compute budget）时，优先增加模型大小（model size），其次才增加数据量（data size）。
比如计算预算 ×100 时，模型参数量 ×25，数据量 ×4。因为直觉上更大的模型能更有效地利用数据，所以数据规模的增长不需要太快。
但是，过度增加模型大小可能会导致过拟合或计算资源浪费。

Hoffmann等人2022年的研究提出了一个新的结论[^9]：

计算预算增加时，模型大小和数据量应该同比例增长。
比如计算预算 ×100 时，模型大小 ×10，数据量 ×10。因为之前的策略导致很多大模型在小数据上训练，出现欠训练（undertrained models） 的问题。
Hoffmann 发现，更大模型并不一定能完全利用固定数量的数据，合理增加数据量可以显著提升模型性能。这个发现也影响了后续 LLM 训练策略，例如 GPT-4、Gemini、PaLM 等更关注 数据与模型规模的均衡增长。

如何推导Hoffmann等人提出的Chinchilla's Scaling Law?
- L: LM average test loss (entropy loss)
- D: dataset size, number of tokens
- N: number of parameters
- C: compute budget, C=C(N, D)
已知一个固定的compute budget C*, 找到
$$
\arg\min_{N, D}L(N, D) = C^*
$$
用power law来建模：
$$
L(N, D) = \frac{a}{N^\alpha} + \frac{b}{D^\beta} + c
$$
$c$是理想的test loss

因此，Hoffmann等人训练出来了两个模型：
- Gopher: 280 billion parameters, 300 billion tokens, L(N, D) = 1.993
- Chinchilla: 70 billion parameters, 1.4 trillion tokens, L(N, D) = 1.936

Chinchilla 遵循这一策略（较小模型 + 更多数据），结果证明它在test loss和下游任务上表现更优。

### 增加效率
语言模型的训练中，主要做的计算就是矩阵乘法和求和归一化（Softmax）。用一个公式概括Transformer做的事情就是$\text{Softmax}(QK^T)V$

GPU是并行处理器，结构如下图所示
![gpu hardware](/img/nluplus/gpu_hardware.png)
- 基础的计算单元是线程
- 线程被分组成Block，Block中的所有线程都有唯一的ID，但运行相同的代码
- 这使得GPU可以做极端的并行化
- Grid是一组Block。每个Block可以运行不同的代码段
- GPU有很大的global memory，很大，但是很慢
- 每个Block都有一个共享内存-Block中的所有线程之间共享这个内存，它效率很高，但很小
- 写代码时要尽可能少用全局内存，多用Block的共享内存

### Double Descent
Double Descent是深度学习训练过程中出现的一种现象，它描述了模型的test error随着model size或dataset size变化的非单调趋势。
Double Descent 现象表明，在“过拟合区域”之后，继续增加模型规模，test error反而会再次下降，进入“第二次泛化区域”。

因此，传统统计学习理论预测过拟合会导致泛化能力下降，但Double Descent说明在超大模型下，泛化能力反而会回升。

## LLMs as Formal Machines

# LLM微调

# Evaluating Generation and Machine Translation

# Ethics

# Evaluation of LLM and Alignment

# QA and RAG


# References
[^1]: 'Transformers from scratch' Available: https://peterbloem.nl/blog/transformers
[^2]: Tu, Zhaopeng, et al. "Neural machine translation with reconstruction." Proceedings of the AAAI Conference on Artificial Intelligence. Vol. 31. No. 1. 2017.
[^3]: Koehn, Philipp, and Rebecca Knowles. "Six challenges for neural machine translation." arXiv preprint arXiv:1706.03872 (2017).
[^4]: Holtzman, Ari, et al. "The curious case of neural text degeneration." arXiv preprint arXiv:1904.09751 (2019).
[^5]: Vinyals, Oriol, et al. "Grammar as a foreign language." Advances in neural information processing systems 28 (2015).
[^6]: Kitaev, Nikita, and Dan Klein. "Constituency parsing with a self-attentive encoder." arXiv preprint arXiv:1805.01052 (2018).
[^7]: Cao, Steven, Nikita Kitaev, and Dan Klein. "Unsupervised parsing via constituency tests." arXiv preprint arXiv:2010.03146 (2020).
[^8]: Kaplan, Jared, et al. "Scaling laws for neural language models." arXiv preprint arXiv:2001.08361 (2020).
[^9]: Hoffmann, Jordan, et al. "Training compute-optimal large language models." arXiv preprint arXiv:2203.15556 (2022).
