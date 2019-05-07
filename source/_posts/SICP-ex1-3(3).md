---
title: SICP 1.3节习题(3)
date: 2016-03-23 20:10:07
tags: "SICP解题集"
categories: "SICP解题集"
---

# 第一章 构造过程抽象
  
## 1.3 用高阶函数做抽象

### EX 1.40

```
;定义过程cubic
(define (cubic a b c)
  (lambda (x) (+ (cube x) (* a (square x)) (* b x) c)))
```

### EX 1.41

```
(define (double f)
  (lambda (x)
    (f (f x))))
```

```
(((double (double double)) inc) 5)  ;答案是21
```

分析: 
((double inc) 5) 					将inc应用2^1次,结果为7
(((double double) inc) 5)			将inc应用2^2次,结果为9
((double (double double)) inc) 5)	将inc应用(2^2)^2次,结果为21

### EX 1.42

```
(define (compose f g)
  (lambda (x)
    (f (g x))))
```

### EX 1.43

```
;递归版本
(define (repeated f n)
  (if (= n 1)
      f
      (lambda (x)
        (f ((repeated f (- n 1)) x)))))
;迭代版本
(define (re-iter f n)
  (define (iter i f-result)
    (if (= i n)
        f-result
        (iter (+ i 1) (lambda (x) (f (f-result x))))))
  (iter 1 f))
```

根据提示,可以使用EX 1.42的compose过程,省去了使用lambda

```
;使用compose的递归版本
(define (repeated f n)
  (if (= n 1)
      f
      (compose f (repeated f (- n 1)))))
;使用compose的迭代版本
(define (repeated f n)
  (define (iter i f-result)
    (if (= i n)
        f-result
        (iter (+ i 1) (compose f f-result))))
  (iter 1 f))
```

### EX 1.44

```
;一次平滑
(define (smooth f)
  (define dx 0.000001)
  (lambda (x)
    (/ (+ (f (- x dx)) (f x) (f (+ x dx))) 3))) 
;n次平滑
(define (n-smooth f n)
  (re1 smooth n) f)
```