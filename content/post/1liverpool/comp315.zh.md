---
title: "云计算(COMP315)"
date: 2024-02-03T17:39:49Z
draft: false
author: "Martin"
tags: 
- 云计算
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
# 虚拟机
> 借助虚拟化技术，用户能以单个物理硬件系统为基础创建多个模拟环境或专用资源。"Hypervisor" （虚拟机监控程序）的软件可直接连接到硬件，从而将一个系统划分为不同的、单独安全环境，即虚拟机（VM）。

## Type 1 & Type 2 虚拟机监控程序
第 1 类虚拟机监控器位于裸机服务器之上，可以直接访问硬件资源。因此，第 1 类虚拟机监控器也称为裸机虚拟机监控器。

相比之下，第 2 类虚拟机监控器是安装在主机操作系统上的应用程序。此类虚拟机监控器也称为托管或嵌入式虚拟机监控器。

- 第 1 类虚拟机监控器或裸机虚拟机监控器直接与底层计算机硬件交互。裸机虚拟机监控器直接安装在主机的物理硬件上，而不是通过操作系统安装。
- 第 2 类虚拟机监控器或托管虚拟机管理监控器通过主机的操作系统与底层主机硬件进行交互。第 2 类虚拟机监控器安装在计算机上，在其中作为应用程序运行。
- Hypervisor属于协调层，同时运行多个虚拟机
![Hypervisor](/img/comp315/hypervisor.png)

# JavaScript
> 比较熟悉了，简单写几个Tutorial的点
- 原型链(Prototype Chain)：这个机制的关键点在于当你尝试访问一个对象的属性或方法时，如果该对象本身没有这个属性或方法，JavaScript引擎就会沿着这个对象的原型链向上查找，直到找到这个属性或方法或到达原型链的末端。```object.getPrototypeOf()```
- function也是一个object
- undefine和null是不同的类型，
    - ```undefine === null``` False
    - ```undefine == null``` True
- 声明变量别用var。变的就用let，不变就用const，舍弃var
- ```html
    <button class="
      bg-green-500 
      hover:bg-green-700 
      text-white 
      font-bold 
      py-2 px-4 
      rounded 
      transform transition 
      hover:scale-110">
        Shop Online
    </button>
    <!-- py-2 px-4 指x和y方向上的padding大小-->
- map方法
    ```js
    const numbers = [1, 2, 3, 4];
    const doubled = numbers.map(num => num * 2);
    console.log(doubled);
    ```
- reduce方法
    ```js
    const numbers = [1, 2, 3, 4];
    const sum = numbers.reduce((acc, curr) => acc + curr, 0);
    console.log(sum);
    ```
# 容器
## Docker

## Docker和Kernel

# 安全
## Vulnerabilities and the Zero-Day Problem
## 主机防御
## SELinux

