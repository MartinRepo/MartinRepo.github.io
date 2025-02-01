---
title: "对汇编语言中DF标志，CLD，STD，REP等指令的理解"
date: 2021-11-16T20:30:41+01:00
draft: false
author: ["Martin"]
tags: 
- 汇编语言
description: ""
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: "assemble"
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
看了几篇博客，感觉博主们把问题描述的好专业，导致本菜看不太懂qwq，这里写下我的个人理解，还望各位大佬多多指正，谢谢！

首先，DF作为标志寄存器的一个位，其值只能是0或1。

若DF = 0  每次操作后si, di递增

若DF = 1  每次操作后si, di递减

好了，我们现在只要知道这些就够了，至于怎么改变DF的值，使其变成0或1，后面我会讨论到。

si, di作为变址寄存器，其值递增或递减的本质就是要逐个取得ds:si/ds:di地址上的数据。

于是我们很难不联想到对于各种串传送的操作，因为串传送就要一个一个地将数据传送到想要的位置，接下来我介绍两个串传送指令：movsb和movsw。

我以movsb为例（movsw同理 只不过其单位是字word）：

```
;movsb指令相当于做如下操作：
 
((es)*16 + (di)) = ((ds)*16 + (di))  ;将ds:di指向的数据传送到es:si指向的地址
 
if(df == 0)  inc si  inc di
 
if(df == 1)  dec si  dec di
```
很明显，movsb指令是将一处内存单元中的字节byte送入另一处，然后判断DF的值来决定是递增还是递减。回归到递增递减的本质可以发现，递增就是将这个数据串中的数据正向传入，递减就是反向传入。

举例：一个数据串’12345678‘，正向传入的结果是’12345678‘，反向传入的结果是’87654321‘。

那么作为程序员我们如何控制其方向，总不能是机器说了算吧，于是我们有了CLD和STD指令

CLD：将DF值设为0  

STD：将DF值设为1 

于是 当写出CLD时，DF为0，si和di递增，则正向传递

当写出STD时，DF为1，si和di递减，则反向传递

所以CLD和STD可被理解为用来设置传递方向，这也是为什么DF叫做Direction Flag。

此外！！教给大家一个指令 用来直接传送数据串，

即rep movsb，可通过循环实现（cx）个字符的传送。

PS: [转载我自己的文章，无所吊谓了](https://blog.csdn.net/m0_60941819/article/details/121363410?spm=1001.2014.3001.5501)
