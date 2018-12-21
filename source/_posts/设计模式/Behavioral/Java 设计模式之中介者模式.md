---
title: Java 设计模式之中介者模式
categories: 设计模式
tags:
  - Java
  - 设计模式
abbrlink: a98fd302
date: 2018-10-20 10:20:30
---

## 模式动机 ###
在用户与用户直接聊天的设计方案中，用户对象之间存在很强的关联性，将导致系统出现如下问题：
 - 系统结构复杂：对象之间存在大量的相互关联和调用，若有一个对象发生变化，则需要跟踪和该对象关联的其他所有对象，并进行适当处理。
 - 对象可重用性差：由于一个对象和其他对象具有很强的关联，若没有其他对象的支持，一个对象很难被另一个系统或模块重用，这些对象表现出来更像一个不可分割的整体，职责较为混乱。
 - 系统扩展性低：增加一个新的对象需要在原有相关对象上增加引用，增加新的引用关系也需要调整原有对象，系统耦合度很高，对象操作很不灵活，扩展性差。

在面向对象的软件设计与开发过程中，根据“单一职责原则”，我们应该尽量将对象细化，使其只负责或呈现单一的职责。

对于一个模块，可能由很多对象构成，而且这些对象之间可能存在相互的引用，为了减少对象两两之间复杂的引用关系，使之成为一个松耦合的系统，我们需要使用中介者模式，这就是中介者模式的模式动机。

## 模式定义 ##
**`中介者模式(Mediator Pattern)`**：用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。中介者模式又称为调停者模式。

> 中介者模式是一种**`对象行为型模式`**。

## 模式结构 ##
### 角色组成 ###
中介者模式包含如下角色：
- **`Mediator(抽象中介者)`**它定义一个接口，该接口用于与各同事对象之间进行通信。
- **`ConcreteMediator(具体中介者)`**它是抽象中介者的子类，通过协调各个同事对象来实现协作行为，它维持了对各个同事对象的引用。
- **`Colleague(抽象同事类)`**它定义各个同事类公有的方法，并声明了一些抽象方法来供子类实现，同时它维持了一个对抽象中介者类的引用，其子类可以通过该引用来与中介者通信。
- **`ConcreteColleague(具体同事类)`**它是抽象同事类的子类；每一个同事对象在需要和其他同事对象通信时，先与中介者通信，通过中介者来间接完成与其他同事类的通信；在具体同事类中实现了在抽象同事类中声明的抽象方法。

### 结构图 ###
![命令模式结构图](https://lyl873825813.github.io/medias/design_pattern/mediator_uml.jpg)

### 时序图 ###
![命令模式时序图](https://lyl873825813.github.io/medias/design_pattern/mediator_seq.jpg)

## 模式实现 ##
首先，是抽象中介者和具体中介者。Mediator 抽象类充当抽象中介者，ConcreteMediator 类充当具体中介者，完整代码如下所示：
```java
/**
 * 抽象中介者
 */
abstract class Mediator {
    protected ArrayList<Colleague> colleagues; //用于存储同事对象

    /**
     * 注册方法，用于增加同事对象
     */
    public void register(Colleague colleague) {
        colleagues.add(colleague);
    }

    /**
     * 声明抽象的业务方法
     */
    public abstract void operation();
}
```

```java
/**
 * 具体中介者
 */
class ConcreteMediator extends Mediator {
    //实现业务方法，封装同事之间的调用
    public void operation() {
		......
        ((Colleague) (colleagues.get(0))).method1(); //通过中介者调用同事类的方法
		......
    }
}
```

然后，是抽象同事类和具体同事类。Colleague 抽象类充当抽象同事类，ConcreteColleagueA 和 ConcreteColleagueB 类充当具体同事类，完整代码如下所示：
```java
/**
 * 抽象同事类
 */
abstract class Colleague {

    protected Mediator mediator; //维持一个抽象中介者的引用

    public Colleague(Mediator mediator) {
        this.mediator = mediator;
    }

    public abstract void method1(); //声明自身方法，处理自己的行为

    /**
     * 定义依赖方法，与中介者进行通信
     */
    public void method() {
        mediator.operation();
        System.out.println("Colleague委托给中介者的业务逻辑...");

    }

}
```

```java
class ConcreteColleagueA extends Colleague {

    public ConcreteColleagueA(Mediator mediator) {
        super(mediator);
    }

    /**
     * 自有方法
     */
    public void methodA() {
        System.out.println("ConcreteColleagueA处理自己的业务逻辑...");
    }

}
```

```java
class ConcreteColleagueB extends Colleague {

    public ConcreteColleagueB(Mediator mediator) {
        super(mediator);
    }

    /**
     * 自有方法
     */
    public void methodB() {
        System.out.println("ConcreteColleagueB处理自己的业务逻辑...");
    }

}
```

最后，是客户端场景类，完整代码如下所示：
```java
public class MediatorClient {

    public static void main(String[] args) {
        Mediator mediator = new ConcreteMediator();

        ConcreteColleagueA colleagueA = new ConcreteColleagueA(mediator);
        ConcreteColleagueB colleagueB = new ConcreteColleagueB(mediator);
        mediator.register(colleagueA);
        mediator.register(colleagueB);

        colleagueA.method();
        colleagueB.method();
        colleagueA.methodA();
        colleagueB.methodB();
    }

}
```

## 模式分析 ##
中介者模式将一个网状的系统结构变成一个以中介者对象为中心的星形结构，在这个星型结构中，使用中介者对象与其他对象的一对多关系来取代原有对象之间的多对多关系。

中介者模式的核心在于中介者类的引入，在中介者模式中，中介者类承担了两方面的职责：
 - **中转作用（结构性）**：通过中介者提供的中转作用，各个同事对象就不再需要显式引用其他同事，当需要和其他同事进行通信时，可通过中介者来实现间接调用。该中转作用属于中介者在结构上的支持。
 - **协调作用（行为性）**：中介者可以更进一步的对同事之间的关系进行封装，同事可以一致的和中介者进行交互，而不需要指明中介者需要具体怎么做，中介者根据封装在自身内部的协调逻辑，对同事的请求进行进一步处理，将同事成员之间的关系行为进行分离和封装。该协调作用属于中介者在行为上的支持。

### 优点 ###
中介者模式的主要优点如下：
 - 中介者模式简化了对象之间的交互，它用中介者和同事的一对多交互代替了原来同事之间的多对多交互，一对多关系更容易理解、维护和扩展，将原本难以理解的网状结构转换成相对简单的星型结构。
 - 中介者模式可将各同事对象解耦。中介者有利于各同事之间的松耦合，我们可以独立的改变和复用每一个同事和中介者，增加新的中介者和新的同事类都比较方便，更好地符合“开闭原则”。
 - 可以减少子类生成，中介者将原本分布于多个对象间的行为集中在一起，改变这些行为只需生成新的中介者子类即可，这使各个同事类可被重用，无须对同事类进行扩展。

### 缺点 ###
中介者模式的主要缺点如下：
 - 在具体中介者类中包含了大量同事之间的交互细节，可能会导致具体中介者类非常复杂，使得系统难以维护。

### 适用环境 ###
在以下情况下可以考虑使用中介者模式：
 - 系统中对象之间存在复杂的引用关系，系统结构混乱且难以理解。
 - 一个对象由于引用了其他很多对象并且直接和这些对象通信，导致难以复用该对象。
 - 想通过一个中间类来封装多个类中的行为，而又不想生成太多的子类。可以通过引入中介者类来实现，在中介者中定义对象交互的公共行为，如果需要改变行为则可以增加新的具体中介者类。

## 模式总结 ###
 - 中介者模式用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。中介者模式又称为调停者模式，它是一种对象行为型模式。
 - 中介者模式包含四个角色：抽象中介者用于定义一个接口，该接口用于与各同事对象之间的通信；具体中介者是抽象中介者的子类，通过协调各个同事对象来实现协作行为，了解并维护它的各个同事对象的引用；抽象同事类定义各同事的公有方法；具体同事类是抽象同事类的子类，每一个同事对象都引用一个中介者对象；每一个同事对象在需要和其他同事对象通信时，先与中介者通信，通过中介者来间接完成与其他同事类的通信；在具体同事类中实现了在抽象同事类中定义的方法。
 - 通过引入中介者对象，可以将系统的网状结构变成以中介者为中心的星形结构，中介者承担了中转作用和协调作用。中介者类是中介者模式的核心，它对整个系统进行控制和协调，简化了对象之间的交互，还可以对对象间的交互进行进一步的控制。
 - 中介者模式的主要优点在于简化了对象之间的交互，将各同事解耦，还可以减少子类生成，对于复杂的对象之间的交互，通过引入中介者，可以简化各同事类的设计和实现；中介者模式主要缺点在于具体中介者类中包含了同事之间的交互细节，可能会导致具体中介者类非常复杂，使得系统难以维护。
 - 中介者模式适用情况包括：系统中对象之间存在复杂的引用关系，产生的相互依赖关系结构混乱且难以理解；一个对象由于引用了其他很多对象并且直接和这些对象通信，导致难以复用该对象；想通过一个中间类来封装多个类中的行为，而又不想生成太多的子类。

## 致谢 ##
[Java Design Pattern](https://www.gitbook.com/book/quanke/design-pattern-java/)

