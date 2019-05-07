---
title: Java集合框架学习(5)——Collection接口和Iterable接口
date: 2017-03-12 15:05:58
tags: "Java"
categories: "Java"
---
***
学习各种集合容器最好先从基本接口入手，之前在[《Java集合框架学习(1)——概览》](http://mrcame.github.io/2017/02/10/Java%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%E5%AD%A6%E4%B9%A01/)中已经梳理清楚了各个类和接口的脉络，现在就从最基本的Collection来看一下：
```
public interface Collection<E> extends Iterable<E> {
	
    int size();  // 返回集合中元素数量，超过最大数量后返回Integer.MAX_VALUE

    boolean isEmpty();  // 集合是否为空

    boolean contains(Object o);  // 集合是否包含特定元素
    
    Iterator<E> iterator();  // 返回该集合的迭代器

    Object[] toArray();  // 返回一个包含集合中所有元素的数组

    // 泛型方法用类型变量声明，在方法的修饰符后，返回类型的前面
    <T> T[] toArray(T[] a);  // 同toArray()
    
    boolean add(E e);  // 向集合中添加元素，返回是否添加成功
    
    boolean remove(Object o);  // 删除集合中的特定元素，返回是否删除成功

    boolean containsAll(Collection<?> c);  // 是否包含特定集合中的全部元素(是否为子集)

    boolean addAll(Collection<? extends E> c);  // 是否成功添加特定集合中的全部元素

    boolean removeAll(Collection<?> c);  // 是否删除特定集合中的全部元素

    boolean retainAll(Collection<?> c);  // 是否包含特定集合中的部分元素(是否有交集)

    void clear();  // 删除集合中所有元素

    // ...省略了几个default方法和Object方法
}
```
这样来看基本的集合操作就一目了然，需要注意的是toArray方法，有两个重构的方法，下面来看一些细节：
```
    List<Integer> list = new ArrayList<Integer>();   
    list.add(1);  
    Integer[] i1 = (Integer[]) list.toArray();  // 造型异常
    Integer[] i2 = (Integer[]) list.toArray(new Integer[0]);  // Ok
```
运行时会报java.lang.ClassCastException。看了一下ArrayList中toArray的实现，在不带参数的toArray方法里：
```
    // 不带参数的toArray
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    // 带参数的toArray
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }

    // Arrays.copyOf
    public static <T> T[] copyOf(T[] original, int newLength) {
        return (T[]) copyOf(original, newLength, original.getClass());
    }

    // copyOf
    public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        T[] copy = ((Object)newType == (Object)Object[].class)  // 主要是这里
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```
采用无参的toArray，在三目运算中直接返回Object数组，此时进行转型是不安全的下溯造型(Downcasting)，会产生ClassCastException，这也就是上述问题的原因了。
采用有参的toArray，newType.getCommponentType()返回数组内容的类型，根据该类型构造对应类型的数组copy，于是不会有问题。
Collection继承了Iterable接口，接下来再看一下这个接口：
```
package java.lang;
public interface Iterable<T> {
    
    Iterator<T> iterator();  // 返回一个迭代器

    // ...省略了几个default方法
}
```
文档中是这样说的，只要实现了这个接口，就可以使用for-each循环来遍历目标对象。需要注意的是，Iterable这个接口比较特殊，不在java.util中。

下一篇来学习一下继承了Collection接口的三个基本接口——List、Set、Queue。