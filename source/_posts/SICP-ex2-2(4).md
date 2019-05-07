---
title: SICP 2.2节习题(4)
date: 2016-05-03 18:36:52
tags: "SICP解题集"
categories: "SICP解题集"
---

# 第二章 构造数据抽象

## 2.2 层次性数据和闭包性质

书中P82页，嵌套映射部分

```
;二元组
(define (flatmap proc seq)
  (accumulate append '() (map proc seq)))
(define (prime-sum? pair)
  (prime? (+ (car pair) (cadr pair))))
(define (make-pair-sum pair)
  (list (car pair) (cadr pair) (+ (car pair) (cadr pair))))

(define (prime-sum-pairs n)
  (map make-pair-sum
       (filter prime-sum?
               (flatmap
                (lambda (i)
                  (map (lambda (j) (list i j))
                       (enumerate-interval 1 (- i 1))))
                (enumerate-interval 1 n)))))
```

P33页判断是否为素数

```
;P33 prime?
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

(define (permutations s)
  (if (null? s)
      (list '())
      (flatmap (lambda (x)
                 (map (lambda (p) (cons x p))
                      (permutations (remove x s))))
               s)))
(define (remove item seq)
  (filter (lambda (x) (not (= x item)))
          seq))
```

### EX 2.40

```
(define (unique-pairs n)
  (flatmap
   (lambda (i)
     (map (lambda (j) (list i j))
          (enumerate-interval 1 (- i 1))))
   (enumerate-interval 1 n)))
(define (new-prime-sum-pairs n)
  (map make-pair-sum
       (filter prime-sum? (unique-pairs n))))
```

### EX 2.41

```
(define (sum-equal-s? triple s)
  (= s (+ (car triple) (cadr triple) (caddr triple))))
(define (make-triple n)
  (flatmap
   (lambda (i)
     (map (lambda (j) (cons i j))
          (unique-pairs (- i 1))))
   (enumerate-interval 1 n)))
(define (remove-not-equal triple s)
  (filter (lambda (pre-triple)
            (sum-equal-s? pre-triple s))
          triple))
(define (2-41 n s)
  (remove-not-equal (make-triple n) s))
```

### EX 2.42

```
(define (queens board-size)
  (define (queen-cols k)  ;第k列
    (if (= k 0)
        (list empty-board)
        (filter
         (lambda (positions) (safe? k positions))
         (flatmap
          (lambda (rest-of-queens)
            (map (lambda (new-row)
                   (adjoin-position new-row k rest-of-queens))
                 (enumerate-interval 1 board-size)))
          (queen-cols (- k 1))))))
  (queen-cols board-size))

(define empty-board '())
(define (adjoin-position new-row k rest)
  (cons new-row rest))
(define (safe? k position)
    (iter-check (car position) 
                (cdr position)
                 1))

(define (iter-check row-of-new-queen rest-of-queens i)
    (if (null? rest-of-queens)  ; 下方所有皇后检查完毕,新皇后安全
        #t
        (let ((row-of-current-queen (car rest-of-queens)))
            (if (or (= row-of-new-queen row-of-current-queen)           ; 行碰撞
                    (= row-of-new-queen (+ i row-of-current-queen))     ; 右下方碰撞
                    (= row-of-new-queen (- row-of-current-queen i)))    ; 左下方碰撞
                #f
                (iter-check row-of-new-queen 
                            (cdr rest-of-queens)    ; 继续检查剩余的皇后
                            (+ i 1))))))            ; 更新步进值
```

```
;测试程序
(define testQueens
  (for-each (lambda (pos)
                    (begin
                        (display pos)
                        (newline)))
                (queens 8)))
```