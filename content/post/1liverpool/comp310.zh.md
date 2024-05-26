---
title: "多智能体系统(COMP310)"
date: 2024-05-05T17:39:49Z
draft: false
author: "Martin"
tags: 
- 多智能体系统
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
# Intelligent Agents
> Agents are objects with attitude, which are autonomous, smart, active.
## Properties in Environment (环境性质描述)
- Fully observable / partially observable
- Deterministic / non-deterministic
- Static / dynamic
- Discrete / continuous
- Episodic / non-episodic
- Real time

## Social ability (社交能力)
The ability to interact with other agents via cooperation, coordination, and negotiation.

## Abstract Architectures for Agents (代理的抽象架构)
表示方法
- State: $E = \\{e, e', ... \\}$
- Actions: $Ac = \\{\alpha, \alpha', ... \\}$
- Run: $r: e_0\xrightarrow{\alpha_0}e_1\xrightarrow{\alpha_1}...\xrightarrow{\alpha_{u-1}}e_u$
    - $R$: 有限序列，包含所有可能的情况
    - $R^{Ac}$: 所有以action结尾的序列
    - $R^E$: 所有以environment结尾的序列

**State transformer function**
$$\tau: R^{Ac} \rightarrow 2^E$$
表示一个代理动作在环境上的作用。

如果$\tau(r) = \emptyset$，那么对于$r$来说没有后继状态，可以定义为运行结束

Environment definition: $Env = <E, e_0, \tau>$
- $E$是一组状态
- $e_0$是初始状态
- $\tau$是状态转移函数

**Agent**
把一个agent建模成一个函数，这个函数映射runs(假设这些序列以state结尾)到actions。
$$Ag: R^E \rightarrow Ac$$

**Systems**
系统可以表示为$R(Ag, Env)$，意思是代理$Ag$在环境$Env$上的一些列runs

**Purely Reactive Agents**
- 他们不根据历史做决定
- 他们做的决定全部根据现在的情况

$$Ag: E\rightarrow Ac$$
温控器就是一个purely reactive agents，当e = temperature, $Ag(e) =$ off，否则，$Ag(e) =$ on。
## Utilities
> Agent在特定环境中目标满意度的量化衡量指标。

期望效用（Expected Utility）的计算在多智能体系统（MAS）和决策理论中有多种用途，主要用于评估、优化和指导智能体的行为与策略。
Expected Utility计算公式如下
$$EU(Ag, Env) = \sum_{r\in R(Ag, Env)} u(r)P(r|Ag, Env)$$

# Deductive Reasoning Agents
## Symbolic Reasoning Agents
建立agents的一个经典方法就是把他们看作一种特别的知识库系统，并且把相关的方法应用到这个系统中。这类范畴就属于symbolic AI。
## Agents as Theorem Provers

## Agent-Oriented Programming
