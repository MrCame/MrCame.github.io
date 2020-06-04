---
title: 设计模式-6（建造者模式）
date: 2020-06-04 23:17:32
tags: "Java"
categories: "Design Pattern"
---

建造者模式

!["builder"](/images/design-pattern-builder.jpg)

建造者模式与抽象工厂方式都是用来创建复杂的大对象，但builder模式是一步步创建，一般不是直接返回对象。好处是对象的构建代码与标识代码分离了。

要建造的产品

```java
public class Product {
    private String part1;
    private String part2;
}
```

建造者接口

```java
public interface Builder {
    void buildPart1();
    void buildPart2();
    Product getProduct();
}
```

具体的建造者

```java
public class ConcreteBuilder implements Builder {
    private Product product = new Product();

    public void buildPart1() {
        product.setPart1("NO: 89757");
    }

    public void buildPart2() {
        product.setPart2("NAME: abc");
    }

    public Product getProduct() {
        return product;
    }
}

```

Director类

```java
public class Director {
    private Builder builder;

    public Director(Builder builder) {
        this.builder = builder;
    }

    public void construct() {
        builder.buildPart1();
        builder.buildPart2();
    }
}
```

具体使用的测试类

```java
public class Client {
    public static void main(String... args) {
        Builder builder = new ConcreteBuilder();
        Director director = new Director(builder);
        director.construct();
        Product product = builder.getProduct();
        System.out.print(product);
    }
}
```

