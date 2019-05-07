---
title: Java集合框架学习(6)——List接口
date: 2017-03-13 17:25:30
tags: "Java"
categories: "Java"
---
***
接着上一篇的内容，看过了Collection以后，再看一下继承了它的三个子接口List、Set、Queue
几种常用的数据结构如ArrayList，LinkedList等，都实现了List接口
```
public interface List<E> extends Collection<E> {
    
    // ... 省略已经在父接口中声明的方法

    boolean addAll(int index, Collection<? extends E> c);  // 在List的特定位置插入某集合的全部元素
    
    default void replaceAll(UnaryOperator<E> operator)  // 替换列表中的每个数据

    default void sort(Comparator<? super E> c)  // 使用比较器对列表排序

    E get(int index);  // 得到列表指定位置的数据

    E set(int index, E element);  // 替换列表中指定位置的数据

    void add(int index, E element);  // 在列表的指定位置插入数据

    E remove(int index);  // 删除列表中指定位置的数据或第一次出现的指定数据

    int indexOf(Object o);  // 得到指定数据在列表中第一次出现的位置，如果列表中不包含指定数据，返回-1

    int lastIndexOf(Object o);  // 返回指定数据在列表中最后一次出现的位置

    ListIterator<E> listIterator();  // 返回列表的迭代器

    ListIterator<E> listIterator(int index);  // 返回列表指定位置的迭代器

    List<E> subList(int fromIndex, int toIndex);  // 返回列表两个位置之间的数据组成的列表

    @Override
    default Spliterator<E> spliterator()  // 创建一个列表的分片迭代器
}
```
首先是一个接口继承的问题，List接口中声明了很多Collection接口中已经声明过的方法，例如size，isEmpty等，没有用@Override做注解，在这里我想为什么已经在Collection中已经声明了size方法还要在子接口List中再次声明呢？
我的理解是，采用这样的设计方法，将层次明确，比如一个List的实现需要size方法时指的就是这个List接口的size，后面的实现类中，明确的实现了List接口中定义的size，而非Collection接口中的size。
其次是几个在JDK1.8中加入的几个default方法：
```
    List<Integer> list = new ArrayList<>();
    list.add(1);
    list.add(2);
    
    // 将每个元素加1 
    list.replaceAll(new UnaryOperator<Integer>() {
        @Override
        public Integer apply(Integer integer) {
            return integer + 1;
        }
    });

    // 使用定义的比较器对list进行排序
    list.sort(new Comparator<String>() {
        public int compare(String  x, String y) {
            return x.compareTo(y);
        }
    });
```
Spliterator方法要细说就不是这个系列所要讲述的内容了。Spliterator（splitable iterator可分割迭代器）接口是Java为了并行遍历数据源中的元素而设计的迭代器，这个可以类比最早Java提供的顺序遍历迭代器Iterator，但一个是顺序遍历，一个是并行遍历。从最早Java提供顺序遍历迭代器Iterator时，那个时候还是单核时代，但现在多核时代下，顺序遍历已经不能满足需求了，如何把多个任务分配到不同核上并行执行，才是能最大发挥多核的能力，于是有了Spliterator。以后再开一篇JDK8的讲解系列。