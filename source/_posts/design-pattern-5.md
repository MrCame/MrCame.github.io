---
title: 设计模式-5（原型模式）
date: 2020-06-03 22:25:14
tags: "Java"
categories: "Design Pattern"
---

原型模式

!["prototype"](/images/design-pattern-prototype.jpg)

原型模式通过复制原型（prototype）获得创建新对象的能力，原型本身就是个工厂。用于隔离类的使用者和具体类型之间的耦合关系，隐藏了对象创建的细节，避免了调用过多参数的构造方法而带来的性能影响。与抽象工厂方法的却别在于重在自身的复制。

原型接口，自行实现clone，也可以是实现Cloneable的抽象类

```java
public interface Prototype {
    Object clone();
}
```

具体对象

```java
public class ConcretePrototypeA implements Prototype {
    @Override
    public Object clone() {
        return new ConcretePrototypeA();
    }
}

public class ConcretePrototypeB implements Prototype {
    @Override
    public Object clone() {
        return new ConcretePrototypeB();
    }
}
```

测试类

```java
public class Test {
    public static void main(String... args) {
        Prototype 
                prototype1 = new ConcretePrototypeA(),
                prototype2 = new ConcretePrototypeB();
        ConcretePrototypeA concretePrototypeA = (ConcretePrototypeA) prototype1.clone();
        ConcretePrototypeB concretePrototypeB = (ConcretePrototypeB) prototype2.clone();
    }
}
```

