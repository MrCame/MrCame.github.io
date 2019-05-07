---
title: Java集合框架学习(3)——ArrayList
date: 2017-02-22 09:51:24
tags: "Java"
categories: "Java"
---
***
ArrayList是Java集合框架里很常用的一个数据结构，底层用数组实现数据存储，所以基本上都是对数据的操作。
!["ArrayList"](/images/java-4-0.png)
### 一些成员变量
```
    // 默认数组容量
    private static final int DEFAULT_CAPACITY = 10;

    // 底层数组
    transient Object[] elementData;

    // ArrayList包含的元素数量
    private int size;
```
其中，transient是Java的关键字。Java的serialization提供了一种持久化对象实例的机制。当持久化对象时，可能有一个特殊的对象数据成员，我们不想用serialization机制来保存它。为了在一个特定对象的一个域上关闭serialization，可以在这个域前加上关键字transient，用来表示一个域不是该对象串行化的一部分。当一个对象被串行化的时候，transient型变量的值不包括在串行化的表示中，然而非transient型的变量是被包括进去的。  

### 构造函数
ArrayList提供了三个构造函数
+ ArrayList(int initialCapacity)：构造一个具有指定初始容量的空列表。
+ ArrayList()：默认构造函数,默认为空，DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {}。
+ ArrayList(Collection<? extends E> c)：构造一个包含指定 collection 的元素的列表，这些元素是按照该 collection 的迭代器返回它们的顺序排列的。
```
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```
在声明一个ArrayList时
+ **ArrayList alist = new ArrayList()**  用到了ArrayList的特性
+ **List list = new ArrayList()**  通用性更强，可直接替换为其他List接口的实现
### 主要API
+ size()，元素数量，复杂度O(1)
```
    public int size() {
        return size;
    }
```
+ isEmpty()，判断是否为空，复杂度O(1)
```
    public boolean isEmpty() {
        return size == 0;
    }
```
+ contains()，判断是否包括特定元素，复杂度O(n)
```
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }
```
主要是indexOf(Object o)方法，实现了O(n)复杂度的线性查找，遍历数组，若找到则返回索引i，最坏情况遍历到最后才找到。因为ArrayList是顺序存储，如果知道存储顺序的话，可以选择用lastIndexOf(Object o)倒序遍历。
```
	public int indexOf(Object o) {
		if (o == null) {
			for (int i = 0; i < size; i++)
				if (elementData[i] == null)
					return i;
		} else {
			for (int i = 0; i < size; i++)
				if (o.equals(elementData[i]))
					return i;
		}
		return -1;
	}
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```
+ get(int index)，得到指定位置的元素，复杂度O(1)
```
    public E get(int index) {
        rangeCheck(index); // 检查索引越界情况

        return elementData(index);
    }
```
+ set(int index, E element)，替换指定位置的元素，复杂度O(1)
```
    public E set(int index, E element) {
        rangeCheck(index); // 检查索引越界情况

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
```
+ add(E e)，在末尾添加元素，复杂度O(1)
ensureCapacityInternal()方法判断是否需要扩容，grow()负责扩容操作
```
	public boolean add(E e) {
	    ensureCapacityInternal(size + 1);  // Increments modCount!!
	    elementData[size++] = e;
	    return true;
	}
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        // 所需最小容量大于当前数组容量，扩容
        if (minCapacity - elementData.length > 0) 
            grow(minCapacity);
    }

    private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
		// 扩容为原来的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
		// 扩容后仍不够
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
!["grow"](/images/java-4-1.png)
+ add(int index, E element)，在指定位置添加元素，复杂度O(n)
需要将该位置后的所有元素向后移动一位，复杂度取决于数组规模
```
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        size++;
    }
```
+ remove(int index) 删除并取得指定位置的元素，复杂度O(n)
需要将该位置后的所有元素向前移动一位，复杂度取决于数组规模
```
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;  // 需要向前移动一位的元素长度
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // 清除该位置的引用，让GC起作用

        return oldValue;
    }
```
关于Java GC这里需要特别说明一下，有了垃圾收集器并不意味着一定不会有内存泄漏。对象能否被GC的依据是是否还有引用指向它，上面代码中如果不手动赋 ***null*** 值，除非对应的位置被其他元素覆盖，否则原来的对象就一直不会被回收。
+ remove(Object o) 删除一个元素，复杂度O(n)
需要遍历数组查找第一个满足条件的元素，然后把后面的元素向前前进一位
```
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
```
+ addAll()
一次性添加多个元素，根据位置不同也有两个把本，一个是在末尾添加的 **addAll(Collection<? extends E> c)** 方法，一个是从指定位置开始插入的 **addAll(int index, Collection<? extends E> c)** 方法。跟 **add()** 方法类似，在插入之前也需要进行空间检查，如果需要则自动扩容；如果从指定位置插入，也会存在移动元素的情况。 **addAll()** 的时间复杂度不仅跟插入元素的多少有关，也跟插入的位置相关。
+ removeAll(Collection<?> c)
删除指定collection中包含的所有元素，即求两个集合的差集，A-B。
+ retainAll(Collection<?> c)
保留指定collection中包含的所有元素，即求两个集合的交集，A∩B。