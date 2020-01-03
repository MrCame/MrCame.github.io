---
title: 设计模式-3（工厂方法模式）
date: 2019-12-19 23:00:01
tags: "Java"
categories: "Design Pattern"
---

工厂方法模式

!["factory-method"](/images/design-pattern-simple-factory.jpg)

工厂类

```java
// 抽象工厂类
abstract class Factory {
    public abstract Product produce();
}

// 具体工厂A——生产A
class Factory AF extends Factory {
    @Override
    public Product produce() {
        return new A();
    }
} 

// 具体工厂B——生产B
class Factory BF extends Factory {
    @Override
    public Product produce() {
        return new B();
    }
}
```

产品类

```java
// 抽象产品类
abstract class Product {
    public abstract void show();
}

// 具体产品类A
class Product A extends Product {
    @Override
    public void show() {
        System.out.println("product A");
    }
}

// 具体产品类B
class Product B extends Product {
    @Override
    public void show() {
        System.out.println("product B");
    }
}

```

测试类

```java
public class Test {
    public static void main(String... args) {
        Factory af = new AF();
        Product a = af.produce();
        a.show();
    }
} 
```

> #### 简单工厂模式与OOP原则
>
> ##### 已遵循的原则
>
> - 依赖倒置原则
> - 迪米特法则
> - 里氏替换原则
> - 接口隔离原则
> - 单一职责原则（每个工厂只负责创建自己的具体产品，没有简单工厂中的逻辑判断）
> - 开闭原则（增加新的产品，不像简单工厂那样需要修改已有的工厂，而只需增加相应的具体工厂类）
>
> ##### 未遵循的原则
>
> - 开闭原则（虽然工厂对修改关闭了，但更换产品时，客户代码还是需要修改）

