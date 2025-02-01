---
title: "Llama3.cpp本地CPU部署大模型"
date: 2024-06-15T17:39:49Z
draft: false
author: "Martin"
tags: 
- 大模型
description: ""
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示当前路径
cover:
    image: ""
    caption: ""
    alt: ""
    relative: false
mermaid: true
---
> 机器是2021年M1的MacBook Pro，斗胆一试。

# 引言
master分支的commit记录：```2075a66```

分别部署了```Llama-2-7B-Chat```，```Llama-2-13B-Chat```, ```Llama-2-70B-Chat```三款大模型，部署方式完全一致，我拿7B模型举例。

PS: 2-bit量化的7B和13B模型本地完全可以带动，毫无压力。但也是因为损失了太多精度导致结果一般。
由于内存限制，我在本地无法下载70B的模型，索性转战到学校的超级计算中心😎。

# 前置知识
为降低本文阅读门槛，我将先介绍一些必备的基础知识。
1. 量化
    - 众所周知，大模型本质就是一个巨大的矩阵计算，计算的每一个数字都是浮点数。大公司的LLM推理计算一般都是使用float32，确保精度足够高，从而保证准确率。但是float32的唯一缺点就是需要大量的计算资源（约等于马内），
    普通人需要的大模型根本也不需要这么高的精度/准确率。所以量化技术应运而生，最常见的两种，即```float32 -> float16```和```float32 -> int8```。
    - 我们可以从huggingface直接下载量化好的大模型（各种量化位数都有），所以这里不过多叙述量化的原理，想要具体了解可以阅读[文档](https://huggingface.co/docs/optimum/en/concept_guides/quantization)。

# 部署步骤
首先克隆仓库，进入目录并编译项目
```shelll
$ git clone https://github.com/ggerganov/llama.cpp
$ cd llama.cpp
$ make
```
编译好项目之后，我们主要关注./examples和./models这两个目录
- ./examples: 这个目录下主要是一些使用示例。
- ./models: 这个目录下主要用来存放我们下载的模型