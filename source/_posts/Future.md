---
title: Future
date: 2019-05-24 14:45:59
tags: "Java"
categories: "Java"
---

Callable提供了带返回值的子线程执行结果，Future提供了获取子线程结果的途径

```java
        Callable<String> callable = () -> null;
        ExecutorService executor = Executors.newSingleThreadExecutor();
        Future<String> future = executor.submit(callable);
        System.out.println(future.get());
```

但是，这种直接`get()`的方式是同步阻塞的，当然，如果轮询`isDone()`的话仍然是换汤不换药。关于老生常谈的同步、异步、阻塞、非阻塞，这篇[《I/O模型》](https://mp.weixin.qq.com/s/uDgueoMIEjl-HCE_fcSmSw)从Java的视角出发来讲解，特别是`NIO`和`AIO`，指出`AIO`并不是字面上的异步含义，值得一看。
那么对于`Future`模式，除了上文的将来式`get()`这种不优雅的同步阻塞方案，还有没有其他的方式可以拿到子线程结果呢？
很容易想到的一种方式是使用回调。如`AIO`提供了`java.nio.channels.CompletionHandler`作为回调接口，当I/O操作结束后，系统将会调用`CompletionHandler`的`completed`或`failed`方法来结束一次调用

```java
        /* Server */
        AsynchronousChannelGroup channelGroup = AsynchronousChannelGroup
                .withThreadPool(executor);
        AsynchronousServerSocketChannel serverChannel = AsynchronousServerSocketChannel
                .open(channelGroup)
                .bind(new InetSocketAddress(serverPort));
        serverChannel.accept(
                null,
                new CompletionHandler<AsynchronousSocketChannel, Object>() {
                    @Override
                    public void completed(AsynchronousSocketChannel result, Object attach) {
                        // doSomething()
                        serverChannel.accept(null, this);
                    }

                    @Override
                    public void failed(Throwable exc, Object attach) { }
                });

        /* Client */
        AsynchronousSocketChannel clientChannel = AsynchronousSocketChannel.open();
        clientChannel.connect(new InetSocketAddress(clientPort));
        clientChannel.read(
                ByteBuffer.allocate(1024),
                null,
                new CompletionHandler<Integer, Object>() {
                    @Override
                    public void completed(Integer result, Object attachment) {
                        // doSomething()
                    }

                    @Override
                    public void failed(Throwable exc, Object attachment) { }
                });
```

`JDK5`的`NIO`已经提供了相关的API，虽然操作更为复杂一些，但在此基础上，诸如`Netty`等通信框架已经发展的十分繁荣。`AIO`似乎并没有达到预计的效果，但这种回调方式显然要比直接`get()`的粗暴方式要更为优雅。
那么有没有不那么粗暴又方便一些的回调方案呢？
答案是有的，一些开源的工具已经为我们提供了这个功能，例如接下来要介绍的Google扩展包`Guava`中提供的并发工具` com.google.common.util.concurrent.ListenableFuture`

```java
    public static void main(String... args) {
        // 对jdk提供的线程池进行修饰，用于返回ListenabledFuture
        ListeningExecutorService executorService = MoreExecutors
                .listeningDecorator(Executors.newCachedThreadPool());

        // 提交一个Callable，返回ListenableFuture
        final ListenableFuture<Integer> listenableFuture = executorService.submit(() -> {
            System.out.println("call execute..");
            TimeUnit.SECONDS.sleep(1);
            return 7;
        });

        // 为listenableFuture添加回调函数，没有任何异常则分支进入onSuccess处理，否则进入onFailure分支
        Futures.addCallback(listenableFuture,  new FutureCallback() {
            public void onSuccess(Object o) {
                System.out.println("异步处理成功,result="+o);
            }

            public void onFailure(Throwable throwable) {
                System.out.println("异步处理失败,e="+throwable);
            }
        }, MoreExecutors.directExecutor());

        System.out.println("不会阻塞");

    }
```

和`Future`的`get()`会阻塞主线程不同，带监听器的`ListenableFuture`可以异步处理`Callable`结果，最终打印结果：

```
call execute..
不会阻塞
异步处理成功,result=7
```

从`ListeningExecutorService`这个修饰后的线程池出发，看看如何修饰后如何将提交的`Callable`输出为`ListenableFuture`，而非`Future`，主要来看`submit`方法。
!["AbstractListeningExecutorService"](/images/future1.jpg)

```java
public abstract class AbstractListeningExecutorService extends AbstractExecutorService
    implements ListeningExecutorService {
    @Override
    public <T> ListenableFuture<T> submit(Callable<T> task) {
        return (ListenableFuture<T>) super.submit(task);
    }
    
    @Override
    protected final <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
        return TrustedListenableFutureTask.create(callable);
    }    
}

public abstract class AbstractExecutorService implements ExecutorService {
    public <T> Future<T> submit(Callable<T> task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task);
        execute(ftask);
        return ftask;
    }
    
    protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
        return new FutureTask<T>(callable);
    }    
}
```

这里的`submit`和`newTaskFor`最终执行的都是子类`AbstractListeningExecutorService`中的，这是`Guava`包内的，而非`J.C.U`包内的`AbstractExecutorService`，返回了`TrustedListenableFutureTask`的实例，看一下依赖关系，能很清楚地看到这是`ListenableFuture`的一个实现类。
!["AbstractListeningExecutorService"](/images/future2.jpg)
那么是如何进行回调的呢？接着从刚才的实现类`TrustedListenableFutureTask`来看，主要做的工作是`InterruptibleTask`里，实现了`Runnable`的`run`方法。

```java
class TrustedListenableFutureTask<V> extends FluentFuture.TrustedFuture<V>
    implements RunnableFuture<V> {
    
    private volatile InterruptibleTask<?> task;

    TrustedListenableFutureTask(Callable<V> callable) {
        this.task = new TrustedFutureInterruptibleTask(callable);
    }    
}

abstract class InterruptibleTask<T> extends AtomicReference<Runnable> implements Runnable {
    // DoNothingRunnable什么也不做，空转
    private static final Runnable DONE = new DoNothingRunnable();  // 完成中断，置为DONE
    private static final Runnable INTERRUPTING = new DoNothingRunnable();
    private static final Runnable PARKED = new DoNothingRunnable();
    // Why 1000?  WHY NOT!  【😐】
    private static final int MAX_BUSY_WAIT_SPINS = 1000;
    @Override
    public final void run() {
        Thread currentThread = Thread.currentThread();
        if (!compareAndSet(null, currentThread)) {
            return;  // 已经有其他运行的线程，CAS失败
        }
        boolean run = !isDone();
        T result = null;
        Throwable error = null;
        try {
            if (run) { 
                result = runInterruptibly();  // 最终执行了callable.call()
            }
        } catch (Throwable t) {
            error = t;
        } finally {
            // 尝试将当前线程置为DONE
            if (!compareAndSet(currentThread, DONE)) {
                boolean restoreInterruptedBit = false;
                int spinCount = 0;
                Runnable state = get();
                while (state == INTERRUPTING || state == PARKED) {
                    spinCount++;
                    // 超过最大自旋次数
                    if (spinCount > MAX_BUSY_WAIT_SPINS) {
                        if (state == PARKED || compareAndSet(INTERRUPTING, PARKED)) {
                            // 恢复中断标志位
                            restoreInterruptedBit = Thread.interrupted()||restoreInterruptedBit;
                            LockSupport.park(this);  // 阻塞线程
                        }
                    } else {
                        Thread.yield();  // 自旋让步
                    }
                    state = get();  // 更新状态
                }
                if (restoreInterruptedBit) {
                    currentThread.interrupt();  // 有中断
                }
            }
            // 完成中断
            if (run) {
                afterRanInterruptibly(result, error);
            }
        }
    }
    
    // 在runInterruptibly之前调用，如果是true，runInterruptibly和afterRanInterruptibly不再调用
    abstract boolean isDone();

    // 不计算Futures的结果，只处理可中断任务，因为listener有可能被中断
    abstract T runInterruptibly() throws Exception;

    // 计算Futures的结果，任何调用interruptTask发生的中断，都在此方法被调用前发生
    abstract void afterRanInterruptibly(@Nullable T result, @Nullable Throwable error);
}
```



