---
title: Java集合框架学习(8)——Set接口和SortedSet接口
date: 2017-03-16 09:38:11
tags: "Java"
categories: "Java"
---
***
Set 是不包含重复元素的 Collection。更确切地讲，Set 不包含满足 e1.equals(e2) 的元素对 e1 和 e2，并且最多包含一个 null 元素。正如其名称所暗示的，此接口模仿了数学上的 set 抽象。声明的方法都是直接继承自父接口Collection的。
```
public interface Set<E> extends Collection<E> {
    
    int size();

    boolean isEmpty();

    boolean contains(Object o);

    Iterator<E> iterator();

    Object[] toArray();

    <T> T[] toArray(T[] a);

    boolean add(E e);

    boolean remove(Object o);

    boolean containsAll(Collection<?> c);

    boolean addAll(Collection<? extends E> c);

    boolean retainAll(Collection<?> c);

    boolean removeAll(Collection<?> c);

    void clear();

    boolean equals(Object o);

    int hashCode();

    @Override
    default Spliterator<E> spliterator() {  // ... }    
}
```

SortedSet 里面的元素一定是有序的。保证迭代器按照元素递增顺序遍历的集合，可以按照元素的自然顺序（参见 Comparable）进行排序，或者按照创建有序集合时提供的 Comparator进行排序。要采用此排序，还要提供一些其他操作。插入有序集合的所有元素都必须实现 Comparable 接口（或者被指定的 Comparator 所接受）。另外，所有这些元素都必须是可相互比较的，比如像 e1.compareTo(e2)
（或 comparator.compare(e1, e2)）对于有序集合中的任意元素 e1 和 e2 都不能抛出ClassCastException。试图违反这些限制将导致违反规则的方法或者构造方法调用抛出 ClassCastException。
注意，如果有序集合正确实现了 Set 接口，则有序集合所保持的顺序（无论是否明确提供了比较器）
都必须保持相等一致性。这也是因为 Set 接口是按照 equals 操作定义的，但有序集合使用它的 compareTo（或 compare）方法对所有元素进行比较，
因此从有序集合的观点来看，此方法认为相等的两个元素就是相等的。
即使顺序没有保持相等一致性，有序集合的行为仍然是 定义良好的，
只不过没有遵守 Set 接口的常规协定。
所有通用有序集合实现类都应该提供 4 个“标准”构造方法：
1) void（不带参数）构造方法，创建空的有序集合，按照元素的自然顺序排序。
2) 带有一个 Comparator 类型参数的构造方法，创建一个空的有序集合，根据指定的比较器排序。
3) 带有一个 Collection 类型参数的构造方法，创建一个元素与参数相同的有序集合，按照元素的自然顺序排序。
4) 带有一个 SortedSet 类型参数的构造方法，创建一个新的有序集合，元素及排序方法与输入的有序集合相同。
除了 JDK 实现（TreeSet 类）遵循此建议外，无法保证强制实施此建议（因为接口不能包含构造方法）。

声明的主要接口
<html><head><title></title></head><body><table id="pubmethods"><tbody><tr><th colspan="12">Public Methods</th></tr><tr><td><nobr>abstract&nbsp;<a>Comparator</a>&lt;?&nbsp;super&nbsp;E&gt;</nobr></td><td width="100%"><nobr><span>comparator</a></span>()</nobr><div>Returns the comparator used to compare elements in this&nbsp;<code>SortedSet</code>.</div><div>返回与此有序集合关联的比较器，如果使用元素的自然顺序，则返回null。</div></td></tr><tr><td><nobr>abstract E</nobr></td><td width="100%"><nobr><span>first</a></span>()</nobr><div>Returns the first element in this&nbsp;<code>SortedSet</code>.</div><div>返回此有序集合中当前第一个（最小的）元素。</div></td></tr><tr><td><nobr>abstract&nbsp;<a>SortedSet</a>&lt;E&gt;</nobr></td><td width="100%"><nobr><span>headSet</a></span>(E end)</nobr><div>Returns a&nbsp;<code>SortedSet</code>&nbsp;which contains elements less than the end element.</div><div>用一个SortedSet,返回此有序集合中小于end的所有元素。</div></td></tr><tr><td><nobr>abstract E</nobr></td><td width="100%"><nobr><span>last</a></span>()</nobr><div>Returns the last element in this&nbsp;<code>SortedSet</code>.</div><div>返回此有序集合中最后一个（最大的）元素</div></td></tr><tr><td><nobr>abstract&nbsp;<a>SortedSet</a>&lt;E&gt;</nobr></td><td width="100%"><nobr><span>subSet</a></span>(E start,E end)</nobr><div>Returns a&nbsp;<code>SortedSet</code>&nbsp;which contains elements greater or equal to the start element but less than the end element.</div><div>返回此有序集合的部分元素，元素范围从fromElement（包括）到toElement（不包括）。</div></td></tr><tr><td><nobr>abstract&nbsp;<a>SortedSet</a>&lt;E&gt;</nobr></td><td width="100%"><nobr><span>tailSet</a></span>(E start)</nobr><div>Returns a&nbsp;<code>SortedSet</code>&nbsp;which contains elements greater or equal to the start element.</div><div>返回此有序集合的部分元素，其元素大于或等于fromElement。</div></td></tr></tbody></table></body></html>