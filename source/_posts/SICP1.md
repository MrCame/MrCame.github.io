---
title: 《计算机程序的解释和构造(SICP)》 (1)
date: 2016-03-04 18:09:43
tags: "SICP"
categories: "SICP"
---

最近开始看SICP这本神书，先草草看了前三章半，看了一点题目，最近开始跟着两位作者的公开课学习。先说一下对这本书的整体感觉吧，抽象程度比较高，对于计算机程序的理解不再停留于语法的层面，而是剖开现象看本质，探讨计算机程序的本质内容——具象化人的思维。

如作者所言，对CS而言，关键在于
> Techniques for controlling the complexity of large system —— 控制大型系统中的复杂度

而这，也是这本书所讨论的核心问题，如何降低复杂度，一层一层抽象过程，简化难度。概括下来方法有以下三点：
> 1. Black-box abstraction ——黑盒抽象
> 2. Establishing conventional interfaces ——按约定实现接口
> 3. making new languages ——定义新的语言

而以上三点，也对应了书中前四章的内容，在控制复杂度的过程中，必然涉及到程序架构的问题，作者也给出了两个方向，operations on aggregates (stream)——聚集操作，和著名的object-oriented——面向对象。也许是因为MIT的CS专业是和EE紧密相连的，各种电气系统的概念总能看到。

OO编程中，各对象通过消息传递进行联系，也联想到OS里的信号量机制(伟大的Dijkstra)，对于流操作，作者又将其类比于大型电气系统，因为我本科也是EE的，对信号与系统，滤波器这些概念听起来还有些怀念。越发佩服这些作者融会贯通的能力。

另外一个收获是对于一种新语言的思考
> General framework ——利用通用框架体系组织语言

对于新语言，探究
> 1. Primitive Elements ——基本元素
> 2. How to put together ——组合元素
> 3. Means of abstraction ——如何抽象(黑盒)

而以上的这些，都是之前的书本没有了解到的，站在一个新的角度重新思考了计算机程序