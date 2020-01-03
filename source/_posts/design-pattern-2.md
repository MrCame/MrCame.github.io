---
title: 设计模式-2（简单工厂模式）
date: 2019-12-17 21:53:56
tags: "Java"
categories: "Design Pattern"
---

简单工厂模式

!["simple-factory"](/images/design-pattern-simple-factory.jpg)

```java
abstract class Car {
    public abstract void speed();
}
```

```java
public class BMW extends Car {
    @Override
    public void speed() {
        // do something
    }
}

public class Benz extends Car {
    @Override
    public void speed() {
        // do something
    }
}
```

```java
class Factory {
    public static Car produce(String productName) {
        switch(productName) {
            case "BMW":
                return new BMW();
            case "Benz":
                return new Benz();
            default:
                return null;
        }
    }
}
```

```java
public class Driver {
    public void drive() {
        // 可以根据配置文件来读取产品名称
        XMLConfiguration config = new XMLConfiguration("car.xml");
        String productName = config.getString("BMW");
        Car car = Factory.produce(productName);
        car.speed();
    }
}
```

> #### **简单工厂模式与OOP原则**
>
> ##### 已遵守的原则
>
> - 依赖倒置原则
> - 迪米特法则
> - 里氏替换原则
> - 接口隔离原则
>
> ##### 未遵循的原则
>
> - 开闭原则（如上文所述，利用配置文件+反射或者注解可以避免这一点）
> - 单一职责原则（工厂类即要负责逻辑判断又要负责实例创建）