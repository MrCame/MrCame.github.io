---
title: AQS源码解析（1）
date: 2019-05-10 17:05:52
tags: "Java"
categories: "Java"
---

CLH（Craig, Landin, and Hagersten）锁常用于实现自旋锁，AQS使用一种变形后的CLH锁队列，来代替阻塞同步。

```
      +------+  prev +-----+       +-----+
 head |      | <---- |     | <---- |     |  tail
      +------+       +-----+       +-----+
```

AQS使用双向队列，每一个Node都代表一个线程，有status属性，标记着线程是否应该被阻塞，prev指针指向前个节点，next指针（图中没有画出）指向后续节点，head和tail不言而喻

```java
/** AbstractQueuedSynchronizer#Node.java **/
static final class Node {
    // 处于共享模式
    static final Node SHARED = new Node();

    // 处于独占模式
    static final Node EXCLUSIVE = null;

    // 表示线程取消。（因为中断或者超时，不会再参与到竞争中，状态也不再改变）
    static final int CANCELLED =  1;
  
    // 表示后续线程等待释放。（因为后续节点被阻塞，处于等待状态，需要当前节点释放或者取消，之后会通知后续节点）
    static final int SIGNAL    = -1;
  
    // 表示线程正在等待条件。（线程正处于Condition队列中等待条件，直到状态转变为0，才会被用于同步队列中）
    static final int CONDITION = -2;
  
    // 表示下一次共享式acquire将无条件传播。(共享式release会传播到其他节点，在doReleaseShared方法中)
    static final int PROPAGATE = -3;
    
    // 等待状态，初始化为0，非负数表示节点无需被通知
    volatile int waitStatus;

    // 注意：该节点的先前节点如果是CANCELLED的，就往前找第一个非CANCELLED的，肯定能找到一个，因为至少头节点就不是CANCELLED。
    volatile Node prev;

    // 注意：只有该节点成为尾节点时才会给先前节点的next域赋值
    volatile Node next;

    // 节点绑定线程
    volatile Thread thread;

    // 等待conditon的下一个节点（独占模式），或者SHARED(共享模式)
    Node nextWaiter;

    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }

    Node() {}

    Node(Thread thread, Node mode) {
        // addWaiter方法使用
        this.nextWaiter = mode;
        this.thread = thread;
    }

    Node(Thread thread, int waitStatus) { 
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```

接下来再看一下AQS定义的域，特别要注意这里的`state`，表示同步状态，而不是上文中的线程等待状态`waitStatus`。

```java
    // 头节点，通过setHead方法修改，等待状态不能是CANCELLED
    private transient volatile Node head;

    // 尾节点，入队时增加
    private transient volatile Node tail;

    // 同步状态
    private volatile int state;

    static final long spinForTimeoutThreshold = 1000L;

    protected final int getState() {
        return state;
    }

    protected final void setState(int newState) {
        state = newState;
    }
```

AQS的一些包装UnSafe的CAS操作

```java
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long stateOffset;
    private static final long waitStatusOffset;
    // ...省略 headOffset、tailOffset、nextOffset

    static {
        try {
            stateOffset = unsafe.objectFieldOffset
                (AbstractQueuedSynchronizer.class.getDeclaredField("state"));
            waitStatusOffset = unsafe.objectFieldOffset
                (Node.class.getDeclaredField("waitStatus"));
            // ...省略 headOffset、tailOffset、nextOffset
          
        } catch (Exception ex) { throw new Error(ex); }
    }

    // CAS同步状态
    protected final boolean compareAndSetState(int expect, int update) {
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }
 
    // 入队时如果队空CAS头节点
    private final boolean compareAndSetHead(Node update) {
        return unsafe.compareAndSwapObject(this, headOffset, null, update);
    }

    // 入队时CAS尾节点
    private final boolean compareAndSetTail(Node expect, Node update) {
        return unsafe.compareAndSwapObject(this, tailOffset, expect, update);
    }

    // CAS节点等待状态
    private static final boolean compareAndSetWaitStatus(Node node,
                                                         int expect,
                                                         int update) {
        return unsafe.compareAndSwapInt(node, waitStatusOffset,
                                        expect, update);
    }

    // CAS节点next域
    private static final boolean compareAndSetNext(Node node,
                                                   Node expect,
                                                   Node update) {
        return unsafe.compareAndSwapObject(node, nextOffset, expect, update);
    }
```

【题外话】补充一下，因为UnSafe做了安全验证，只允许信任的JDK调用，如果使用如上所示的`Unsafe.getUnsafe()`或者直接实例化，那么会抛`Caused by: java.lang.SecurityException: Unsafe`的异常，可以通过反射其内部的`theUnsafe`的域来进行实例化。

```java
    Field f = Unsafe.class.getDeclaredField("theUnsafe"); //Internal reference
    f.setAccessible(true);
    Unsafe unsafe = (Unsafe) f.get(null);
```

