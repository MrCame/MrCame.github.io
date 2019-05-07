---
title: SICP 1.2节习题(1)
date: 2016-03-07 19:45:13
tags: "SICP解题集"
categories: "SICP解题集"
---

# 第一章 构造过程抽象
  
## 1.2 过程与它们所产生的计算

### EX 1.9

过程1

    (+ 4 5) 							——>
	(inc (+ 3 5)) 						——>
	(inc (inc (+ 2 5))) 				——>
	(inc (inc (inc (+ 1 5)))) 			——>
	(inc (inc (inc (inc (+ 0 5))))) 	——>
	(inc (inc (inc (inc 5)))) 			——>
	(inc (inc (inc 6))) 				——>
	(inc (inc 7)) 						——>
	(inc 8) 							——>
	9

过程2

    (+ 4 5)				——>
	(+ 3 6)				——>
	(+ 2 7)				——>
	(+ 1 8)				——>
	(+ 0 9)				——>
	9

明显，过程1是线性递归(Linear-Recursion)，过程2是迭代(Iteration)。如公开课(链接)Lec1b里讲到的那样，
对于过程1，时间复杂度和空间复杂度都随着问题规模的增大而增大
> time = O(a)
> space = O(a)
对于过程2，显然空间复杂度会是一个常数(当然这也并不完全准确，随着计算的数增大，所需要的空间也在变化)
> time = O(a)
> space = O(1)

### EX 1.10

    (A 1 10) 	;answer is 1024
	(A 2 4)		;answer is 65536
	(A 3 3)		;answer is 65536
	(define (f n) (A 0 n))		;f(n)=2n
	(define (g n) (A 1 n))		;g(n)=2^n
	(define (k n) (A 2 n))		;k(n)=2^2^2^…… 连续求n次平方

### EX 1.11

    递归版本
	(define (f n)
		(if (< n 3)
      		n
      		(+ (f (- n 1)) (* 2 (f (- n 2))) (* 3 (f (- n 3))))))

    迭代版本
	(define (f n)
		(define (iter n a b c i)
			(if (= i n)
				c
				(iter n 
					  (+ a (* 2 b) (* 3 c))
					  a
					  b
					  (+ i 1))))
		(iter n 2 1 0 0))
迭代版本需要观察
f(0)=0, f(1)=1, f(2)=2
f(3)=f(2)+2f(1)+3f(0)
f(4)=f(3)+2f(2)+3f(1)
f(5)=f(4)+2f(3)+3f(2)
a、b、c分别表示f(n-1) f(n-2) f(n-3)
这里的i，个人认为可以类比其他语言中循环语法里的计数变量i，而迭代器iter可以类比于for、while等循环

### EX 1.12

```
(define (pascal r c)
	(cond ((or (= r c) (= c 0)) 1)
    	  ((> c r) "error")
    	  (else (+ (pascal (- r 1) (- c 1)) (pascal (- r 1) c)))))
```

### EX 1.13 1.14略

### EX 1.15

    a) p将被使用五次
	(sine 12.15)						——>
	(p (sine 12.15/3))					——>
	(p (p (sine 12.15/6)))				——>
	(p (p (p (sine 12.15/9))))			——>
	(p (p (p (p (p (sine 12.15/12)))))	——>
	(p (p (p (p (p (sine 0.0514……)))))	——>
	…………

	b) 求值(sine a)时候，除以3.0，sine是递归程序，因此时间复杂度和空间复杂度都是O(log a)

### EX 1.16

```
(define (even? x)
  (= (remainder x 2) 0))
(define (square x)
	(* x x))
(define (exp b n)
	(define (exp-iter b n p)
		(cond ((= n 0) p)
      		  ((even? n) (exp-iter (square b) (/ n 2) p))
      		  (else (exp-iter b (- n 1) (* b p)))))
	(exp-iter b n 1))
```

### EX 1.17

```
(define (double x)
  (+ x x))
(define (halve x)
  (/ x 2))
(define (even? x)
  (= (remainder x 2) 0))
(define (expt b n)
  (cond ((= n 0) 0)
        ((even? n) (double (expt b (halve n))))
        (else (+ b (expt b (- n 1))))))
```

### EX 1.18

```
(define (double x)
  (+ x x))
(define (halve x)
  (/ x 2))
(define (even? x)
  (= (remainder x 2) 0))
(define (multi a b)
  (define (multi-iter a b p)
    (cond ((= b 0) p)
          ((even? b) (multi-iter (double a) (halve b) p))
          (else (multi-iter a (- b 1) (+ a p)))))
  (multi-iter a b 0))
```

### EX 1.19
通过两次变换，可以得到
p' = p^2 + q^2
q' = 2pq + q^2

```
(define (even? x)
  (= (remainder x 2) 0))
(define (fib n)
  (define (fib-iter a b p q count)
    (cond ((= count 0) b)
          ((even? count)
           (fib-iter a
                     b
                     (+ (square p) (square q))
                     (+ (* 2 p q) (square q))
                     (/ count 2)))
          (else (fib-iter (+ (* b q) (* a q) (* a p))
                          (+ (* b p) (* a q))
                          p
                          q
                          (- count 1)))))
  (fib-iter 1 0 0 1 n))
```

### EX 1.20

    正则序
	(gcd 206 40)		//a=206,b=60				——>
	(gcd 40 (r 206 40)) //a=40,b=6,r1=(r 206 40)	——>r1:1
	(gcd r1 (r 40 r1))  //a=6,b=4,r2=(r 40 r1)		——>r2:1+1=2
	(gcd r2 (r r1 r2))  //a=4,b=2,r3=(r r1 r2)		——>r3:1+1+2=4
	(gcd r3 (r r2 r3))  //a=2,b=0,r4=(r r2 r3)		——>1+2+4=7
	r3 = (r r1 r2)
		|
		V
	(r (r 206 40) (r 40 (r 206 40))					——>1
		|
		V
	(r (r 206 40) (r 40 6))
		|
		V
	(r 6 4)											——>2
	2												——>1

	正则序执行了1+2+4+7+1+2+1=18次remainder运算

    应用序
	(gcd 206 40)
	(gcd 40 (r 206 40))  		——>1
	(gcd 40 6)
	(gcd 6 (r 40 6)) 			——>1
	(gcd 6 4)
	(gcd 4 (r 6 4))  			——>1
	(gcd 4 2)
	(gcd 2 (r 4 2)) 			——>1
	(gcd 2 0)
	2
	
	应用序执行了1+1+1+1次remainder运算