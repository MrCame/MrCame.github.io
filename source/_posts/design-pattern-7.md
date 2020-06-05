---
title: 设计模式-7（单例模式）
date: 2020-06-05 20:15:24
tags: "Java"
categories: "Design Pattern"
---

单例模式

懒汉式——线程不安全

```java
public class Singleton0 {
    private static Singleton0 singleton0;
    private Singleton0() {}
    public static Singleton0 getInstance() {
        if (singleton0 == null) {
            singleton0 = new Singleton0();
        }
        return singleton0;
    }
}
```

懒汉式——线程安全，但浪费内存

```java
public class Singleton1 {
    private static Singleton1 singleton1;
    private Singleton1() {}
    public static synchronized Singleton1 getInstance() {
        if (singleton1 == null) {
            singleton1 = new Singleton1();
        }
        return singleton1;
    }
}
```

饿汉式——线程安全，但浪费内存

```java
public class Singleton2 {
    private static Singleton2 singleton2 = new Singleton2();
    private Singleton2() {}
    public static Singleton2 getInstance() {
        return singleton2;
    }
}
```

登记式/静态内部类——推荐

```java
public class Singleton3 {
    private static class SingletonHolder {
        private static Singleton3 singleton3 = new Singleton3();
    }
    private Singleton3() {}
    public static Singleton3 getInstance() {
        return SingletonHolder.singleton3;
    }
}
```

双检锁（DCL）——推荐

```java
public class Singleton4 {
    // volatile禁止指令重排序
    private volatile static Singleton4 singleton4;
    private Singleton4() {}
    public static Singleton4 getInstance() {
        if (singleton4 == null) {
            synchronized (Singleton4.class) {
                if (singleton4 == null) {
                    singleton4 = new Singleton4();
                }
            }
        }
        return singleton4;
    }
}
```

枚举——推荐

```java
public enum Singleton5 {
    INSTANCE;
    public void whatSoEverMethod() { }
    // 该方法非必须
    public static Singleton5 getInstance() {
        return INSTANCE;
    }
}
```

