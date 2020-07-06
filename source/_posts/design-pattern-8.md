---
title: 设计模式-8（适配器模式）
date: 2020-07-06 21:54:52
tags: "Java"
categories: "Design Pattern"
---

适配器模式

!["adapter"](/images/design-pattern-adapter.jpg)

将一个类的接口转换成另一个接口。 Adapter 模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

目标接口Target

```java
public interface Target {
    void operation1();
}
```

待适配类Adaptee，与目标接口不一致

```java
public class Adaptee {
    void operation2() {}
}
```

适配器类

```java
public class Adapter implements Target {
    private Adaptee adaptee = new Adaptee();

    public void operation1() {
        // do something
        adaptee.sampleOperation1();
    }
}
```

测试类

```java
public class AdapterClient {
    public static void main(String... args) {
        Target adapter = new Adapter();
        adapter.operation1();
        
        // 比如目标接口的一个实现是Concrete，客户端感知不到接口的不同
        // Target concrete = new Concrete();
        // concrete.operation1
    }
}
```

