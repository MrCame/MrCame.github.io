---
title: Java集合框架学习(4)——ArrayList中的modCount变量
date: 2017-02-27 23:56:39
tags: "Java"
categories: "Java"
---
***
在上一篇学习了一些ArrayList基本的API及其实现后，看一些细节性的东西，其中这个modCount变量值得注意。
```
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {

    // ...

    protected transient int modCount = 0;
	
    // ...
}

```
modCount是定义在AbstractList中的成员变量
+ **protected**，说明该变量只可以在同包中的任何类、不同包中的任何当前类的子类中所访问。即不同包中的任何不是该类的子类不可访问
+ **transient**，说明串行化时被忽略

在ArrayList中与add和remove相关的操作，都会对该变量进行修改，例如remove方法中用到的fastRemove
```
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```

那么这个变量的作用是什么呢？ArrayList的内部类Itr实现了迭代器，其中定义了成员变量expectedModCount，在checkForComodification方法中对expectedModCount是否与ModCount相同进行了检查。
```
    private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```

可想而知，在使用迭代器遍历ArrayList时，如果使用ArrayList的add、remove方法，导致modCount改变，迭代器在检查时发现与期待的expectModCount不同，会抛出ConcurrentModificationException异常，如下例所示：
```
	public class Test {
	    public static void main(String[] args)  {
	        ArrayList<Integer> list = new ArrayList<Integer>();
	        list.add(2);
	        Iterator<Integer> iterator = list.iterator();
	        while(iterator.hasNext()){
	            Integer integer = iterator.next(); // 检查抛出异常
	            if(integer==2)
	                list.remove(integer); 
	        }
	    }
	}
```

那么如果在迭代过程中只是将ArrayList的remove方法换成在迭代器中定义的remove方法就对了吗？
```
	public class Test {
	    public static void main(String[] args)  {
	        ArrayList<Integer> list = new ArrayList<Integer>();
	        list.add(2);
	        Iterator<Integer> iterator = list.iterator();
	        while(iterator.hasNext()){
	            Integer integer = iterator.next();
	            if(integer==2)
	                iterator.remove();   //注意这个地方
	        }
	    }
	}
```

在单线程环境下确实如此，可当有多个线程对ArrayList进行操作时，线程1进行remove操作后改变了modCount值，但由于modCount不是volatile变量，线程2可能会看到原来的modCount，也可能看到新的modCount，当发现与本线程的expectModCount不同时，仍然会抛出异常。
即使换成Vector容器，可Vector也是继承自AbstractList，仍然会有问题。因此一般有2种解决办法：
+ 在使用iterator迭代的时候使用synchronized或者Lock进行同步；
+ 使用并发容器CopyOnWriteArrayList代替ArrayList和Vector。