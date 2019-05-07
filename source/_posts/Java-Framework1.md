---
title: 《从零开始写Java Web框架》——笔记(1)
date: 2017-02-27 11:30:40
tags: "Java"
categories: "Java"
---
***
第一章环境搭建，第二章示范了如何使用书中所开发的Smart Framework。核心是第三章的框架实现，几个重要类的关系如下图所示：

!["framework"](/images/javaFramework-1-1.png)

工具类中，核心的就是ClassUtil和ReflectionUtil了。
助手类中
+ HelperLoader负责加载几个助手类，可以视为入口

+ ClassHelper提供ClassSet，包括所有类的Set或者用特定注释的类的Set。例如：用Controller注释的类的集合ControllerSet

+ BeanHelper根据ClassSet中的每个Class，生成对应实例，提供BeanMap<Class<?>, Object>

+ IocHelper根据BeanMap得到每个Bean中用Inject注释的域类型及其object，再用Reflection动态设置该值（实现依赖注入）

+ ControllerHelper负责在静态块中根据ControllerSet得到Controller中的Action方法，解析request方法和路径，提供ActionMap<Request，Handler>，将请求映射到对应的控制器