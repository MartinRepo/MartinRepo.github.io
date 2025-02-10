---
title: "数据分析"
date: 2025-02-01T10:39:49Z
draft: false
author: ["Martin"]
tags:
- 数据分析
description: ""
weight: 
slug: "data-analysis"
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
    relative: true
mermaid: true
---
> 记录一些基础方法

# 时间序列
## 可视化三种时间序列
- Brownian motion (stochastic process)
- Lorenz attractor (chaotic system)
- Lotka-Volterra system (deterministic system of differential equations)
![three types of time series](/img/ddbi/3types_time_series.png)
- 随机过程是一种非确定性的过程，即其未来状态依赖于概率分布，无法通过确定的数学公式精确预测。布朗运动（Brownian Motion）是一种典型的随机过程，常见于金融市场、分子扩散等领域。
- 混沌系统是一类确定性但不可预测的系统，即尽管系统是由确定性的方程描述的，但对初始条件极其敏感，导致长期行为难以预测。洛伦兹系统（Lorenz System）是一个经典的混沌系统，最早由 Edward Lorenz 研究气象现象时提出。其特征包括：
    - 由确定性方程控制（如微分方程）。
    - 对初始条件极其敏感（“蝴蝶效应”）。
    - 具有某种结构（如吸引子），但长期行为是不可预测的。
- 确定性系统是完全由数学规则或方程所控制的系统，即 如果初始条件相同，每次运行都会得到相同的结果。洛特卡-沃尔泰拉（Lotka-Volterra）方程描述了捕食者（如狼）和被捕食者（如兔子）种群的动态变化。这类系统的特征包括：
    - 未来状态由当前状态唯一决定。
    - 没有随机性，系统演化完全可预测。
    - 在生物学、物理学、化学等领域有广泛应用。

## 数据填充技术
这主要用于处理时间序列上的缺失值，主要有三种方法
- Forward Fill: 用前一个已知的值填充当前缺失的值。
- Average of Neighbors: 用缺失值相邻的两个已知值的平均值进行填充，常用于平稳数据。
- Linear Interpolation: 假设缺失值可以沿着相邻观测点的直线关系进行插值。适用于趋势平稳或线性增长/下降的数据。
## Preprocessing, scaling & reversing transformations
下面是几种常见的transformation：
- **Differencing:** Remove trends by looking at changes rather than absolute values.
- **Log-Differencing:** Stabilize variance and remove trends, often used when dealing with multiplicative trends.
- **Normalization (Min-Max):** Scale data to [0,1].
- **Standardization (Z-score):** Transform data to have mean 0 and std 1.

