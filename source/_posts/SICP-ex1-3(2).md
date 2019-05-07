---
title: SICP 1.3节习题(2)
date: 2016-03-22 20:55:27
tags: "SICP解题集"
categories: "SICP解题集"
---

# 第一章 构造过程抽象
  
## 1.3 用高阶函数做抽象

### EX 1.32

#### a)

```
;accumulate递归版本
(define (accumulate combiner null-value term a next b)
  (if (> a b)
      null-value
      (combiner (term a)
                (accumulate combiner null-value term (next a) next b))))
;sum测试,计算立方和
(define (cube-sum a b)
  (accumulate + 0 cube a next b))
;product测试,计算阶乘
(define (factorial n)
  (define (f x) x)
  (accumulate * 1 f 1 next n))
```

    (cube-sum 1 10)
	(factorial 6)

结果
![“1-32”](/images/ex1-32.png)

#### b)

```
;accumulate迭代版本
(define (accumulate combiner null-value term a next b)
  (define (iter a result)
    (if (> a b)
        result
        (iter (next a) (combiner (term a) result))))
  (iter a null-value))
```

### EX 1.33

#### a)

```
;相比EX 1.32,增加参数filter,用于过滤一些项
(define (filtered-accumulate combiner null-value filter f a next b)
  (if (> a b)
      null-value
      (if (filter a)
          (combiner (f a) (filtered-accumulate combiner null-value filter f (next a) next b))
          (filtered-accumulate combiner null-value filter f (next a) next b))))
;计算a到b之间素数和
(define (prime-sum a b)
  (define (f x) x)
  (filtered-accumulate + 0 prime? f a next b))
;prime?见书P33页
```

其实可以使用let来定义剩余项,因为按照书中课后习题的学习顺序,这里并没有使用let

    (let rest (filtered-accumulate combiner null-value filter f (next a) next b))
	
#### b)

```
(define (gcd-product n)
  (define (f x) x)
  (define (its-prime? x) (= (gcd x n) 1))
  (filtered-accumulate * 1 its-prime? f 1 next n))
```

### EX 1.34

    (f f)		-> 
	(f 2)		->
	(2 2)		-> 解释器无法解释(2 2),因为2并不是一个过程(not a procedure)
	
### EX 1.35

```
;fixed-point见书P46页
(define golden-ratio
  (fixed-point (lambda (x) (+ (/ 1 x) 1)) 1.0))
```

    golden-ratio is 1.6180327868852458
	
### EX 1.36

```
;可以打印近似值序列的new-fixed-point, close-enough见书P45页
(define (new-fixed-point f first-guess)
  (define (close-enough? v1 v2) (< (abs (- v1 v2)) tolerance))
  (define (try guess step)
    (let ((next (f guess)))
      (cond ((close-enough? guess next) next)
            (else
             (display "step: ")
             (display step)
             (display ", ")
             (display next)
             (newline)
             (try next (+ step 1))))))
  (try first-guess 1))
;计算公式
(define loglog (lambda (x) (/ (log 1000) (log x))))
;平均阻尼计算过程,见书P48页
(define (average-damp f) (lambda (x) (average x (f x))))
;不使用平均阻尼
(define (no-damp-loglog x) (new-fixed-point loglog x))
;使用平均阻尼
(define (damp-loglog x) (new-fixed-point (average-damp loglog) x))
```

使用平均阻尼，可以使过程更快的收敛，因此计算步数更少。

结果
![“1-36-1”](/images/ex1-36-1.png)
![“1-36-2”](/images/ex1-36-2.png)

### EX 1.37

#### a)

```
;递归版本
(define (cont-frac n d k)
  (define (f i)
    (if (= k i)
        (/ (n i) (d i))
        (/ (n i) (+ (f (+ i 1)) (d i)))))
  (f 1))
;测试是否逼近
(define (check-cont-frac k)
  (cont-frac (lambda (i) 1.0)
             (lambda (i) 1.0)
             k))
```

需要至少k=11，具有十进制的4位精度

结果
![“1-37”](/images/ex1-37.png)

#### b)

```
;迭代版本
(define (cont-frac n d k)
  (define (iter i result)
    (if (= i 0)
        result
        (iter (- i 1) (/ (n i) (+ (d i) result)))))
  (iter (- k 1) (/ (n k) (d k))))
```

### EX 1.38

```
(define (e k)
  (define (di i)
    (if (= (remainder i 3) 2)
        (* (/ (+ i 1) 3) 2)
        1))
  (+ (cont-frac (lambda (ni) 1.0) di k) 2.0))
```

问题的关键在于找到Di序列的规律,发现三个数为一组,发现mod 3 = 2时,Di不等于1。随着k的增大，e的值精确度越高

结果
![“1-38”](/images/ex1-38.png)

### EX 1.39

```
(define (tan-cf x k)
  (define (di i)
    (if (= i 1)
        1
        (- (* i 2) 1)))
  (define (ni i)
    (if (= i 1)
        x
        (- (square x))))
  (cont-frac ni di k))
```

问题的关键在于生成奇数序列Di和x序列Ni