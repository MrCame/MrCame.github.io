---
title: SICP 1.3节习题(1)
date: 2016-03-21 01:13:58
tags: "SICP解题集"
categories: "SICP解题集"
---

# 第一章 构造过程抽象
  
## 1.3 用高阶函数做抽象

### EX 1.29

```
;38页递归版本sum
(define (sum term a next b)
  (if (> a b)
      0
      (+ (term a) (sum term (next a) next b))))
(define (simpson f a b n)
  (define h (/ (- b a) n))
  (define (yk k) (f (+ a (* k h))))
  (define (next k) (+ k 1))
  (define (coefficient k)
    (cond ((or (= k 0) (= k n)) 1)
          ((odd? k) 4)
          (else 2)))
  (define (term k)
    (* (coefficient k) (yk k)))
  (* (sum term 0 next n) (/ h 3)))
```

结果
![“1-29”](/images/ex1-29.png)

### EX 1.30

```
;sum的迭代版本
(define (sum term a next b)
  (define (iter a result)
    (if (> a b)
        result
        (iter (next a) (+ (term a) result))))
  (iter a 0))
```

### EX 1.31

#### a)
```
;类似于sum的过程product,递归版本
(define (product f a next b)
  (if (> a b)
      1
      (* (f a) (product f (next a) next b))))
;用product定义阶乘过程factorial
(define (factorial n)
  (define (f x) x)
  (define (next i) (+ i 1))
  (product f
           1
           next
           n))
;计算pi值,首先计算每一个分数
(define (fraction i)
  (if (odd? i)
      (/ (+ i 1) (+ i 2))
      (/ (+ i 2) (+ i 1))))
;计算pi值,n是精确度
(define (pi n)
  (define (next i) (+ i 1))
  (* (product fraction 1.0 next n) 4.0))
```

结果
![“1-31a”](/images/ex1-31a.png)

#### b)
 
```
;product的迭代版本
(define (product f a next b)
  (define (iter a result)
    (if (> a b)
        result
        (iter (next a) (* (f a) result))))
  (iter a 1))
```