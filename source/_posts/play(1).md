---
title: Play Framework环境搭建
date: 2016-03-16 13:38:22
tags: "Play Framework"
categories: "搭建环境"
---

# Play Framework 笔记(1)

从去年七月开始进入实验室，一直使用的就是Play Framework来进行Web开发，慢慢的应该总结一些东西，免得以后忘记。

先说说[Play](https://www.playframework.com/)这个框架吧，使用了一段时间，直观的感觉是：

* MVC架构很清晰，Model层针对数据封装、数据库操作，Controller层负责响应各个请求，View层主要就是一些Html的东西来呈现在浏览器上。
* Debug之后可以在浏览器上直接看到error

以下援引自[Wiki](https://en.wikipedia.org/wiki/Play_Framework)
> Play is an open source web application framework, written in Scala and Java, which follows the model–view–controller (mvc) architectural pattern. It aims to optimize developer productivity by using convention over configuration, hot code reloading and display of errors in the browser.
> Support for the Scala programming language has been available since version 1.1 of the framework.In version 2.0, the framework core was rewritten in Scala. Build and deployment was migrated to SBT, and templates use Scala instead of Groovy.

一个软件工程的术语[convention over configuration](https://en.wikipedia.org/wiki/Convention_over_configuration)
> 约定优于配置(convention over configuration)，也称作按约定编程，是一种软件设计范式，旨在减少软件开发人员需做决定的数量，获得简单的好处，而又不失灵活性。
> 本质是说，开发人员仅需规定应用中不符约定的部分。例如，如果模型中有个名为Sale的类，那么数据库中对应的表就会默认命名为sales。只有在偏离这一约定时，例如将该表命名为"products_sold"，才需写有关这个名字的配置。

一开始配环境的时候，被人告知应该装一个typesafe-activator，很懵逼的是不知道这个玩意儿和Play框架有什么关系，继续google。发现typesafe这公司已经更名为[Lightbend](https://www.lightbend.com/)了，当然这并没有什么影响。继续看看他们的[Activator](https://www.lightbend.com/activator/download)
> Activator is the Lightbend Reactive Platform's build and tutorial tool

看起来Activator是一个构建工具的样子，在[Play](https://www.playframework.com/documentation/2.5.x/Installing)可以看到教程，如何使用Lightbend Activator安装Play，以及一些相关的配置需求。最开始接触的Play 2.2,现在version已经到了2.5，感觉已经有点跟不上节奏了快。大致扫了一眼新版本的配置，相比之前越来越简洁了，activator也有UI界面，可以新建各个项目，当然Play项目也是可以的。
看了一下Lightbend的官网，这个公司做的东西也有一点意思，他们的产品Lightbend Reactive Platform包括以下几个

+ ConductR
+ Monitoring
+ Lagom
+ Apache Spark

Spark还算知道一点点，前三个完全是很陌生的东西了，随便查了一下，有这样的内容

> 《Lagom：一个新的微服务框架》——Lightbend，也就是Akka背后的公司，最近发布了一款开源的微服务框架Lagom，它构建在他们的Reactive平台之上，尤其是使用了Play框架和Akka的家族产品，并添加了ConductR用于部署。

对[Typesafe(Lightbend)](https://en.wikipedia.org/wiki/Typesafe_Inc.)这个公司看了一下

> Typesafe (now Lightbend) is a company founded by Martin Odersky, the creator of the Scala programming language, Jonas Bonér, the creator of the Akka middleware, and Paul Phillips in 2011. It provides an Open source platform for building Reactive applications for the JVM, consisting of the Play Framework, Akka middleware and Scala programming language—with additional supporting products and development tools such as the Scala IDE for Eclipse, the Slick database query and access library for Scala and the sbt build tool. Typesafe also provides training, consulting and commercial support on the platform.
> Typesafe is one of the main contributors of Reactive Streams.

这里有几个点值得关注

+ Akka: 简化并发问题的开源工具包
+ Reactive: Reactive Programming, 一种面向数据流和变化传播的编程范式
+ sbt: 构建Scala和Java项目的工具，类似于Maven和Ant

言归正传，Play的工程结构这样的，MVC层次分明。
![“play”](/images/play1.png)

在一开始配置play2.2的时候，遇到的主要的问题有

+ 集成在Intellij 13中
+ 依赖的jar包很多

我采用的办法是使用Activator直接new play application，再导入Intellij中。所依赖的jar包很多，只能从公司的服务器上把ivy仓库全部download下来覆盖本地。
后来用了intellij 14、15版本，对于Scala、playframework的集成好多了，直接装个Scala plugin，ivy一覆盖就行。