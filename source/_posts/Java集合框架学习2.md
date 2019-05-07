---
title: Java集合框架学习(2)——Iterator
date: 2017-02-11 13:20:36
tags: "Java"
categories: "Java"
---
***
### Iterator
**default**修饰符是Java8中新的概念，在Java8发布时，需要Java在现有实现架构的下能往接口里增加新方法。引入Default方法，可以在优化接口的同时，避免跟现有实现架构的兼容问题。
```
package java.util;

import java.util.function.Consumer;

public interface Iterator<E> {

    // 是否有下一元素
    boolean hasNext();

    // 返回下一元素
    E next();

	// Default方法是指，在接口内部包含了一些默认的方法实现
	// 也就是接口中可以包含方法体，这打破了Java之前版本对接口的语法限制
    // 从而使得接口在进行扩展的时候，不会破坏与接口相关的实现类代码。
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }

    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
}

```
***
### ListIterator
ListIterator继承自Iterator，除了父接口原有的方法**hasNext()**、**next()**外，还申明了新的查询操作**hasPrevious()**、**previous()**、**nextIndex()**、**previousIndex()**，以及修改操作**remove()**、**set()**、**add()**。相关操作见下图
!["ListIterator"](/images/java-3-0.png)
```
package java.util;
public interface ListIterator<E> extends Iterator<E> {
    // Query Operations

    // 继承自父类
    boolean hasNext();

    // 继承自父类
    E next();

    // 是否有前一元素
    boolean hasPrevious();

    // 返回前一元素
    E previous();

    // 返回当前iterator的下标
    int nextIndex();

    // 返回当前iterator的前一下标，如果当前下标为0，前一下标为-1
    int previousIndex();

    // Modification Operations

    // 删除元素
    // remove当前元素必须先next()，否则remove的是前一元素
    // 若iterator下标为0，直接remove抛出IllegalStateException异常
    void remove();

    // 替换元素
    // 同remove，set当前元素前必须先next，否则set的是前一元素
    // 若iterator下标为0，直接set抛出IllegalStateException异常
    void set(E e);

    // 插入元素，在该iterator的位置add
    void add(E e);
}
```
