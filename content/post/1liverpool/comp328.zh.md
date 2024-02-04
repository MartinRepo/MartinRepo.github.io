---
title: "高性能计算(COMP328)"
date: 2024-02-03T17:39:49Z
draft: false
author: "Martin"
tags: 
- 高性能计算
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
# Week1
高性能计算的目标
- 对于有限的数据集，最小化解决时间
- 对于无限的数据集，最大化吞吐量(throughput)
- 有能力解决一些对于可用的内存来说太大的问题
- 最大化资源利用 - CPU/内存/网络/加速器(GPU)/电力

## 一些常用术语
1. Parallelism vs Concurrency
    - Parallelism: 多个进程同时且独立执行
    - Concurrency: 多个进程同时执行且共享至少一种资源
2. Processor, Die & Socket
    - Processor: 执行程序指令的电路。计算机系统中可能有许多处理器，例如图形处理器、视频处理器。在没有限定的情况下，通常指中央处理器
    - CPU: 计算机系统中主要的通用处理器（之一），而非特定用途（如视频解压缩）。
    - Die: 指硅晶片，包含处理器（通常是中央处理器）以及接口所需的其他组件（如内存控制器）。
    - Socket: 处理器和计算机主板之间的物理接口，它定义了处理器与主板连接的方式。不同的处理器和主板可能需要不同类型的Socket。
3. Core & Thread
    - Core: 核心是CPU内部的一个物理处理单元，能够独立执行计算任务。每个核心可以独立处理指令和执行计算操作。
    - Thread: 线程是操作系统能够进行计算调度的最小单位。它是程序执行流的一个单一顺序，可以被操作系统调度（启动、停止、挂起等）。
    - 核心和线程共同定义了处理器的处理能力
4. Node
    - 指一个服务器节点（一台计算机）
5. Cluster/Supercomputer
    - 成百上千个节点组成集群
6. Single precision floating-point
    - 通常占用32位（4字节）的存储空间，C语言中的float32类型，1符号位，8指数位，23有效数字位
7. Double precision floating-point
    - 通常占用64位（8字节）的存储空间，C语言中的float64类型，1符号位，11指数位，52有效数字位
8. Flop
    - Floating-point operations per second

## 如何量化/评估性能
公式如下👇

$R_{peak} = 2\times w_{vec} \times r_{clock} \times n_{core} \times n_{socket}$

变量解释：
1. $R_{peak} - $峰值理论性能
2. $w_{vec} - $向量宽度，表示每个处理器核心每个时钟周期内可以执行的浮点运算数。
3. $r_{clock} - $表示处理器核心每秒钟可以执行的时钟周期数。
4. $n_{core} - $表示单个处理器（CPU）或计算节点中的核心数量。
5. $n_{socket} - $表示系统中处理器（CPU）的数量。

**Linpack性能测试**：Linpack性能测试是一种衡量计算系统解决高密度线性代数问题能力的测试。这个测试通过测量系统在执行大规模双精度（64位）浮点算术矩阵运算时的性能来评估计算机的速度和效率。Linpack测试的一个典型应用是计算给定大小的矩阵$A$和向量$b$，求解线性方程组$Ax = b$
# Week2
## Plot our programs performance

## The von Neumann bottleneck

## How to gain performance form a single core/socket
