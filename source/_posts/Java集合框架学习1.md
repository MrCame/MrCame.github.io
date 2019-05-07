---
title: Java集合框架学习(1)——概览
date: 2017-02-10 15:12:21
tags: "Java"
categories: "Java"
---
***
从源码入手学习Java集合框架，选用的JDK版本是1.8.0_111。首先当然是理清java.util下的各个集合类与接口的关系。整理了一下，如下图所示，关系应该比较清晰了吧。
!["Collection"](/images/java-2-0.png)
当然经历了几个JDK版本的变化，集合框架也越来越庞大，最基本的东西却是没怎么变。
核心是**以collection为父类的接口**和**以AbstractCollection为父类的抽象类**，从这两部分入手，顺藤摸瓜，就可以理清整个框架的脉络，然后结合各个具体集合的实现来温习学习过的数据结构和算法。
比较奇怪的一点是Iterable接口是在java.lang中，而不是在java.util中。为什么Collection不直接继承Iterator呢？有这么一种说法：
JDK的作者之所以采用Iterator设计模式来设计，因为如果Collection直接从Iterator继承，那么Collection的实现类必须直接实现hasNext, next, remove方法。这么做有以下缺点：
+ 这么做会造成代码混乱，迭代代码与Collection本身实现代码混淆在一起，造成阅读困难，而且有方法重复，比如remove，不能做到迭代与本身实现分离。
+ 在Collection实现类中必须包含当前cursor指针，在并发时处理相当尴尬。
+ 访问接口不统一，Collection从Iterable继承的话，在迭代时只需拿到其Iterator(内部类实现)，用统一的对象迭代，而且多个迭代器可以做到互不干扰。

从接口看，整体可以分为两大类：Collection和Map。因为Map表示的是关联式容器。接下来，将从比较独立的Iterator来进行分析。