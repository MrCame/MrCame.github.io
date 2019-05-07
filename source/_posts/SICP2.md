---
title: 《计算机程序的解释和构造(SICP)》 (2)
date: 2016-03-24 11:23:50
tags: "SICP"
categories: "SICP"
---

断断续续看完第一章的三节公开课，做完第一章的课后习题，回顾一下。

+ Lisp的基本语法

+ 组合式的概念

    例如: (+ 3 (* 5 6))

+ 表达式类型
 + numbers 
 + symbol 
 + lambda-expressions 
 + definitions 
 + conditionals(条件表达式) 
 + combinations(组合式)

    例如：

	```
	(define  a  (* 5 5))  ;a是符号
	(define (a) (* 5 5))  ;a是过程
	```

+ 应用序和正则序

    应用序：先求参后应用
    正则序：先展开后规约

+ 线性递归(Linear-Recurison)与迭代(Iteration)，时间复杂度、空间复杂度简单分析

+ 函数式编程(functional programming)将过程(函数)作为参数，返回值等等

    例如：
	
```
(define (compose f g)
  (lambda (x)
    (f (g x))))
```
	

+ 复杂问题的分析与分解，划分成一个个自问题来解决

+ 语法糖衣(syntactic sugar)

 + lambda

    以下两种定义等价，都是f(x) = x + 1

```
(define (f x) (+ x 1))
```

```
(define f 
  (lambda (x) 
    (+ x 1)))
```

 + let

    以下两种定义等价，都是f(x) = 2x + 1
	
```
(define (f x)
  ((lambda (a)		;a + x 
    (+ a x))
  (+ 1 x))		;a = x + 1
```

```
(define (f x)
  (let ((a (+ x 1)))	;a = x + 1
    (+ a x)))		;a + x
```