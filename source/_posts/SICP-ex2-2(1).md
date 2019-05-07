---
title: SICP 2.2节习题(1)
date: 2016-04-21 20:11:25
tags: "SICP解题集"
categories: "SICP解题集"
---

# 第二章 构造数据抽象

## 2.2 层次性数据和闭包性质

### EX 2.17

```
;返回列表尾元素
(define (last-pair x)
  (if (null? (cdr x))
      x
      (last-pair (cdr x))))
```

```
;测试
(last-pair (list 1 2 3 4))
```

### EX 2.18

```
;反转列表
(define (reverse x)
  (define (iter remain result)
    (if (null? remain)
        result
        (iter (cdr remain) (cons (car remain) result))))
  (iter x '()))
```

### EX 2.19

题目本身并不难，只要看过“换零钱”的程序，根据分治思想，分成使用的币种和为未使用的币种，两部分分别是list的car和cdr

```
;源程序
(define (cc amount coin-values)
  (cond ((= amount 0) 1)
        ((or (< amount 0) (no-more? coin-values)) 0)
        (else (+ (cc amount (except-first-denomination coin-values))
                 (cc (- amount (first-denomination coin-values)) coin-values)))))
```

```
;补全程序
(define (first-denomination coin-values)
  (car coin-values))
(define (except-first-denomination coin-values)
  (cdr coin-values))
(define (no-more? coin-values)
  (null? coin-values))
```

### EX 2.20

一般的思路是利用循环遍历列表，逐个判断元素的奇偶性，当该项符合要求时加入新的list中
这里的思路是抽象出一种通用的过程filter，体现了动态语言的特性，将过程(函数)作为参数传递

```
;P78 filter的定义
(define (filter predicate sequence)
  (cond ((null? sequence) '())
        ((predicate (car sequence))
         (cons (car sequence) (filter predicate (cdr sequence))))
        (else (filter predicate (cdr sequence)))))
```

```
;一种same-parity  
(define (same-parity x . y)
  (cond ((odd? x) (filter odd? (cons x y)))
        ((even? x) (filter even? (cons x y)))
        (else (display "x error"))))

;一种same-parity的简洁定义
(define (same-parity x . y)
  (filter (if (odd? x)
              odd?
              even?)
          (cons x y)))
```

### EX 2.21

不难补全代码
第一种定义：对car项做平方，cdr项继续迭代回过程中
第二种定义：根据map的定义，形式参数是操作类型和被操作的项即可

```
;一种square-list的定义
(define (square-list1 items)
  (if (null? items)
      '()
      (cons (square (car items))
            (square-list1 (cdr items)))))
```

```
;另一种square-list的定义
(define (square-list2 items)
  (map square items))

```

### EX 2.22

```
;原始程序(未交换cons参数)
(define (square-list items)
  (define (iter things answer)
    (if (null? things)
        answer
        (iter (cdr things)
              (cons (square (car things))
                    answer))))
  (iter items '()))
```

分析如下：

    (square-list (list 1 2 3))					->
	(iter (1 2 3) '())							->
	(iter (2 3) (cons 1 '()))					->
	(iter (3) (cons 4 (cons 1 '())))			->
	(iter '() (cons 9 (cons 4 (cons 1 '()))))	->	(cons 9 (cons 4 (cons 1 '())))) = (list 9 4 1)
	(9 4 1)

	
```
;原始程序(交换cons参数)
(define (square-list items)
  (define (iter things answer)
    (if (null? things)
        answer
        (iter (cdr things)
              (cons answer
                    (square (car things))))))
  (iter items '()))
```

分析如下：

    (square-list (list 1 2 3))					->
	(iter (1 2 3) '()) 							->
	(iter (2 3) (cons '() 1))					->
	(iter (3) (cons (cons '() 1) 4))			->
	(iter '() (cons (cons (cons '() 1) 4) 3))	->	(cons (cons (cons '() 1) 4) 3) = (((() . 1) . 4) . 9)
	(((() . 1) . 4) . 9)

	
```
;正确程序
(define (square-list items)
  (define (iter things answer)
    (if (null? things)
        (reverse answer)
        (iter (cdr things)
              (cons (square (car things))
                    answer))))
  (iter items '()))
```

分析如下：

    (square-list (list 1 2 3))					->
	(iter (1 2 3) '())							->
	(iter (2 3) (cons 1 '()))					->
	(iter (3) (cons 4 (cons 1 '())))			->
	(iter '() (cons 9 (cons 4 (cons 1 '()))))	->	(cons 9 (cons 4 (cons 1 '()))) = (list 9 4 1)
	(list 1 4 9)

### EX 2.23

```
;类似于map的定义，返回逻辑值
(define (for-each proc items)
  (if (null? items)
      #f
      (cons (proc (car items))
            (for-each proc (cdr items)))))
```