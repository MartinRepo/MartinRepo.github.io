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

Linux内核支持容器化依赖以下三个功能
- Namespaces
命名空间能够隔离和分割系统资源，使得进程只能看到自己命名空间内的资源。Linux提供了多种类型的命名空间，每种类型都隔离一组特定的系统资源。例如，PID（进程ID）命名空间隔离进程ID，使得不同命名空间中的进程可以有相同的PID，而网络命名空间隔离网络接口和路由表，等等。这意味着在一个容器中运行的进程可以有自己的网络接口和IP地址，就好像它在自己的虚拟网络中一样。
- Control groups
控制组允许对进程群组进行资源控制、限制、分配和监视。通过cgroups，系统管理员可以指定CPU、内存、磁盘I/O等资源的限制，确保某个容器不会使用过多的资源，从而影响到系统上其他的进程或容器。
- Union File Systems
联合文件系统是一种特殊的文件系统，它可以将多个不同的文件系统“叠加”在一起，形成一个统一的文件系统视图。这意味着可以将不同的目录层次结构合并成一个目录视图，对于容器而言，这使得可以通过叠加不同的层（每层包含文件的变化）来构建容器的文件系统。Docker中的镜像和容器层就是使用联合文件系统技术实现的。这种方式使得容器镜像变得更加高效，因为多个容器可以共享同一个基础镜像的只读层，而每个容器的变化都存储在自己的可写层中。
# 安全
## Vulnerabilities and the Zero-Day Problem
零日问题（Zero-day vulnerability）指的是一种安全漏洞，这种漏洞在被发现后还没有得到修复，攻击者可以利用这个漏洞来进行攻击，而且被攻击的目标并没有提前意识到这个漏洞的存在。因此，攻击者可以在漏洞被公开之前利用它，这就是为什么称之为“零日”，因为漏洞被发现的那一天就被利用了。

## 主机防御
> 通常来讲，我们无法阻止零日攻击，但是可以降低零日攻击带来的损害，这就涉及到主机防御(host defences)。
### 最小权限原则
The principle of least privilege (PoLP)，其核心思想是任何用户、程序或系统进程在执行某项操作时，应该仅具有完成该操作所需的最低权限或最小权限集。最小权限原则的目的是减少安全风险并提高系统的安全性。它有助于防止恶意软件扩散，减少系统漏洞的攻击面，并限制对敏感数据的访问，从而降低数据泄露或损坏的风险。

### 权限提升
权限提升（Privilege Escalation）是指在计算机安全领域中，通过利用系统中的漏洞、配置错误或设计缺陷，未授权的用户、程序或进程获得了比原本分配给它们的更高的访问权限或功能的过程。
- 垂直权限提升
垂直权限提升，也称为提权，发生在当低权限用户或程序获取高权限级别（如管理员或root权限）的能力时。这种类型的权限提升允许攻击者执行通常只有系统管理员才能执行的操作，如访问敏感文件、修改系统设置或安装恶意软件。
- 水瓶权限提升
水平权限提升发生在当用户或程序从一个普通账户跳转到另一个具有相同权限级别的账户时。虽然攻击者没有获取更高的权限级别，但他们可能能够访问特定的、原本不应该访问的数据或功能。例如，一个普通用户通过水平权限提升成为另一个普通用户，可能访问到第二个用户的私人文件或数据。

### Sandboxes
沙盒（Sandbox）是一种安全技术，旨在在一个隔离的环境中运行程序或代码，以防止该程序或代码对宿主系统造成潜在的损害或不安全的影响。沙盒提供了一个严格控制的执行环境，其中的应用程序可以执行，但只能访问明确允许的资源和操作。

沙盒的特点
- 隔离性：沙盒环境与宿主系统及其他沙盒环境隔离，以防止程序间的相互影响和对宿主系统的潜在破坏。
- 资源控制：沙盒限制了运行在其中的程序所能访问的系统资源（如文件系统、网络、系统配置等），并可以监控和控制对这些资源的访问。
- 操作限制：沙盒通常限制程序执行的操作类型，比如禁止修改系统设置或访问敏感数据。
- 监控与审计：沙盒环境可以监控程序的行为，记录程序执行过程中的所有操作，为安全分析和审计提供依据。

### 自主访问控制
Discretionary Access Control 是一种访问控制模型，它允许资源的拥有者或拥有者授权的用户控制对资源的访问。在DAC模型中，用户可以根据自己的判断（即“自主”地）决定谁可以访问特定的资源。

特点
- 所有权和权限：在DAC系统中，资源的访问权限通常基于资源的所有权。所有者有权设置或更改对其资源的访问权限。
- 灵活性：DAC提供了较高的灵活性，允许用户根据需要分享和限制资源的访问。这种模型适合需要频繁更改权限设置的环境。
- 用户控制：用户（或用户组）可以直接控制对其拥有的资源的访问，而不是由中央政策或管理员决定。

一些指令👇
```shell
# 查看文件权限
$ ls -l filename

# 更改文件权限，把文件改成可读可写可执行
$ chmod 755 filename

# root user执行，给管理员权限
$ sudo chmod 755 filename
```
在Linux中，文件权限由三位数组成，可读是4，可写是2，可执行是1。所以755意思
- 7: (r+w+x) 有可读可写可执行的权限。
- 5: (r+x) 当前分组有可读+可执行的权限，但没有可写的权限。
- 5: (r+x) 其他人也有可读+可执行的权限，但没有可写的权限。

### 强访问控制
Mandatory Access Control (MAC)。它根据中央权威的决策而不是用户的自主决定来限制对系统资源的访问。与自主访问控制（Discretionary Access Control，DAC）相比，MAC提供了更为严格和详细的控制，确保只有授权用户才能访问特定的信息或资源。

特点
- 中央管理：在MAC策略中，访问控制策略由系统管理，而不是由资源的所有者或用户设置。这意味着用户不能更改他们不拥有的文件或资源的访问控制设置。
- 安全标签：MAC系统使用安全标签来标识主体（如用户或进程）和客体（如文件、目录或设备）。这些标签包含了安全级别或分类信息，如机密、秘密等。
- 访问决策：访问控制决策基于比较主体的安全级别与客体的安全级别。只有当主体的安全级别足以访问客体时，访问才会被允许。
- 策略强制执行：MAC强制执行预定义的安全策略，这些策略旨在保护信息不被未授权的访问、泄露或篡改。

在Red Hat Enterprise Linux, SELinux作为解决方案实现MAC。
## SELinux
> 这是一个在Linux内核中实现的安全框架

SELinux有三种执行模式
1. enforcing
    - 这是SELinux的默认模式。在这种模式下，SELinux会强制执行其安全策略，限制进程访问资源。如果一个进程尝试执行一个不被允许的操作，SELinux会阻止这个操作并记录相应的违规信息。这意味着，如果SELinux的策略不允许某个操作，那么这个操作就不能被执行，无论执行它的用户或进程有多高的权限。
2. permissive
    - 在这种模式下，SELinux同样会检查所有的操作，判断它们是否违反了安全策略，但它不会实际阻止任何操作。相反，它会记录所有被检测到的违规行为，就像在强制模式下那样。这个模式对于调试SELinux策略非常有用，因为它允许管理员看到哪些操作会被阻止，而不需要实际上阻止这些操作，这样可以在不影响系统正常运行的情况下测试和修改策略。
3. disabled
    - 在这种模式下，SELinux被完全禁用，不会检查或限制任何操作。这意味着系统的安全将完全依赖于传统的离散访问控制（DAC）机制，如文件权限等。禁用SELinux模式通常不推荐，因为这会降低系统的安全性。然而，在某些特定场合，比如兼容性测试或者在系统上运行不兼容SELinux的软件时，可能需要暂时禁用SELinux。

**Labelling in SELinux**

在SELinux（Security-Enhanced Linux）中，标签（Labeling）是实现其安全策略的核心机制之一。SELinux通过为系统中的每个对象（如文件、进程、用户等）和主体（通常指用户或进程）分配安全标签来工作。这些标签用于定义和控制主体对对象的访问权限，依据的是定义在SELinux策略中的规则。

作用：SELinux通过匹配主体的标签与对象的标签来决定是否允许某个访问请求。这种基于标签的访问控制机制允许SELinux实现强制访问控制（MAC），即使在主体拥有足够Linux传统权限（如root）的情况下，也能限制对敏感资源的访问。

SELinux的安全标签通常由以下几部分组成：
- 用户（User）：不同于Linux系统的用户，SELinux用户代表的是安全策略中的一个元素，用于控制访问权限。它定义了一组可以执行的角色。
- 角色（Role）：角色用于将权限集合与SELinux用户和类型关联起来。进程必须拥有对应的角色才能访问某些类型的对象。
- 类型（Type）：也称为域（Domain）对于进程而言。类型是SELinux中最重要的部分，它为对象和主体定义了具体的安全策略。通过类型，SELinux策略可以非常细致地控制哪些主体可以访问哪些对象。
- 敏感度（Sensitivity）/清晰度（Clearance）：这些是用于多级安全（MLS）策略的可选部分，它们定义了数据的敏感度级别和主体的访问级别。

# Ansible
## Inventory file
使用Ansible之前，Inventory 文件是一个定义了可被 Ansible 管理的所有主机及其组的关键组件。这些文件可以是静态的或动态的，允许 Ansible 知道它可以连接哪些服务器，以及这些服务器如何组织。
```yaml
leafs:
    hosts:
        leaf01:
            ansible_host: 10.16.10.11
        leaf02:
            ansible_host: 10.16.10.12
spines:
    hosts:
        spine01:
            ansible_host: 10.16.10.13
        spine02:
            ansible_host: 10.16.10.14
```
有了inventory file之后，可以使用ansible命令行工具在主机上运行inventory中的指令，例如使用ping检查inventory中所有主机的连接。
```shell
ansible -i hosts.ini local -m ping
```
## Playbook
Playbook能描述比命令行更复杂的配置，例如：
```yaml
---
- name: Ping all hosts
  hosts: all
  tasks:
    - name: Check connectivity
      ping:
      loop: "{{ range(1, 4) | list }}"
      loop_control:
        label: "Ping number {{ item }}"
```

## Playbook的结构
Playbook由以下几个部分组成
- **Plays:** 运行任务的基本单元。每个 Play 指定了目标主机、需要执行的 Tasks，以及可能的一些变量设置。一个 Playbook 可以包含一个或多个 Play，每个 Play 为不同的主机或主机组定义操作。
- **Tasks:** 是 Play 中执行的具体操作。每个 Task 通常调用一个 Ansible Module，用于执行特定的操作，如安装包、编辑文件或同步目录等。
- **Modules:** 是 Ansible 的功能单元，它们可以独立执行或由 Task 调用。每个 Module 设计为处理特定的任务，如管理系统包（yum, apt）、操作文件系统（file, copy）等。
- **Become:** 是一种权限提升机制，允许用户任务在目标主机上以其他用户的权限执行。这在需要执行管理员权限的操作时非常有用。例如，使用 ```become: yes``` 可以使任务以 root 用户执行。
- **When:** 一种条件语句，用于控制 Tasks 的执行。只有当指定的条件满足时，相关的 Task 才会执行。
- **Register:** Register用于捕获 Task 的输出并将其存储到一个变量中。
- **Loops:** 允许在 Playbook 中重复执行一个 Task 多次，每次可以使用不同的数据。这对于执行批量操作（如创建多个用户、安装多个软件包等）非常有用。
# Networking
## Protocol and Protocol Stacks
协议是一组规则和标准，用于定义数据在网络中的传输方式。这些规则包括数据如何被格式化、传输、接收，以及如何对错误进行响应。

网络协议栈，是一组工作在不同层次上的网络协议的集合。每一层都承担特定的网络通信子任务，通过层与层之间定义良好的接口相互作用。数据在一个层处理后，会传递到另一个层进一步处理，直到发送到网络上，接收方则按相反顺序处理数据。

> 两个主要的协议栈：OSI和TCP/IPTCP/IP
### OSI模型
OSI模型有7层
- Layer 7: 应用层(Application)
- Layer 6: 表示层(Presentation)
- Layer 5: 会话层(Session)
- Layer 4: 传输层(Transport)
- Layer 3: 网络层(Network)
- Layer 2: 数据链路层(Data Link)
- Layer 1: 物理层(Physical)

# Hosting

# Storage

# Typescript

# Kubernetes

# React

# Next.js
