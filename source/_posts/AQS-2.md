---
title: AQS源码解析（2）
date: 2019-05-13 17:55:22
tags: "Java"
categories: "Java"
---

上一篇 [AQS源码解析(1)](http://mrcame.github.io/2019/05/10/AQS-1/)已经介绍了AQS的数据结构。在这篇，将从J.U.C里的许多基于AQS实现的同步工具出发，看一看AQS到底提供了哪些功能。
比如`Lock lock = new ReentrantLock()`，默认是非公平锁的实现，使用`lock()`、`unlock()`进行同步，先来看一下如何加锁。

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    private final Sync sync;
  
    public ReentrantLock() {
      sync = new NonfairSync(); 
    }
  
    public ReentrantLock(boolean fair) { 
      sync = fair ? new FairSync() : new NonfairSync(); 
    }
  
    public void lock() { 
      sync.lock(); 
    }

    abstract static class Sync extends AbstractQueuedSynchronizer { 
      abstract void lock(); 
    }
    
    static final class FairSync extends Sync {
        final void lock() {
            acquire(1);
        }
      
        protected final boolean tryAcquire(int acquires) {}  // 见下文
    }
    
    static final class NonfairSync extends Sync {
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }
      
        protected final boolean tryAcquire(int acquires) {}  // 见下文
    }       
}
```

看代码还是很清晰的，核心就在`acquire()`中。这个方法是在父类AQS内定义实现的，而`tryAcquire()`在AQS内只给了定义，具体实现还是在子类中。

```java
/** AbstractQueuedSynchronizer.java **/
    public final void acquire(int arg) {
        // tryAcquire失败，将独占模式节点入队，再从队列中获取
        if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))  
            selfInterrupt();
    }

    protected boolean tryAcquire(int arg) {
        throw new UnsupportedOperationException();
    }
```

以`ReentrantLock`为例，内部类`Sync`的子类`FairSync`和`NonFairSync`分别实现了公平锁与非公平锁。这时能清楚看到，在AQS中用来表示同步状态的`state`，会随着每次`acquire()`而从`0`开始累加。同理每次`release()`会减少直到为`0`，当`state == 0`可以认为资源都被释放，公平锁需要把资源让给等待更久的线程。如果当前线程已经持有资源，可以继续增加`state`，无需重新等待竞争，这就是可重入的意义。

```java
/** ReentrantLock#FairSync.java 公平锁tryAcquire实现 **/
static final class FairSync extends Sync {
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();  // volatile int state
        if (c == 0) {
            // 公平锁：需要先看看是不是有线程比当前线程等的更久，没有的话再acquire资源
            if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);  // 设置当前线程独占
                return true;
            }
        }
        // 如果当前线程独占，无需等待再次进入
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}

/** ReentrantLock#NonfairSync.java 非公平锁tryAcquire实现 **/
static final class NonfairSync extends Sync {
    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);  // 父类Sync中的nonfairTryAcquire方法
    }
}

abstract static class Sync extends AbstractQueuedSynchronizer {
    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            // 非公平锁：不用hasQueuedPredecessors，直接acquire资源
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);  // 设置当前线程独占
                return true;
            }
        }
        // 如果当前线程独占，无需等待再次进入
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```

如果`tryAcquire(arg)`失败，将创建独占模式节点入队，`addWaiter(Node.EXCLUSIVE)`

```java
/** AbstractQueuedSynchronizer.java **/    
    private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);  // 节点与当前线程关联，并设置模式
        // 尝试快速方式入队
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            // 如果多个线程尝试入队，操作必须是原子的，将尾节点设置为node
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;  // 返回新节点
            }
        }
        // 快速方式失败，自旋入队
        enq(node);
        return node;  // 返回先前节点
    }

    private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;  // 返回先前节点
                }
            }
        }
    }
```

进行到此，再回顾一下：`!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg)`
`acquireQueued()`将开始从队列中自旋获取刚加入的节点，主要做的就是三件事，一是将满足条件的节点设置为新的头节点，二是给不满足条件的节点调整到最近的一个非`CANCELLED`节点后，并且将其先前节点的状态修改为`SIGNAL`，三是使用`LockSupport.park`将合适节点的线程休眠，最终返回结果表示当线程等待过程中是否被中断。

```java
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                // 当前节点满足条件：是第二个节点，并且acquire成功
                if (p == head && tryAcquire(arg)) {
                    // 设置为头节点，这里不用CAS，因为已经acquire到资源
                    setHead(node);  
                    p.next = null; // help GC，回收旧的头节点
                    failed = false;
                    return interrupted;
                }
                // 当前节点不满足条件，调整节点，如果可以的话，休眠线程
                if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
                    interrupted = true;  // 只要有一次中断，就将中断标志设置为true
            }
        } finally {  // return前，先检查failed，如果出现异常则取消该节点的acquire
            if (failed)
                cancelAcquire(node);
        }
    }

    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)  // 先前节点状态已经是SIGNAL，当前节点位置安全，可以休眠
            return true;
        if (ws > 0) {  
            // 先前节点状态是CANCELLED，将当前节点调整到最近的一个非CANCELLED节点后
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            // 先前节点状态不是CANCELLED，CAS为SIGNAL
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        // 此时只是将先前节点状态修改，但考虑到并发情况，也有其他线程在做同样的事情，需要再次自旋检查
        return false;
    }

    private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);  // 阻塞线程，被unpark或被interrupt可以唤醒
        return Thread.interrupted();  // 返回线程是否被中断过
    }

    private void cancelAcquire(Node node) {
        if (node == null)
            return;
        node.thread = null;
        // 跳过CANCELLED状态的先前节点，调整该节点pred域
        Node pred = node.prev;
        while (pred.waitStatus > 0)
            node.prev = pred = pred.prev;
        Node predNext = pred.next;  // 先前节点的后续节点

        node.waitStatus = Node.CANCELLED;
      
        // 如果该节点是队尾，将先前节点设置为队尾，然后将其next域置空
        if (node == tail && compareAndSetTail(node, pred)) {
            compareAndSetNext(pred, predNext, null);
        } else {  
            // 不是队尾
            // 先前节点同时如下满足条件：不是头节点、状态是SIGNAL或者可以变为SIGNAL、绑定了线程
            // 将其pred节点的next域置为该节点的后续节点
            int ws;
            if (pred != head &&
                ((ws = pred.waitStatus) == Node.SIGNAL ||
                 (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
                pred.thread != null) {
                Node next = node.next;
                if (next != null && next.waitStatus <= 0)
                    compareAndSetNext(pred, predNext, next);
            } else {
                unparkSuccessor(node);  // 唤醒后续节点
            }

            node.next = node; // help GC，使CANCELLED状态的节点不可达，被GC回收
        }
    }
```

在看源码时有一个问题困扰着我，为什么在`shouldParkAfterFailedAcquire`时一定是从尾向头去找，而不是反过来呢？
至此，对于`ReentrantLock`的`lock()`的实现，还剩一步`unparkSuccessor()`来唤醒后续节点，这个留在接下来要学习的`unlock()`实现中来分析。

