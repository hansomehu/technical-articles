---
layout: post
title: "design patterns"
permalink: /java-design-patterns
---

常见的设计模式，比较实用且简单的几种，面试必备！



#### 适配器模式

![image-20220521210816227](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220521210816227.png)

关于适配器模式最经典的例子就是电源适配器，220V的交流电不能给手机充电，那么需要一个电源适配器来将高压交流电转化为低压直流电。

在程序语言里就是，某个类由于种种原因无法直接使用，但是我们又需要用到那个类中的方法所提供的功能。那么这个时候，我们使用一个适配器，将目标类中的方法做一个转换，使得客户端能对其调用（间接）。

适配器模式中的角色：Target（对象，上图中的Print类）、Client(请求者)、Adaptee（被适配，上图中的Banner类）、Adapter（适配器，上图中的PrintBanner类）

```java
public class AdapterPattern {
    public static void main(String[] args) {

//        new Adapter(new OriginalVoltage()).converter();

    }

    class OriginalVoltage{
        public String output220V(){
            return "220V";
        }
    }

    interface Converter{
        public String converter();
    }

    class Adapter implements Converter{
        private OriginalVoltage originalVoltage;

        public Adapter(OriginalVoltage originalVoltage) {
            this.originalVoltage = originalVoltage;
        }

        @Override
        public String converter() {
            String voltage = originalVoltage.output220V();
            String result = "failed";
            if (voltage == "220V") {
                result = "20V";
                return result;
            }
            return result;
        }
    }

}

```



#### 单例模式

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220521213018603.png" alt="image-20220521213018603" style="zoom:33%;" />

一个类不管怎么new都只能存在一个实例，在Spring中管理的Bean对象大多数都是单例的

这种模式实现的核心在于，单例类中新建实例的方法是static的，不能new

```java
public class Singleton {

    private static Singleton singleton = new Singleton();

    private Singleton(){
        System.out.println("creating a instance");
    }

    // 实现的逻辑就在于该获取实例的方法是静态的不能new，导致全局只有一个类的实例
    public static Singleton getInstance(){
        return singleton;
    }

    // 即便是调用了两次getInstance，但实际上Singleton的构造方法只被调用了一次
    public static void main(String[] args) {
        Object o1 = Singleton.getInstance();
        Object o2 = Singleton.getInstance();

        System.out.println(
                o2==o1
        );
    }
}

```



#### 建造者模式 Builder

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220521220236231.png" alt="image-20220521220236231" style="zoom: 25%;" />

用于组装具有复杂结构的实例。这里我将用造电动汽车的例子来解释，把电动汽车的制造分为Shell和Batery两部分，这两个就是具体的ConcreteBuilder，负责业务逻辑。他们的父类Builder则是定义了一辆电动汽车需要由哪些部件组成。而Director则是决定如何组装的。

```java
// builder
abstract class TeslaBuilder {
    public abstract String buildShell();
    public abstract String buildBatery();
}

// concrete builder X
public class TeslaModelXBuilder extends TeslaBuilder {
    @Override
    public String buildShell() {
        return "building Shell for Tesla Model X...";
    }

    @Override
    public String buildBatery() {
        return "building Batery for Tesla Model X...";
    }
}

// concrete builder Y
public class TeslaModelYBuilder extends TeslaBuilder {
    @Override
    public String buildShell() {
        return "building Shell for Tesla Model Y...";
    }

    @Override
    public String buildBatery() {
        return "building Shell for Tesla Model Y...";
    }
}

// director
public class TeslaDirector {

    private TeslaBuilder builder;

    public TeslaDirector(TeslaBuilder builder) {
        this.builder = builder;
    }

    public void ShanghaiFactoryConstruct(){
        builder.buildShell();
        builder.buildBatery();
    }

    public static void main(String[] args) {
        // build model x
        TeslaModelXBuilder modelXBuilder = new TeslaModelXBuilder();
        TeslaDirector directorX = new TeslaDirector(modelXBuilder);
        directorX.ShanghaiFactoryConstruct();

        // build model y
        TeslaModelYBuilder modelYBuilder = new TeslaModelYBuilder();
        TeslaDirector directorY = new TeslaDirector(modelYBuilder);
        directorY.ShanghaiFactoryConstruct();
    }
}
```





#### 工厂模式 FactoryMethod