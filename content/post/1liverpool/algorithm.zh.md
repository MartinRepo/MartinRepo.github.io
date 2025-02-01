---
title: "基础数据结构"
date: 2023-03-17T17:39:49Z
draft: false
author: ["Martin"]
tags: 
- Algorithm
description: ""
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: "basic-data-structures"
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
上星期被算法课考试狠狠拷打，痛思悔过一小时。发现自己学习算法缺乏实际操作，只知理论，操作甚少，导致对理论的理解也十分浅显。

所以从基础数据结构开始，重新系统学习算法，在这里记录学习的过程。

# Stacks and Queue
非常简单的两个概念。栈就是后进先出，队列是先进先出。栈和队列都是依靠数组实现，只不过是用上了指针，使其效果看起来像是栈或队列。
比如弹出操作。数组A[1,2,3,4,5,6]，弹出位置6处的元素，并不是位置6真的在这个结构中消失了，而是定义了一个指针，当pop(A)的时候，将指针向前挪一位到5。

栈中主要有三个方法：push（推入栈），pop（弹出栈），还有判断栈是否为空，因为栈为空了就不能进行pop操作了。

伪代码也很好理解👇
```pseudo code
STACK-EMPTY(S)
// S.top表示栈顶元素，在数组的末尾。
if S.top==0
    return true
else return false

PUSH(S, x)
S.top=S.top+1
S[S.top]=x

POP(S)
if STACK-EMPTY(S)
    error "underflow"
else S.top=S.top-1
    return S[S.top+1]
```
**队列**

队列和栈非常相似，也是一个数组。和栈不同的是，队列有两个指针，一个指向队列头部一个指向队列尾部（头部指向第一个元素，尾部指向的是最后一个元素的后面一个位置）。当插入一个元素时，队列的尾部+1，当删除一个元素时，队列的头部+1。
当队列的头和尾两个指针指向的索引位置相同时，队列为空，这是再想拿出来元素，就会underflow。当头部指针=尾部指针+1 或 头部=1，尾部=数组长度，那么队列就是满载的。这是想再插入数据，就会overflow。

```
//这里暂时省略判断underflow和overflow的逻辑
// ENQUEUE 向队列中插入数据
ENQUEUE(Q, x)
Q[Q.tail] = x
if Q.tail == Q.length
    // 判断是否满载，满载把尾部索引设置回到头部，提示满载
    Q.tail = 1
else Q.tail = Q.tail+1

//DEQUEUE 从队列中删除数据
DEQUEUE(Q)
x = Q[Q.head]
if Q.head==Q.length
    Q.head = 1
else Q.head = Q.head+1
return x
```

栈和队列的所有操作都是在O(1)时间内
# Linked lists
链表是一种将对象以线性顺序排列的数据结构。和数组不同，数组是以索引来定义线性顺序。而链表中的顺序是有每个对象上的指针决定的。现在使用的都是双向链表，包含前后(prev, next)两个指针，也就是说，要使列表保持完整，插入的时候prev和next要分别赋值一次和被赋值一次。当x.next为空时，说明x是链表的尾部元素，当x.prev为空时，说明x是链表的头部元素。

链表还有单向链接的，这还分为有序和无序。还有循环链接的，就是头部元素的prev指针不指向空，而是指向尾部元素。

```
//k表示key，就是节点的值，x表示整个节点
LIST-SEARCH(L, k)
x = L.head
while x is not NULL and x.key is not k
    x = x.next
return x
```
search的复杂度是Θ(n),因为最坏情况可能要遍历整个链表元素

```
//插入操作是将x插在链表的头部，而不是尾部。
LIST-INSERT(L, x)
x.next = L.head //原来链表的头部赋值到x.next
if L.head is not NIL
    L.head.prev = x //把x赋值到链表头部.prev上，这样才完整
L.head = x //将头部设置成x
x.prev = NIL //将x.prev设置为空
```
insert的复杂度为O(1)。不用过多解释了

删除操作之前，应该调用search找到想要删除的key的位置。所以时间复杂度是Θ(n)，因为删除之前通常要search
```
LIST-DELETE
if x.prev is not NIL
    x.prev.next = x.next
else L.head = x.next
if x.next is not NIL
    x.next.prev = x.prev
```
对于删除操作，要多说一下关于x.prev.next = x.next。为什么不直接使用x=x.next?

现在已知一个列表1->2->3->4->5，如果删除操作中代码是x=x.next，那么结果会变成1->2->4->4->5，我们想要的结果是1->2->4->5。这就说明3处的节点依然存在，并没有被跳过。而使用x.prev.next = x.next的意思是，将x.prev指向的节点的next位置，赋值给x.next节点。这样是将2和4两个节点跳过3连接起来了。