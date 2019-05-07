---
title: Java集合框架学习(7)——Queue接口和Deque接口
date: 2017-03-15 00:24:13
tags: "Java"
categories: "Java"
---
***
上一篇已经讲解了List接口的一些内容，现在继续看一下Collection的另一个子接口Queue，以及Queue的子接口Deque。
```
public interface Queue<E> extends Collection<E> {
    
    // 继承自Collection
    boolean add(E e);  // 在队列中插入特定元素
    
    boolean offer(E e);  // 在队列中插入特定元素
    
    E remove();  // 删除并取得队列头元素
    
    E poll();  // 删除并取得队列头元素
    
    E element();  // 从头部取得元素但不删除
    
    E peek();  // 从头部取得元素但不删除
}
```
声明的这些方法中，只有Collection是继承自Collection接口的。
+ add与offer的不同之处在于：在容量受限的队列中，offer方法比add方法更好，因为add在插入失败时仅仅抛出异常
+ remove与poll的不同之处在于：如果队列为空，remove将抛出异常，poll返回null
+ element与peek的不同之处在于：如果队列为空，element将抛出异常，peek返回null

Deque是双端队列(double ended queue)的缩写，大部分Deque用于不固定元素数量的情形，当然也支持容量受限的情形。和Queue接口一样，Deque的插入、删除以及获取元素的操作都有两种形式，当操作失败时，前一种形式抛出异常，后一种形式返回特殊值(null或者false)。后一种形式的插入操作用于容量受限的Deque实现，而在大多数情况中，插入操作不会失败。

<table><caption>Deque方法总结</caption><tbody><tr><td></td><td align="CENTER" colspan="2"> <b>首元素 (头)</b></td><td align="CENTER" colspan="2"> <b>尾元素 (尾)</b></td></tr> <tr> <td></td> <td align="CENTER"><em>抛出异常</em></td><td align="CENTER"><em>特殊值</em></td><td align="CENTER"><em>抛出异常</em></td><td align="CENTER"><em>特殊值</em></td></tr><tr><td><b>插入</b></td><td>void addFirst(E e)</td><td>boolean offerFirst(E e)</td><td>void addLast(E e)</td><td>boolean offerLast(E e)</td></tr><tr><td><b>删除</b></td><td>E removeFirst()</td><td>E pollFirst()</td><td>E removeLast()</td><td>E pollLast()</td></tr><tr><td><b>获取</b></td><td>E getFirst()</td><td>E peekFirst()</td><td>E getLast()</td><td>E peekLast()</td></tr></tbody></table>

Deque用做队列时，表现为FIFO(先入先出)，从队尾进入，从队首删除，与Queue中的一些方法等价：

<table><caption>Queue和Deque方法比较</caption><tr><td ALIGN=CENTER><b>Queue方法</b></td><td ALIGN=CENTER><b>等价的Deque方法</b></td></tr><tr><td>boolean add(E e)</td><td>void addLast(E e)</td></tr><tr><td>boolean offer(E e)</td><td>boolean offerLast(E e)</td></tr><tr><td>E remove()</td><td>E removeFirst()</td></tr><tr><td>E poll()</td><td>E pollFirst()</td></tr><tr><td>E element()</td><td>E getFirst()</td></tr><tr><td>E peek()</td><td>E peekFirst()</td></tr></table>

用做栈时，表现为LIFO(后入先出)，入栈和出栈都是从队首，与Stack中的一些方法等价：
<table><caption>Deque和Stack方法比较</caption><tr><td align="center"><b>Stack方法</b></td><td align="center"><b>等价的Deque方法</b></td></tr><tr><td>E push(E e)</td><td>void addFirst(E e)</td></tr><tr><td>E pop()</td><td>E removeFirst()</td></tr><tr><td>E peek()</td><td>E peekFirst()</td></tr></table>

```
public interface Deque<E> extends Queue<E> {

    // Deque方法总结中提到的12个方法
    
    boolean removeFirstOccurrence(Object o);  // 删除第一个出现的特定元素，没有该元素不发生改变

    boolean removeLastOccurrence(Object o);  // 删除最后一个出现的特定元素，没有该元素不发生改变

    // 继承自Queue的6个方法

    // *** Stack methods ***
    void push(E e);  // 入栈

    E pop();  // 出栈

    // *** Collection methods ***
    // 继承自Collection的remove、contain、size、iterator方法

    Iterator<E> descendingIterator();  // 返回反向迭代器，和iterator相反，实现从两端访问元素
}
```

Deque接口提供了两个删除内部元素的方法，removeFirstOccurrence、removeLastOccurrence，可以避免只能从首或尾操作的麻烦。与List不同，Deque不支持通过索引的方式访问元素。