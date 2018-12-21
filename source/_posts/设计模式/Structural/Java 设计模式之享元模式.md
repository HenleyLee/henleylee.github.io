---
title: Java 设计模式之享元模式
categories: 设计模式
tags:
  - Java
  - 设计模式
abbrlink: 545f3e64
date: 2018-09-27 18:36:42
---

## 模式动机 ##
面向对象技术可以很好地解决一些灵活性或可扩展性问题，但在很多情况下需要在系统中增加类和对象的个数。当对象数量太多时，将导致运行代价过高，带来性能下降等问题。享元模式通过共享技术实现相同或相似对象的重用提高系统资源的利用率。本文首先阐述了享元模式要解决的问题和解决问题的理念，然后从实现角度重点说明了该模式的本质，并进一步给出了其所包含的角色和组织结构。最后，给出了共享模式的应用实例和使用场景。这就是享元模式的动机。

## 模式定义 ##
**`享元模式(Flyweight Pattern)`**运用共享技术有效地支持大量细粒度对象的复用。系统只使用少量的对象，而这些对象都很相似，状态变化很小，可以实现对象的多次复用。由于享元模式要求能够共享的对象必须是细粒度对象，因此它又称为轻量级模式。

> 享元模式是一种**`对象结构型模式`**。

## 模式结构 ##
### 角色组成 ###
享元模式包含如下角色：
 - **`Flyweight(抽象享元类)：`**通常是一个接口或抽象类，在抽象享元类中声明了具体享元类公共的方法，这些方法可以向外界提供享元对象的内部数据（内部状态），同时也可以通过这些方法来设置外部数据（外部状态）。
 - **`ConcreteFlyweight(具体享元类)：`**它实现了抽象享元类，其实例称为享元对象；在具体享元类中为内部状态提供了存储空间。通常我们可以结合单例模式来设计具体享元类，为每一个具体享元类提供唯一的享元对象。
 - **`UnsharedConcreteFlyweight(非共享具体享元类)：`**并不是所有的抽象享元类的子类都需要被共享，不能被共享的子类可设计为非共享具体享元类；当需要一个非共享具体享元类的对象时可以直接通过实例化创建。
 - **`FlyweightFactory(享元工厂类)：`**享元工厂类用于创建并管理享元对象，它针对抽象享元类编程，将各种类型的具体享元对象存储在一个享元池中，享元池一般设计为一个存储“键值对”的集合（也可以是其他类型的集合），可以结合工厂模式进行设计；当用户请求一个具体享元对象时，享元工厂提供一个存储在享元池中已创建的实例或者创建一个新的实例（如果不存在的话），返回新创建的实例并将其存储在享元池中。

### 结构图 ###
![享元模式结构图](https://lyl873825813.github.io/medias/design_pattern/flyweight_uml.jpg)

### 时序图 ###
![享元模式时序图](https://lyl873825813.github.io/medias/design_pattern/flyweight_seq.jpg)

## 模式实现 ##
标准的享元模式结构图中既包含可以共享的具体享元类，也包含不可以共享的非共享具体享元类。但是在实际使用过程中，我们有时候会用到两种特殊的享元模式：单纯享元模式和复合享元模式。

### 单纯享元模式 ###
首先，是抽象享元类和具体享元类。Flyweight 接口充当抽象享元类，ConcreteFlyweight 类充当具体享元类，完整代码如下所示：
```java
/**
 * 抽象享元类
 */
public interface Flyweight {

    /**
     * 示意性方法，参数state是外蕴状态
     */
    void operation(String state);
}
```

```java
/**
 * 具体享元类
 */
public class ConcreteFlyweight implements Flyweight {

    /** 内部状态intrinsicState作为成员变量，同一个享元对象其内部状态是一致的 */
    private String intrinsicState;

    /**
     * 构造函数，内蕴状态作为参数传入
     */
    public ConcreteFlyweight(String intrinsicState) {
        this.intrinsicState = intrinsicState;
    }


    /**
     * 外蕴状态作为参数传入方法中，改变方法的行为，但是并不改变对象的内蕴状态。
     */
    @Override
    public void operation(String extrinsicState) {
        System.out.println("Intrinsic State = " + this.intrinsicState);
        System.out.println("Extrinsic State = " + extrinsicState);
    }

}
```

然后，是享元工厂类。Abstraction 类充当享元工厂类，完整代码如下所示：
```java
/**
 * 享元工厂类
 */
public class FlyweightFactory {

    /**
     * 定义一个HashMap用于存储享元对象，实现享元池
     */
    private HashMap<String, Flyweight> flyweights = new HashMap<String, Flyweight>();

    public Flyweight getFlyweight(String state) {
        // 如果对象存在，则直接从享元池获取
        if (flyweights.containsKey(state)) {
            return flyweights.get(state);
        }
        // 如果对象不存在，先创建一个新的对象添加到享元池中，然后返回
        Flyweight flyweight = new ConcreteFlyweight(state);
        // 把这个新的Flyweight对象添加到缓存中
        flyweights.put(state, flyweight);
        return flyweight;
    }
}
```

最后，是客户端场景类，完整代码如下所示：
```java
public class FlyweightClient {

    public static void main(String[] args) {
        FlyweightFactory factory = new FlyweightFactory();
        Flyweight flyweight = factory.getFlyweight("A");
        flyweight.operation("First Call");

        flyweight = factory.getFlyweight("B");
        flyweight.operation("Second Call");

        flyweight = factory.getFlyweight("A");
        flyweight.operation("Third Call");
    }

}
```
> 在单纯享元模式中，所有的具体享元类都是可以共享的，即所有抽象享元类的子类都可共享，不存在非共享具体享元类。

### 复合享元模式 ###
首先，是抽象享元类、具体享元类和非共享具体享元类。Flyweight 接口充当抽象享元类，ConcreteFlyweight 类充当具体享元类，UnsharedConcreteFlyweight 类充当非共享具体享元类，完整代码如下所示：
```java
/**
 * 抽象享元类
 */
public interface Flyweight {

    /**
     * 示意性方法，参数state是外蕴状态
     */
    void operation(String state);
}
```

```java
/**
 * 具体享元类
 */
public class ConcreteFlyweight implements Flyweight {

    /** 内部状态intrinsicState作为成员变量，同一个享元对象其内部状态是一致的 */
    private String intrinsicState;

    /**
     * 构造函数，内蕴状态作为参数传入
     */
    public ConcreteFlyweight(String intrinsicState) {
        this.intrinsicState = intrinsicState;
    }


    /**
     * 外蕴状态作为参数传入方法中，改变方法的行为，但是并不改变对象的内蕴状态。
     */
    @Override
    public void operation(String extrinsicState) {
        System.out.println("Intrinsic State = " + this.intrinsicState);
        System.out.println("Extrinsic State = " + extrinsicState);
    }

}
```

```java
/**
 * 非共享具体享元类
 */
public class UnsharedConcreteFlyweight implements Flyweight {

    private Map<String,Flyweight> flyweights = new HashMap<String,Flyweight>();
    /**
     * 增加一个新的单纯享元对象到聚集中
     */
    public void add(String key , Flyweight flyweight){
        flyweights.put(key,flyweight);
    }

    /**
     * 外蕴状态作为参数传入到方法中
     */
    @Override
    public void operation(String state) {
        Collection<Flyweight> values = flyweights.values();
        for(Flyweight flyweight : values){
            flyweight.operation(state);
        }

    }

}
```

然后，是享元工厂类。Abstraction 类充当享元工厂类，完整代码如下所示：
```java
/**
 * 享元工厂类
 */
public class FlyweightFactory {

    private Map<String,Flyweight> flyweights = new HashMap<String,Flyweight>();

    /**
     * 复合享元工厂方法
     */
    public Flyweight getFlyweight(List<String> compositeState) {
        UnsharedConcreteFlyweight compositeFly = new UnsharedConcreteFlyweight();
        for (String state : compositeState) {
            compositeFly.add(state, this.getFlyweight(state));
        }
        return compositeFly;
    }

    /**
     * 单纯享元工厂方法
     */
    public Flyweight getFlyweight(String state) {
        // 如果对象存在，则直接从享元池获取
        if (flyweights.containsKey(state)) {
            return flyweights.get(state);
        }
        // 如果对象不存在，先创建一个新的对象添加到享元池中，然后返回
        Flyweight flyweight = new ConcreteFlyweight(state);
        // 把这个新的Flyweight对象添加到缓存中
        flyweights.put(state, flyweight);
        return flyweight;
    }

}
```

最后，是客户端场景类，完整代码如下所示：
```java
public class FlyweightClient {

    public static void main(String[] args) {
        List<String> compositeState = new ArrayList<String>();
        compositeState.add("A");
        compositeState.add("B");
        compositeState.add("C");
        compositeState.add("A");
        compositeState.add("B");

        FlyweightFactory factory = new FlyweightFactory();
        Flyweight compositeFly1 = factory.getFlyweight(compositeState);
        Flyweight compositeFly2 = factory.getFlyweight(compositeState);
        compositeFly1.operation("Composite Call");
        System.out.println("复合享元模式是否可以共享对象：" + (compositeFly1 == compositeFly2));

        String state = "A";
        Flyweight flyweight1 = factory.getFlyweight(state);
        Flyweight flyweight2 = factory.getFlyweight(state);
        System.out.println("单纯享元模式是否可以共享对象：" + (flyweight1 == flyweight2));
    }

}
```
> 将一些单纯享元使用组合模式加以组合，可以形成复合享元对象，这样的复合享元对象本身不能共享，但是它们可以分解成单纯享元对象，而后者则可以共享。

## 模式分析 ##
享元模式是一个考虑系统性能的设计模式，通过使用享元模式可以节约内存空间，提高系统的性能。

享元模式的核心在于享元工厂类，享元工厂类的作用在于提供一个用于存储享元对象的享元池，用户需要对象时，首先从享元池中获取，如果享元池中不存在，则创建一个新的享元对象返回给用户，并在享元池中保存该新增对象。

享元模式以共享的方式高效地支持大量的细粒度对象，享元对象能做到共享的关键是区分内部状态(Internal State)和外部状态(External State)：
 - 内部状态是存储在享元对象内部并且不会随环境改变而改变的状态，因此内部状态可以共享。
 - 外部状态是随环境改变而改变的、不可以共享的状态。享元对象的外部状态必须由客户端保存，并在享元对象被创建之后，在需要使用的时候再传入到享元对象内部。一个外部状态与另一个外部状态之间是相互独立的。

### 优点 ###
享元模式的主要优点如下：
 - 可以极大减少内存中对象的数量，使得相同或相似对象在内存中只保存一份，从而可以节约系统资源，提高系统性能。
 - 享元模式的外部状态相对独立，而且不会影响其内部状态，从而使得享元对象可以在不同的环境中被共享。

### 缺点 ###
享元模式的主要缺点如下：
 - 享元模式使得系统变得复杂，需要分离出内部状态和外部状态，这使得程序的逻辑复杂化。
 - 为了使对象可以共享，享元模式需要将享元对象的部分状态外部化，而读取外部状态将使得运行时间变长。

### 适用环境 ###
在以下情况下可以考虑使用享元模式：
 - 一个系统有大量相同或者相似的对象，造成内存的大量耗费。
 - 对象的大部分状态都可以外部化，可以将这些外部状态传入对象中。
 - 在使用享元模式时需要维护一个存储享元对象的享元池，而这需要耗费一定的系统资源，因此，应当在需要多次重复使用享元对象时才值得使用享元模式。

### 模式扩展 ###
享元模式通常需要和其他模式一起联用，几种常见的联用方式如下：
 - 在享元模式的享元工厂类中通常提供一个静态的工厂方法用于返回享元对象，使用简单工厂模式来生成享元对象。
 - 在一个系统中，通常只有唯一一个享元工厂，因此可以使用单例模式进行享元工厂类的设计。
 - 享元模式可以结合组合模式形成复合享元模式，统一对多个享元对象设置外部状态。

## 模式总结 ##
 - 享元模式运用共享技术有效地支持大量细粒度对象的复用。系统只使用少量的对象，而这些对象都很相似，状态变化很小，可以实现对象的多次复用，它是一种对象结构型模式。
 - 享元模式包含四个角色：抽象享元类声明一个接口，通过它可以接受并作用于外部状态；具体享元类实现了抽象享元接口，其实例称为享元对象；非共享具体享元是不能被共享的抽象享元类的子类；享元工厂类用于创建并管理享元对象，它针对抽象享元类编程，将各种类型的具体享元对象存储在一个享元池中。
 - 享元模式以共享的方式高效地支持大量的细粒度对象，享元对象能做到共享的关键是区分内部状态和外部状态。其中内部状态是存储在享元对象内部并且不会随环境改变而改变的状态，因此内部状态可以共享；外部状态是随环境改变而改变的、不可以共享的状态。
 - 享元模式主要优点在于它可以极大减少内存中对象的数量，使得相同对象或相似对象在内存中只保存一份；其缺点是使得系统更加复杂，并且需要将享元对象的状态外部化，而读取外部状态使得运行时间变长。
 - 享元模式适用情况包括：一个系统有大量相同或者相似的对象，由于这类对象的大量使用，造成内存的大量耗费；对象的大部分状态都可以外部化，可以将这些外部状态传入对象中；多次重复使用享元对象。

## 致谢 ##
[Java Design Pattern](https://www.gitbook.com/book/quanke/design-pattern-java/)

