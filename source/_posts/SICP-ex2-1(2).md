---
title: SICP 2.1节习题(2)
date: 2016-04-21 20:09:17
tags: "SICP解题集"
categories: "SICP解题集"
---

# 第二章 构造数据抽象

## 2.1 数据抽象引导

### EX 2.7

```
(define (upper-bound x) (cdr x))

(define (lower-bound x) (car x))

(define (make-interval a b) (cons a b))

(define test (make-interval 1 2))
```

### EX 2.8

```
(define (sub-interval x y)
  (make-interval (- (lower-bound x) (lower-bound y))
                 (- (upper-bound x) (upper-bound y))))
```

### EX 2.9

    显而易见，若两个区间分别为(1,2) (3,4)，被加宽度为0.5
    对于加：(4,6)，宽度为0.5+0.5=1
    对于减：(-2,-2),宽度为0.5-0.5=0
    对于乘：p1=3 p2=4 p3=6 p4=8，(3,8), 宽度为2.5
    对于除：第一个区间乘上第二个区间的倒数，p1=1/4 p2=1/3 p3=1/2 p4=2/3，(1/4,2/3)，宽度为5/24

### EX 2.10

    显然，如果两个区间分别为(0,1)，(1,2)
    对于除：取倒数的时候可能会出现分母为0的情况
    如果区间横跨0，区间取倒数的时候会出现上界小于下界的情况，(-1,2)取倒数后(1/2, -1)

```
(define (new-div-interval x y)
  (let ((y1 (lower-bound y))
        (y2 (upper-bound y)))
    (cond ((or (= y1 0) (= y2 0)) (display "error"))
          ((< y1 0) (display "error"))
          (else
           (mul-interval x
                         (make-interval (/ 1.0 y1)
                                        (/ 1.0 y2)))))))
```