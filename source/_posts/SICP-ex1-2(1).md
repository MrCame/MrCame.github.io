---
title: SICP 1.2节习题(2)
date: 2016-03-011 15:42:13
tags: "SICP解题集"
categories: "SICP解题集"
---

# 第一章 构造过程抽象
  
## 1.2 过程与它们所产生的计算

### EX 1.21

smallest-divisor程序如下

```
(define (smallest-divisor n)
	(find-divisor n 2))
(define (find-divisor n test-divisor)
	(cond ((> (square test-divisor) n) n)
		  ((divides? test-divisor n) test-divisor)
		  (else (find-divisor n (+ test-divisor 1)))))
(define (divides? a b)
	(= (remainder b a) 0))
```

    (smallest-divisor 199)		;answer is 199
	(smallest-divisor 1999)		;answer is 1999
	(smallest-divisor 19999)	;answer is 7

### EX 1.22

解决这个问题，可以把问题划分成几个子问题来解决，这也是SICP这本书所教授的核心问题职业——如何将复杂问题分解

> 如何生成连续奇数？
> 如何描述大于某数x的前i个素数？
> 如何判断素数？
> 如何计算程序运行时间？

```
;生成奇数
(define (odd-generator x)
	(if (odd? x)
		(+ x 2)
		(+ x 1)))
```

```
;大于x的前i个素数
(define (continue-prime i x)
	(cond ((= i 0) (display "These are all primes"))
		  ((prime? x)
			(display x)
			(newline)
			(continue-prime (- i 1) (odd-generator x)))
		  (else (continue-prime i (odd-generator x)))))
```

```
;判断素数,可以采用33页最基本的方法
(define (prime? n)
	(define (smallest-divisor n)
		(find-divisor n 2))
	(define (find-divisor n test-divisor)
		(cond ((> (square test-divisor) n) n)
			  ((divides? test-divisor n) test-divisor)
			  (else (find-divisor n (+ test-divisor 1)))))
	(define (divides? a b)
		(= (remainder b a) 0))
	(= n (smallest-divisor n)))
```

```
;运行时间计算
;(runtime)返回的是秒,(real-time-clock)返回毫秒
;最终的search-for程序:
(define (search-for-primes x)
	(let ((start-time (real-time-clock)))
	(continue-prime 3 x)
	(- (real-time-clock) start-time)))
```

结果
![“1-22”](/images/ex1-22.png)

结果分析放在1.24，显然并不遵循预期的结果

### EX 1.23

首先定义一个新的prime过程——new prime
```
(define (new-prime? n)
	(define (next x)
		(if (= x 2)
		    3
		    (+ x 2)))
	(define (smallest-divisor n)
		(find-divisor n 2))
	(define (find-divisor n test-divisor)
		(cond ((> (square test-divisor) n) n)
			  ((divides? test-divisor n) test-divisor)
			  (else (find-divisor n (next test-divisor)))))
	(define (divides? a b)
		(= (remainder b a) 0))
	(= n (smallest-divisor n)))
```

然后用这个新的素数判断过程重写continue-prime
```
(define (continue-newprime i x)
	(cond ((= i 0) (display "These are all primes"))
		  ((new-prime? x)
			(display x)
			(newline)
			(continue-prime (- i 1) (odd-generator x)))
		  (else (continue-prime i (odd-generator x)))))
```

最后重写search-for方法
```
(define (search-for-newprimes x)
	(let ((start-time (real-time-clock)))
	(continue-newprime 3 x)
	(- (real-time-clock) start-time)))
```

当然，scheme里新绑定的过程会覆盖原来的，可以局部替换为新的过程，为了更明显一些，都重写了这些过程
结果
![“1-23”](/images/ex1-23.png)

结果分析放在1.24，显然也并不遵循预期的结果

### EX 1.24

同1.23。使用费马定理来判断是否为素数
```
(define (fast-prime? n times)
  (define (expmod base exp m)
    (cond ((= exp 0) 1)
        ((even? exp)
         (remainder (square (expmod base (/ exp 2) m)) m))
        (else
         (remainder (* base (expmod base (- exp 1) m)) m))))
  (define (fermat-test n)
    (define (try-it a)
      (= (expmod a n n) a))
    (try-it (+ 1 (random (- n 1)))))
  (cond ((= times 0) true)
        ((fermat-test n) (fast-prime? n (- times 1)))
        (else false)))

(define (expmod base exp m)
  (cond ((= exp 0) 1)
        ((even? exp)
         (remainder (square (expmod base (/ exp 2) m)) m))
        (else
         (remainder (* base (expmod base (- exp 1) m)) m))))
```

重写continue-prime和search-for
```
(define (continue-fastprime i x)
	(cond ((= i 0) (display "These are all primes"))
		  ((fast-prime? x 100)
			(display x)
			(newline)
			(continue-prime (- i 1) (odd-generator x)))
		  (else (continue-prime i (odd-generator x)))))
(define (search-for-fastprimes x)
	(let ((start-time (real-time-clock)))
	(continue-fastprime 3 x)
	(- (real-time-clock) start-time)))
```
结果
![“1-24”](/images/ex1-24.png)

结果分析,1.22,1.23,1.24三个例子中，可以明显看到，随着问题复杂度的增加，用时并不按照理论耗时增长。结果可能有以下原因，机器的差异，运行环境的差异，资源的占用情况，编译器优化等等，都会影响到程序的运行速度。然而，随着复杂度逐渐加大，可以看到，复杂度更低的算法效率更高。

### EX 1.25

Alyssa的程序理论上更快，但是实际操作中发现，当底数和幂都很大的时候，使用fast-expt计算速度非常慢，而且占用大量CPU资源。究其原因，书中34页的expmod程序，每一次幂的结果都会进行remainder操作，从而将幂控制在一个合理的范围内，防止溢出。

### EX 1.26

显然，Louis的程序在偶次幂时，两次调用expmod。也看到有这样的分析：

    Louis的expmod使得原来的线性递归变成了具有两个分支的树形递归
	计算量是原来expmod复杂度的平方，即Θ(logn)的平方，即Θ(n)
	而prime?的复杂度是Θ(sqrt n)，所以比prime?还慢

### EX 1.27

```
;是否同余
(define (same-remainder? a n)
	(= (expmod a n n) a))

(define (carmichael n)
	(iter 1 n))
(define (iter a n)
	(cond ((= a n) "YES")
		  ((same-remainder? a n) (iter (+ a 1) n))
		  (else "NO")))
```

    (carmichael 561)
	(carmichael 1105)
	(carmichael 1729)
	(carmichael 2465)
	(carmichael 2821)
	(carmichael 6601)

以上几个数均骗过了费马检查

结果
![“1-27”](/images/ex1-27.png)

### EX 1.28

```
;求一个数是否是非凡平方根,即非零非n-1的数x，x平方模n结果为1
(define (special-square-root? x n)
	(and (= (remainder (square x) n) 1) 
		 (not (= x 0))
		 (not (= x (- n 1)))))

;改进expmod过程,增加判断非凡平方根
(define (expmod base exp m)
	(cond ((= exp 0) 1)
		  ((special-square-root? base m) 0)
		  ((even? exp)
			(remainder (square (expmod base (/ exp 2) m)) m))
		  (else
			(remainder (* (expmod base (- exp 1) m) base) m))))

;随机取一个数a,并且a<n
(define (rand-smaller-than-n n)
	(let ((x (random n)))
	(if (not (= x 0))
		x
		(rand-smaller-than-n n))))

;Miller-Rabin测试
(define (miller-rabin-test n)
	(let ((times n))
		(define (miller-prime? n times)
				  (cond ((= times 0) true)
				  ((= (expmod (rand-smaller-than-n n) (- n 1) n) 1) (miller-prime? n (- times 1)))
				  (else false)))
		(miller-prime? n times)))
```

    (miller-rabin-test 561)
	(miller-rabin-test 1105)
	(miller-rabin-test 1729)
	(miller-rabin-test 2465)
	(miller-rabin-test 2821)
	(miller-rabin-test 6601)
	(miller-rabin-test 13)
	(miller-rabin-test 19)
	(miller-rabin-test 23)

结果
![“1-28”](/images/ex1-28.png)
