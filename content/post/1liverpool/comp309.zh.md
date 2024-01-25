---
title: "算法进阶(COMP309)"
date: 2023-03-17T17:39:49Z
draft: false
author: "Martin"
tags: 
- Algorithm
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
# 1. Algorithmic Paradigms
# Greedy Algorithm
## Greedy Choice Property
通过每一步的局部最优选择，可以最终构建出全局最优解。

在活动选择问题中的表现：S是一组活动，A是S中具有最早结束时间的活动，对于最优解$ X \subseteq S $, A被包含在X中。

**证明方法：**
假设$X \subseteq S$是一个最优解，A'是X中具有最早结束时间的活动，如果A=A'，证明结束。否则，用A替换A'获得另一个方案X'(X'中的活动数量和X的活动数量相同)，那么X'就是一个包含活动A的最优解。
## Recursive Property
递归性质指的是一种方法或函数能够通过调用自身来解决问题。递归方法将问题分解为更小的子问题，这些子问题结构上与原问题相似但规模更小。

在活动选择问题中的表现：可以从较小问题实例 S' 的任何最优解中找到S中包含 A 的最优解： 特别是，如果 X ′′ 是 S′ 的最优解，那么 X ′′ ∪ {A} 是 S 的可行解中包含 A 的最佳解。

**证明方法：**
假设X''是S'中的最优解，要证明X'' U {A}有尽可能多的活动，在S的可行方案（包含A）中。通过反证法，假设这里对于S有更好的解Y' U {A}，那么在S‘中，Y'应该比现存的最优解X''更优。这互相矛盾，因此反证成立。

## 一些贪心算法不适用的问题
1. Weighted Activities Selection Problem
2. Fractional Knapsack Problem

## Divide and Conquer
可以用分治解决Weighted Activities Selection Problem

## Dynamic Programming
添加了备忘录的分治就是动态规划，即不用重复计算子问题的解。
# 2. Pattern Matching
