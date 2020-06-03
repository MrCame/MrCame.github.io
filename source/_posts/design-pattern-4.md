---
title: 设计模式-4（抽象工厂模式）
date: 2020-05-27 23:06:55
tags: "Java"
categories: "Design Pattern"
---

抽象工厂模式

!["abstract-factory"](/images/design-pattern-abstract-factory.jpg)

核心思想：向客户端提供一个接口，使得客户端在不必指定具体产品的情况下，创建多个产品族中的产品对象

与工厂方法模式的区别：工厂方法模式中一种工厂只能创建一种具体产品。而在抽象工厂模式中一种具体工厂可以创建多个种类的具体产品

抽象产品（CPU、主板）

```java
public interface Cpu {
    public void calculate();
}

public interface MainBoard {
    public void installCPU();
}
```

具体CPU产品

```java
// AMD CPU
public class AmdCpu implements Cpu {
    private int pins = 0;
    public AmdCpu(int pins) {
        this.pins = pins;
    }
    @Override
    public void calculate() {
        System.out.println("AMD CPU 的针脚数" + pins);
    }
}

// INTEL CPU
public class IntelCpu implements Cpu {
    private int pins = 0;
    public IntelCpu(int pins) {
        this.pins = pins;
    }
    @Override
    public void calculate() {
        System.out.println("Intel CPU 的针脚数" + pins);
    }
}
```

具体主板产品

```java
// AMD主板
public class AmdMainBoard implements MainBoard {
    private int cpuHoles = 0;
    public AmdMainBoard(int cpuHoles) {
        this.cpuHoles = cpuHoles;
    }
    @Override
    public void installCPU() {
        System.out.println("AMD主板的CPU插槽孔数是：" + cpuHoles);
    }
}

// INTEL主板
public class IntelMainBoard implements MainBoard {
    private int cpuHoles = 0;
    public IntelMainBoard(int cpuHoles) {
        this.cpuHoles = cpuHoles;
    }
    @Override
    public void installCPU() {
        System.out.println("Intel主板的CPU插槽孔数：" + cpuHoles);
    }
}
```



抽象工厂类

```java
public interface AbstractFactory {
    public Cpu createCpu();
    public MainBoard createMainBoard();
}
```

具体工厂类

```java
// AMD工厂
public class AmdFactory implements AbstractFactory {
    @Override
    public Cpu createCpu() {
        return new AmdCpu(938);
    }

    @Override
    public MainBoard createMainBoard() {
        return new AmdMainBoard(938);
    }
}

// INTEL工厂
public class IntelFactory implements AbstractFactory {
    @Override
    public Cpu createCpu() {
        return new IntelCpu(755);
    }

    @Override
    public MainBoard createMainBoard() {
        return new IntelMainBoard(755);
    }
}
```

测试类

```java
public class ComputerEngineer {
    private Cpu cpu = null;
    private MainBoard mainboard = null;

    public static void main(String... args) {
        AbstractFactory
                amdFactory   = new AmdFactory(),
                intelFactory = new IntelFactory();
        ComputerEngineer engineer = new ComputerEngineer();
        engineer.makeComputer(amdFactory);
        engineer.makeComputer(intelFactory);
    }

    public void makeComputer(AbstractFactory af) {
        /**
         * 组装机器的基本步骤
         */
        //1:首先准备好装机所需要的配件
        prepareHardwares(af);
        //2:组装机器
        //3:测试机器
        //4：交付客户
    }

    private void prepareHardwares(AbstractFactory af){
        //这里要去准备CPU和主板的具体实现，为了示例简单，这里只准备这两个
        //可是，装机工程师并不知道如何去创建，怎么办呢？

        //直接找相应的工厂获取
        this.cpu = af.createCpu();
        this.mainboard = af.createMainBoard();

        //测试配件是否好用
        this.cpu.calculate();
        this.mainboard.installCPU();
    }
}
```



