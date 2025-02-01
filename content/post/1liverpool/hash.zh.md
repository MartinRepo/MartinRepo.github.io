---
title: "详解java中的几种hash"
keywords:
- hashtable
- hashmap
- hashset
- java
date: 2023-01-01T15:37:32Z
lastmod: 2023-01-10T21:02:21Z
draft: false
author: ["Martin"]
categories: 
- 分类1
- 分类2
tags: 
- java
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
---
刷leetcode总能看见哈希表，对哈希表有大致的了解，但是一直不知道HashTable，HashMap和HashSet在Java中的区别。今天看到一个博主的文章再结合官方文档的学习，受益良多，记录一下。

(:  [点我进入官方文档](https://docs.oracle.com/en/java/javase/15/docs/api/java.base/java/util/Hashtable.html) :)
# HashTable
## 负载因子（load factor）
我觉得有必要对负载因子进行一波解释，方便更多人看懂这篇文章。

负载因子：构造hashtable时，初始容量和负载因子决定了容器该什么时候扩容。当负载系数过大时，查询效率大幅降低，但是空间利用率上升，因为相同容量下能存的数据更多；而负载系数太小的话，空间利用率会变的非常低，相同的容量可能存不了几个数据就要扩容。所以根据官方文档的建议，负载因子默认为0.75。

## 文档定义
``` java
public class Hashtable<K,​V>
extends Dictionary<K,​V>
implements Map<K,​V>, Cloneable, Serializable
```
继承了Dictionary并且实现了Map接口以及接口的可克隆化和序列化

可克隆化：可以被克隆一个一样的

序列化：在进行IO操作时将对象数据转换为字节流，之后将字节流数据转换为特定的对象。
## HashTable的几个常用方法
| 返回类型| 方法 | 功能|
| -----------  | ----------- |-----------|
| boolean| contains(Object value)|检查某个键是否映射到这个hashtable中的指定值。           |
| boolean| containsKey(Object key)|检查指定的对象是否是这个hashtable中的一个键。           |
| boolean| containsValue(Object value)|如果这个hashtable将一个或多个键映射到这个值，则返回true。           |
| V | 	put​(K key, V value)|将指定的V映射到该hashtable中的指定K。|
| V | get(Object key) | 返回指定键所映射的值，如果不存在这个映射，则返回null|
| V | remove​(Object key)|从hashtable中删除键（和它对应的值）。|
| boolean | 	isEmpty()| 检查这个hashtable是否有将键映射到值。|
| V | getOrDefault(Object key, V defaultValue) | 返回指定的键被映射到的值，如果没有键的映射的值，则返回defaultValue。|



PS: V -- 映射值的类型  K -- 映射所维护的键的类型

<font color=red>Hashtable类实现了哈希表，可以将键和值进行对应。Hashtable支持同步，但不支持空值。由于其同步特性，它是线程安全的。</font>

# HashMap

## 文档定义
``` java
public class HashMap<K,​V>
extends AbstractMap<K,​V>
implements Map<K,​V>, Cloneable, Serializable
```
显然，HashMap和HashTable功能基本相同，只不过HashMap时继承AbstractMap
## HashMap的几个常用方法
常用的几个方法与HashTable基本相同，查看上面的表格即可。

## HashMap与HashTable的区别
主要区别：线程安全，空值，同步，速度
1. Hashtable是线程安全，而HashMap非线程安全.
2. HashMap可以使用null作为key，而Hashtable则不允许null作为key
3. HashMap的初始容量为16，Hashtable初始容量为11，两者的填充因子默认都是0.75
    - HashMap扩容时：new capacity =  2*capacity 
    - Hashtable扩容时：new capacity = 2*capacity + 1
4. 两者计算hash的方法不同
# HashSet

## 文档定义
``` java
public class HashSet<E>
extends AbstractSet<E>
implements Set<E>, Cloneable, Serializable
```
可以看到，HashSet继承了AbstractSet，实现了Set接口。HashSet是基于HashMap来实现的，是一个不允许有重复元素的集合。
因此，HashSet也允许有空值，而且HashSet是线程不安全的。

## HashSet的几个常用方法
| 返回类型| 方法 | 功能|
| -----------  | ----------- |-----------|
| boolean| add(E e)|如果指定的元素还没有出现，则将其添加到这个集合中。           |
| boolean| contains(Object o)|如果这个集合包含指定的元素，则返回真。           |
| boolean | 	isEmpty()| 如果这个集合不包含任何元素，则返回true。|
|int| size()|返回这个集合中的元素数量。|

<font color=red>简单理解：HashSet就是简单的HashMap</font>

# 参考文章👇
[Java中HashSet、HashMap和HashTable的区别](https://juejin.cn/post/7082318379591303176)