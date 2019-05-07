---
title: SICP 1.1节习题
date: 2016-03-04 20:01:25
tags: "SICP解题集"
categories: "SICP解题集"
---

# 第一章 构造过程抽象
  
## 1.1 程序设计的基本元素

ex 1.1 ~ ex 1.4都比较容易理解，就不写出结果了

### EX 1.5

```    
(define (p) (p))

(define (test x y)
  (if (= x 0)
       0
       y))
```

求值

    (test 0 (P))
分析：

> 采用应用序，先求参数值再应用， (p)无限递归调用自身，编译器停滞
> 采用正则序，先展开后规约求值，实参0和(p)不会先求值，解释到if之后，返回0

### EX 1.6
```
(define (new-if predicate then-clause else-clause)
  (cond (predicate then-clause)
        (else else-clause)))
```
对于这个程序题中演示的使用

    (new-if (= 2 3) 0 5)
    (new-if (= 1 1) 0 5)
都是没错的，但若用来重写sqrt程序的话
![“error”](/images/ex1-6.png)

分析：错误原因是超过递归最大深度
**new-if** 作为定义的一个过程，由于scheme采用应用序，先求参数值再应用，调用顺序
**sqrt-iter -> new-if -> cond -> sqrt-iter -> new-if -> …… -> ** 最终超出栈最大深度

### EX 1.7
原始程序
![“很大的数”](/images/ex1-7-1.png)
![“很小的数”](/images/ex1-7-2.png)
当数字很大时，程序跑不动，当数字很小时，结果错误

修改过程**good-enough**为

```
(define (good-enough? guess x)
	(< 
	  (/ (abs (- x guess)) guess) 
	  0.001))
```
对于很大和很小的数字都可以计算平方根

### EX 1.8
```
(define (square x)
  (* x x))
(define (cube x)
  (* x x x))
(define (cube-root x)
  (define (good-enough? guess)
    (< (abs (- (cube guess) x)) 0.001))
  (define (improve guess)
    (/ (+ (/ x (square guess)) (* 2 guess)) 3))
  (define (cube-iter guess)
    (if (good-enough? guess)
        guess
        (cube-iter (improve guess))))
  (cube-iter 1.0))
```