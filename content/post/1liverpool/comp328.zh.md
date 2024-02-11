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
## Plot programs performance
用"Arithmetic Intensity"，记作$I(n)$，来评估性能。$I(n) = {W(n)\over Q(n)}$。

$W(n)$:程序执行flops的次数，$Q(n)$:从内存传输到缓存的字节数。

- 低Arithmetic Intensity的程序叫内存受限程序
- 高Arithmetic Intensity的程序叫计算受限程序
- 对于内存受限的程序，处理器需要花费更多时间等待操作数从内存中传送出来（更多的时间花费在访问内存上，而不是运算上）。

### Roofline Model
Roofline Model是帮助了解软件性能的可视化工具。

Roofline Model两个组成部分
- **峰值性能（$R_{peak} / R_{max}$）**：这是计算系统在理想情况下可以达到的最高计算性能，通常以每秒浮点运算次数（FLOPS）来衡量。峰值性能由处理器的硬件特性决定，比如核心数量、时钟频率和向量化能力。图中水平线就是峰值性能
- **内存带宽（Memory Bandwidth）**：这是计算系统在单位时间内能从内存中读写数据的最大速率，通常以每秒传输的字节数（Bytes/s）来衡量。内存带宽是由系统的内存架构和内存类型决定的。带角度的斜线表示内存带宽
## 冯·诺伊曼
Von Neumann架构图如下所示👇
![The von Neumann Architecture](/img/comp328/vonNeumann.png)
- 由控制单元和算术/逻辑单元组成的CPU。
- 独立的存储区，可存储指令和数据。
- 指令由CPU执行，因此必须将指令从存储器带入CPU。
- 数据也必须从存储器进入CPU才能执行。
- CPU包含寄存器，作为临时存储的刮板。
- **The von Neumann bottleneck:** 数据和指令共用一条总线，因此指令获取和数据操作不能同时进行。

### 冯·诺伊曼瓶颈
1. 从内存中抓取对应程序计数器的指令
2. 解码指令
3. 从内存中抓取数据
4. 执行指令
5. 写回结果

### Pipelining
**Pipelining的基本概念**

流水线技术通过将指令的执行过程分解为多个阶段，并让不同的指令在不同的时间并行处理这些阶段来提高处理速度。这就好比是在组装线上，每个工人负责组装线上的一个特定任务，产品可以更快地完成，因为多个任务是在同时进行，而不是一个接一个地完成。

**在von neumann cycle中的应用**

在应用流水线技术后，处理器可以在完成当前指令的某个阶段的同时，开始执行下一条指令的前一个阶段。例如，当第一条指令在执行阶段时，第二条指令可以同时进行译码，第三条指令可以进行取指。这样，虽然每条指令的执行仍然需要串行经过所有阶段，但处理器可以在同一时刻处理多条指令的不同阶段，从而大大提高了指令的吞吐率。

### 对抗冯·诺伊曼瓶颈的方法
> 在芯片上添加高速缓存（cache)，但高速缓存也存在问题

例如，高速缓存越大，数据访问速度越慢。可以采用多级缓存架构，通过在处理器和主内存之间引入多个层级的缓存，旨在平衡缓存大小、访问速度和命中率之间的关系。

高速缓存利用了程序的空间局部性（spatial locality）和时间局部性（temporal locality）。

- 时间局部性指的是在较短的时间内，被访问过一次的数据项很可能在不久的将来再次被访问的特性。这种访问模式意味着一旦数据被加载到缓存中，它很可能很快再次被需要，因此保留这些数据项在缓存中可以减少对较慢主存的访问次数。
- 空间局部性是指如果一个数据项被访问，那么其附近的数据项很快也可能被访问的特性。这种模式基于数据存储的物理结构，相邻的数据项通常也在内存中相邻存储。高速缓存系统利用这一特性通过预取附近的数据项到缓存中，即使这些数据项还没有被显式请求。

AMD Bulldozer 服务器插槽的内存层次结构
![AMD Bulldozer 服务器插槽](/img/comp328/Hwloc.png)


## How to gain performance form a single core/socket
```java
for (int i = 0; i<1000; i++) {
    b[i] = a[i]*a[i];
}
```
对于上面这个程序，应该运行1000个clock cycles。假如一个clock cycle不是做一次迭代，而是做4次迭代，那么总共需要250个clock cycles。这就叫vector processing，也叫Single Instruction, Multiple Data（SIMD）。

在单核上压榨更多性能：SMT(simultaneous multithreading)，通过在单个物理CPU核心上同时执行多个线程来提高处理器的效率和性能。SMT允许单个核心像操作系统和应用程序呈现出多个逻辑核心或线程，使得处理器可以更有效地利用其资源，特别是在一个线程等待数据访问或执行长时间操作时，处理器可以转而执行另一个线程的任务。


