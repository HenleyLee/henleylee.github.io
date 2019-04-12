---
title: Java 设计模式之访问者模式
categories: 设计模式
tags:
  - Java
  - 设计模式
abbrlink: 1d0ccac7
date: 2018-10-08 18:29:25
---

## 模式动机 ##
访问者模式的目的是封装一些施加于某种数据结构元素之上的操作，一旦这些操作需要修改的话，接受这个操作的数据结构可以保持不变。为不同类型的元素提供多种访问操作方式，且可以在不修改原有系统的情况下增加新的操作方式，这就是访问者模式的模式动机。

## 模式定义 ##
**`访问者模式(Visitor Pattern)`**提供一个作用于某对象结构中的各元素的操作表示，它使我们可以在不改变各元素的类的前提下定义作用于这些元素的新操作。

> 访问者模式是一种**`对象行为型模式`**。

## 模式结构 ##
### 角色组成 ###
访问者模式包含如下角色：
 - **`Vistor(抽象访问者)：`**抽象访问者为对象结构中每一个具体元素类 ConcreteElement 声明一个访问操作，从这个操作的名称或参数类型可以清楚知道需要访问的具体元素的类型，具体访问者需要实现这些操作方法，定义对这些元素的访问操作。
 - **`ConcreteVisitor(具体访问者)：`**具体访问者实现了每个由抽象访问者声明的操作，每一个操作用于访问对象结构中一种类型的元素。
 - **`Element(抽象元素)：`**抽象元素一般是抽象类或者接口，它定义一个 accept() 方法，该方法通常以一个抽象访问者作为参数。
 - **`ConcreteElement(具体元素)：`**具体元素实现了 accept() 方法，在 accept()方法中调用访问者的访问方法以便完成对一个元素的操作。
 - **`ObjectStructure(对象结构)：`**对象结构是一个元素的集合，它用于存放元素对象，并且提供了遍历其内部元素的方法。它可以结合组合模式来实现，也可以是一个简单的集合对象，如一个 List 对象或一个 Set 对象。

### 结构图 ###
![访问者模式结构图](https://henleylee.github.io/medias/design_pattern/visitor_uml.jpg)

## 示例代码 ##
首先，是抽象访问者和具体访问者。Visitor 抽象类充当抽象访问者，ConcreteVisitor 类充当具体访问者，完整代码如下所示：
```java
/**
 * 抽象访问者
 */
abstract class Visitor {

    public abstract void visit(ConcreteElementA elementA);

    public abstract void visit(ConcreteElementB elementB);

}
```
> 在访问者模式中，抽象访问者定义了访问元素对象的方法，通常为每一种类型的元素对象都提供一个访问方法，而具体访问者可以实现这些访问方法。这些访问方法的命名一般有两种方式：一种是直接在方法名中标明待访问元素对象的具体类型，如 visitElementA(ElementA elementA)，还有一种是统一取名为 visit()，通过参数类型的不同来定义一系列重载的 visit()方法。当然，如果所有的访问者对某一类型的元素的访问操作都相同，则可以将操作代码移到抽象访问者类中。

```java
/**
 * 具体访问者
 */
public class ConcreteVisitor extends Visitor {

    public void visit(ConcreteElementA elementA) {
        System.out.println("元素ConcreteElementA的业务逻辑....");
    }

    public void visit(ConcreteElementB elementB) {
        System.out.println("元素ConcreteElementB的业务逻辑....");
    }

}
```
> 在这里使用了重载 visit() 方法的方式来定义多个方法用于操作不同类型的元素对象。在抽象访问者 Visitor 类的子类 ConcreteVisitor 中实现了抽象的访问方法，用于定义对不同类型元素对象的操作。

其次，是抽象元素和具体元素。Element 接口充当抽象访问者，ConcreteElementA 和 ConcreteElementB 类充当具体访问者，完整代码如下所示：
```java
/**
 * 抽象元素
 */
interface Element {
    void accept(Visitor visitor);
}
```
> 对于元素类而言，在其中一般都定义了一个 accept()方法，用于接受访问者的访问。需要注意的是该方法传入了一个抽象访问者 Visitor 类型的参数，即针对抽象访问者进行编程，而不是具体访问者，在程序运行时再确定具体访问者的类型，并调用具体访问者对象的 visit() 方法实现对元素对象的操作。

```java
/**
 * 具体元素A
 */
public class ConcreteElementA implements Element {


    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }

    public void operationA() {
        System.out.println("元素ConcreteElementA的业务逻辑....");
    }

}
```

```java
/**
 * 具体元素B
 */
public class ConcreteElementB implements Element {

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }

    public void operationB() {
        System.out.println("元素ConcreteElementB的业务逻辑....");
    }

}

```
> 在抽象元素类Element的子类中实现了accept()方法，用于接受访问者的访问，在具体元素类中还可以定义不同类型的元素所特有的业务方法。

然后，是对象结构。ObjectStructure 类充当对象结构，完整代码如下所示：
```java
public class ObjectStructure {

    private List<Element> elements = new ArrayList<Element>();

    public void action(Visitor vistor) {
        for (Element element : elements) {
            element.accept(vistor);
        }
    }

    public void add(Element node) {
        elements.add(node);
    }

}
```
> 对象结构是一个元素的集合，它用于存放元素对象，并且提供了遍历其内部元素的方法。

最后，是客户端场景类，完整代码如下所示：
```java
public class VisitorClient {

    public static void main(String[] args) {
        ObjectStructure structure = new ObjectStructure();
        structure.add(new ConcreteElementA());
        structure.add(new ConcreteElementB());

        Visitor visitor = new ConcreteVisitor();
        structure.action(visitor);

    }

}
```

## 模式分析 ##
访问者模式中对象结构存储了不同类型的元素对象，以供不同访问者访问。访问者模式包括两个层次结构：
 - 一个是访问者层次结构，提供了抽象访问者和具体访问者。
 - 一个是元素层次结构，提供了抽象元素和具体元素。

相同的访问者可以以不同的方式访问不同的元素，相同的元素可以接受不同访问者以不同访问方式访问。在访问者模式中，增加新的访问者无须修改原有系统，系统具有较好的可扩展性。

由于访问者模式的使用条件较为苛刻，本身结构也较为复杂，因此在实际应用中使用频率不是特别高。当系统中存在一个较为复杂的对象结构，且不同访问者对其所采取的操作也不相同时，可以考虑使用访问者模式进行设计。

### 优点 ###
访问者模式的主要优点如下：
 - 增加新的访问操作很方便。使用访问者模式，增加新的访问操作就意味着增加一个新的具体访问者类，实现简单，无须修改源代码，符合“开闭原则”。
 - 将有关元素对象的访问行为集中到一个访问者对象中，而不是分散在一个个的元素类中。类的职责更加清晰，有利于对象结构中元素对象的复用，相同的对象结构可以供多个不同的访问者访问。
 - 让用户能够在不修改现有元素类层次结构的情况下，定义作用于该层次结构的操作。

### 缺点 ###
访问者模式的主要缺点如下：
 - 增加新的元素类很困难。在访问者模式中，每增加一个新的元素类都意味着要在抽象访问者角色中增加一个新的抽象操作，并在每一个具体访问者类中增加相应的具体操作，这违背了“开闭原则”的要求。
 - 破坏封装。访问者模式要求访问者对象访问并调用每一个元素对象的操作，这意味着元素对象有时候必须暴露一些自己的内部操作和内部状态，否则无法供访问者访问。

### 适用环境 ###
在以下情况下可以使用访问者模式：
 - 一个对象结构包含多个类型的对象，希望对这些对象实施一些依赖其具体类型的操作。在访问者中针对每一种具体的类型都提供了一个访问操作，不同类型的对象可以有不同的访问操作。
 - 需要对一个对象结构中的对象进行很多不同的并且不相关的操作，而需要避免让这些操作“污染”这些对象的类，也不希望在增加新操作时修改这些类。访问者模式使得我们可以将相关的访问操作集中起来定义在访问者类中，对象结构可以被多个不同的访问者类所使用，将对象本身与对象的访问操作分离。
 - 对象结构中对象对应的类很少改变，但经常需要在此对象结构上定义新的操作。

### 模式拓展 ###
访问者模式与组合模式联用：在访问者模式中，包含一个用于存储元素对象集合的对象结构，我们通常可以使用迭代器来遍历对象结构，同时具体元素之间可以存在整体与部分关系，有些元素作为容器对象，有些元素作为成员对象，可以使用组合模式来组织元素。

## 总结 ##
 - 访问者模式提供一个作用于某对象结构中的各元素的操作表示，它使我们可以在不改变各元素的类的前提下定义作用于这些元素的新操作。访问者模式是一种对象行为型模式。
 - 访问者模式中对象结构存储了不同类型的元素对象，以供不同访问者访问。访问者模式包括两个层次结构，一个是访问者层次结构，提供了抽象访问者和具体访问者，一个是元素层次结构，提供了抽象元素和具体元素。相同的访问者可以以不同的方式访问不同的元素，相同的元素可以接受不同访问者以不同访问方式访问。
 - 访问者模式中，增加新的访问操作就意味着增加一个新的具体访问者类，实现简单，无须修改源代码，符合“开闭原则”。
 - 访问者模式将有关元素对象的访问行为集中到一个访问者对象中，而不是分散在一个个的元素类中。类的职责更加清晰，有利于对象结构中元素对象的复用，相同的对象结构可以供多个不同的访问者访问。
 - 访问者模式让用户能够在不修改现有元素类层次结构的情况下，定义作用于该层次结构的操作。

## 致谢 ##
[Java Design Pattern](https://www.gitbook.com/book/quanke/design-pattern-java/)

