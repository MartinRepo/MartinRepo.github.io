---
title: "云计算(COMP315)"
date: 2024-02-26T17:39:49Z
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
> 容器是虚拟机的轻量级替代产品，易于创建和管理。
容器封装了应用程序及其所需要的依赖，位于操作系统的内核之上。从应用程序的角度来看，容器就是虚拟机，只不过”更轻“。
## Docker
> Docker是最令人熟知的容器化技术。

在Docker中，容器由镜像创建，镜像由Dockerfile编写
```shell
# 拉一个Ubentu镜像
$ docker pull ubuntu:latest

# 运行这个容器
$ docker run -it ubuntu:latest /bin/bash
```
写一个Dockerfile，创建docker镜像👇
```dockerfile
# 指定基础镜像
FROM ubuntu : latest
# 创建shell脚本并将“echo Hello, World!”写入脚本
RUN echo " echo Hello , World ! " > / hello - world . sh
# 为文件添加执行权限
RUN chmod + x / hello - world . sh
# 指定容器启动时默认执行的命令。
CMD [ " / hello - world . sh " ]
```
运行Docker👇
```shell
# Build the Docker image
$ docker build -t hello - world .

# Run the Docker container
$ docker run hello - world
```
常用Docker命令
```shell
# 查看镜像
docker images
# 查看正在运行的容器
docker ps
# 查看所有容器，包括停止的
docker ps -a
# 停止容器
docker stop [containerId]
# 删除容器
docker rm [containerId]
# 删除镜像
docker rmi ubentu:lastest
```
image（镜像）和container（容器）的区别
最根本的区别在于它们操作的对象不同。```docker images```关注的是Docker镜像，这些镜像是容器的模板。而```docker ps```关注的是由这些镜像创建的容器的实际运行状态
## Docker和Linux Kernel
- 共享Kernel：容器利用宿主机的内核，它们不需要像虚拟机那样虚拟化硬件和运行自己的操作系统内核。这意味着容器非常轻量，启动快，占用资源少。
- 隔离性：尽管容器共享宿主机的内核，但它们通过Linux内核的特性如cgroups（控制组）和namespaces（命名空间）来实现进程、网络、文件系统等方面的隔离。这使得容器表现得就像是在独立的环境中运行一样。
- Kernel依赖性：因为容器直接依赖于宿主机的Linux内核，所以容器化的应用必须与宿主机的内核兼容。这也意味着，如果你需要特定的内核特性或版本，你的宿主机必须运行支持这些特性的Linux内核。

### Namespaces

### Control groups

### Union File Systems
# 安全
## Vulnerabilities

## The Zero-Day Problem

## 主机防御
## SELinux

