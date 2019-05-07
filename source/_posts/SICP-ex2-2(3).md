---
title: SICP 2.2节习题(3)
date: 2016-04-28 20:08:11
tags: "SICP解题集"
categories: "SICP解题集"
---

# 第二章 构造数据抽象

## 2.2 层次性数据和闭包性质

### EX 2.33

accumulate中参数op是二元操作
lambda (x y) 中，x:(car seq) y:(accumulate op init (cdr seq))

```
;原始map定义
(define (map proc items)
  (if (null? items)
      '()
      (cons (proc (car items))
            (map proc (cdr items)))))
```

```
;用accumulate定义
(define (map1 p sequence)
  (accumulate (lambda (x y) (cons (p x) y)) '() sequence))
```

```
;原始append定义
(define (append list1 list2)
  (if (null? list1)
      list2
      (cons (car list1) (append (cdr list1) list2))))
```

```
;用accumulate定义
(define (append1 seq1 seq2)
  (accumulate cons seq2 seq1))
```

```
;原始length定义
(define (length items)
  (define (length-iter a count)
    (if (null? a)
        count
        (length-iter (cdr a) (+ 1 count)))))
```

```
;用accumulate定义
(define (length1 sequence)
  (accumulate (lambda (x y) (+ y 1)) 0 sequence))
```

### EX 2.34

主要是采用Horner规则将多项式变形

![“ex2-34”](/images/ex2-34.jpg)

```
(define (horner-eval x coefficient-sequence)
  (accumulate (lambda (this-coeff higher-terms) (+ this-coeff (* higher-terms x)))
              0
              coefficient-sequence))
```

### EX 2.35

```
;原始count-leaves定义
(define (count-leaves t)
  (cond ((null? t) 0)
        ((not (pair? t)) 1)
        (else (+ (count-leaves (car t))
                 (count-leaves (cdr t))))))
```

```
;用accumulate定义
;map计算每一个结点的叶子数量,得到如(0 0 1 1 ......)的list，再累积便可
(define (count-leaves1 t)
  (accumulate + 0 (map
                   (lambda (x)
                     (if (pair? x)
                         (count-leaves1 x)
                         1))
                     t)))
```

### EX 2.36

重点在于如何得到每一个内部序列的第一个元素，于是map一次seqs，对每个内部序列都得到其car

```
(define (accumulate-n op init seqs)
  (if (null? (car seqs))
      '()
      (cons (accumulate op init (map car seqs))
            (accumulate-n op init (map cdr seqs)))))
```

### EX 2.37

```
;测试程序 矩阵1、2 向量1、2
(define test-matrix1 (list (list 1 2 3 4) (list 4 5 6 6) (list 6 7 8 9)))
(define test-matrix2 (list (list 1 4 6) (list 2 5 7) (list 3 6 8) (list 4 6 9)))
(define test-vector1 (list 1 2 3 4))
(define test-vector2 (list 4 5 6 6))
```

```
;求点积
(define (dot-product v w)
  (accumulate + 0 (map * v w)))
```

```
;矩阵乘向量，(矩阵列数 = 向量行数) 用map对每一行做点积
(define (matrix-*-vector m v)
  (map (lambda (row)
         (dot-product row v))
         m))
```

```
;求转置矩阵
(define (transpose mat)
  (accumulate-n cons '() mat))
```

```
;矩阵相乘
(define (matrix-*-matrix m n)
  (let ((cols (transpose n)))
    (map (lambda (row)
           (map (lambda (col)
                  (dot-product row col))
                cols))
         m)))
```

### EX 2.38

书中fold-left过程如下

```
(define (fold-left op initial sequence)
  (define (iter result rest)
    (if (null? rest)
        result
        (iter (op result (car rest))
              (cdr rest))))
  (iter initial sequence))
```

从右向左对元素使用op操作，类似于accumulate

```
(define (fold-right op initial sequence)
  (if (null? sequence)
      initial
      (op (car sequence)
          (accumulate op initial (cdr sequence)))))
```

由几个例子可以看出，加法、乘法等满足结合律的op，对任何序列可以产生同样的结果

### EX 2.39

```
;原始reverse
(define (reverse x)
  (define (iter remain result)
    (if (null? remain)
        result
        (iter (cdr remain) (cons (car remain) result))))
  (iter x '()))
```

```
(define (reverse1 sequence)
  (fold-right (lambda (x y) (append y (list x))) '() sequence))
(define (reverse2 sequence)
  (fold-left (lambda (x y) (cons y x)) '() sequence))
```