---
title: AQS源码解析（3）
date: 2019-05-16 17:24:24
tags: "Java"
categories: "Java"
---

上一篇[AQS源码解析(2)](http://mrcame.github.io/2019/05/13/AQS-2/)，学习了`ReentrantLock`的加锁过程，对照地来看一下如何释放锁。与加锁有一些不同的地方是，释放时直接就`release(1)`，这是在AQS中实现的，不像加锁时在`Sync`的子类中才实现。

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    private final Sync sync;
    
    public void unlock() {
        sync.release(1);
    }
    
    abstract static class Sync extends AbstractQueuedSynchronizer { 
        protected final boolean tryRelease(int releases) {}  // 见下文
    }
}
```

核心方法有两个`tryRelease()`和`unparkSuccessor()`，看过加锁实现，应该也能猜到，`tryRelease`应该是在子类中才实现的，果不其然，AQS里只是定义，但不像加锁那样分为公平/非公平各自实现。而后者从名字能猜到，是要唤醒链表中的后续节点。

```java
/** AbstractQueuedSynchronizer.java **/
    public final boolean release(int arg) {
        if (tryRelease(arg)) {  // 尝试释放资源
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);  // 唤醒头节点的后续节点
            return true;
        }
        return false;
    }
    
    protected boolean tryRelease(int arg) {
        throw new UnsupportedOperationException();
    }

/** ReentrantLock#Sync.java **/
abstract static class Sync extends AbstractQueuedSynchronizer { 
    protected final boolean tryRelease(int releases) {
        int c = getState() - releases;
        // 当前线程不是独占的，抛异常
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        if (c == 0) {  // 之前acquire(1)，现在releases(1)
            free = true;
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;         
    }
}
```

以`ReentrantLock`为例，`lock()`本质上就是`acquire(1)`的过程，在`unlock()`时只需释放掉这拿到的1个状态资源，接着分析上一篇中遗留的`unparkSuccessor()`方法。

```java
/** AbstractQueuedSynchronizer.java **/
    private void unparkSuccessor(Node node) {
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);  // 节点置为无状态

        // 找到后续节点，如果是null或者是CANCELLED状态的话，从尾节点向前找
        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);
    }
```

至此，可以看到`Reentrant`的整个`lock()`和`unlock()`过程，当节点`acquire`成功时，置为头节点，同时其他节点也在`acquire`,得不到竞态资源(state)就入队自旋等待，直到持有资源的头节点释放，并唤醒其后续节点。