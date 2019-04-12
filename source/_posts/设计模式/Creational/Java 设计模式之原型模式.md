---
title: Java 设计模式之原型模式
categories: 设计模式
tags:
  - Java
  - 设计模式
abbrlink: 246f0bcd
date: 2018-09-15 12:55:40
---

## 模式定义 ##
**`原型模式(Prototype Pattern)`**使用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

原型模式的工作原理很简单：将一个原型对象传给那个要发动创建的对象，这个要发动创建的对象通过请求原型对象拷贝自己来实现创建过程。由于在软件系统中我们经常会遇到需要创建多个相同或者相似对象的情况，因此原型模式在真实开发中的使用频率还是非常高的。原型模式是一种“另类”的创建型模式，创建克隆对象的工厂就是原型类自身，工厂方法由克隆方法来实现。

需要注意的是通过克隆方法所创建的对象是全新的对象，它们在内存中拥有新的地址，通常对克隆所产生的对象进行修改对原型对象不会造成任何影响，每一个克隆对象都是相互独立的。通过不同的方式修改可以得到一系列相似但不完全相同的对象。

> 原型模式属于**`对象创建型模式`**。

## 模式结构 ##
### 角色组成 ###
原型模式包含如下角色：
 - **`Prototype(抽象原型类)：`**它是声明克隆方法的接口，是所有具体原型类的公共父类，可以是抽象类也可以是接口，甚至还可以是具体实现类。
 - **`ConcretePrototype(具体原型类)：`**它实现在抽象原型类中声明的克隆方法，在克隆方法中返回自己的一个克隆对象。
 - **`Client(客户端)：`**让一个原型对象克隆自身从而创建一个新的对象，在客户类中只需要直接实例化或通过工厂方法等方式创建一个原型对象，再通过调用该对象的克隆方法即可得到多个相同的对象。由于客户类针对抽象原型类 Prototype 编程，因此用户可以根据需要选择具体原型类，系统具有较好的可扩展性，增加或更换具体原型类都很方便。

### 结构图 ###
![原型模式结构图](https://henleylee.github.io/medias/design_pattern/prototype_uml.jpg)

### 时序图 ###
![原型模式时序图](https://henleylee.github.io/medias/design_pattern/prototype_seq.jpg)

## 模式实现 ##
首先，是抽象原型类。Object 类充当抽象原型类。

学过 Java 语言的人都知道，所有的 Java 类都继承自 `java.lang.Object`。事实上，`Object` 类提供一个 `clone()` 方法，可以将一个 Java 对象复制一份。因此在 Java 中可以直接使用 `Object` 提供的 `clone()` 方法来实现对象的克隆，Java 语言中的原型模式实现很简单。

> 需要注意的是能够实现克隆的 Java 类必须实现一个标识接口 `Cloneable`，表示这个 Java 类支持被复制。如果一个类没有实现这个接口但是调用了 `clone()` 方法，Java 编译器将抛出一个 `CloneNotSupportedException` 异常。

然后，是具体原型类。WeeklyLog 充当具体原型类，完整代码如下所示：
```java
/**
 * 具体原型类
 */
class WeeklyLog implements Cloneable {

    private String name;
    private String date;
    private String content;

    public void setName(String name) {
        this.name = name;
    }

    public void setDate(String date) {
        this.date = date;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public String getName() {
        return this.name;
    }

    public String getDate() {
        return this.date;
    }

    public String getContent() {
        return this.content;
    }

    // 克隆方法clone()，此处使用Java语言提供的克隆机制
    public WeeklyLog clone() {
        Object obj = null;
        try {
            obj = super.clone();
            return (WeeklyLog) obj;
        } catch (CloneNotSupportedException e) {
            System.out.println("不支持复制！");
            return null;
        }
    }

}
```

最后，是客户端场景类，完整代码如下所示：
```java
/**
 * 原型模式的客户端场景类
 */
public class PrototypeClient {

    public static void main(String[] args) throws CloneNotSupportedException {
        WeeklyLog weeklyLog1 = new WeeklyLog();  //创建原型对象
        weeklyLog1.setName("张无忌");
        weeklyLog1.setDate("第1周");
        weeklyLog1.setContent("这周工作很忙，每天加班！");

        System.out.println("****周报****");
        System.out.println("周次：" + weeklyLog1.getDate());
        System.out.println("姓名：" + weeklyLog1.getName());
        System.out.println("内容：" + weeklyLog1.getContent());
        System.out.println("--------------------------------");

        WeeklyLog weeklyLog2 = weeklyLog1.clone(); // 调用克隆方法创建克隆对象
        weeklyLog2.setDate("第2周");
        System.out.println("****周报****");
        System.out.println("周次：" + weeklyLog2.getDate());
        System.out.println("姓名：" + weeklyLog2.getName());
        System.out.println("内容：" + weeklyLog2.getContent());
    }

}
```

## 浅克隆和深克隆 ##
克隆方法有两种：
 - 浅克隆(ShallowClone)
 - 深克隆(DeepClone)

在 Java 语言中，数据类型分为值类型（基本数据类型）和引用类型，值类型包括 `int`、`double`、`byte`、`boolean`、`char` 等简单数据类型，引用类型包括类、接口、数组等复杂类型。浅克隆和深克隆的主要区别在于`是否支持引用类型的成员变量的复制`，下面将对两者进行详细介绍。

### 浅克隆 ###
在浅克隆中，如果原型对象的成员变量是值类型，将复制一份给克隆对象；如果原型对象的成员变量是引用类型，则将引用对象的地址复制一份给克隆对象，也就是说原型对象和克隆对象的成员变量指向相同的内存地址。简单来说，在浅克隆中，当对象被复制时只复制它本身和其中包含的值类型的成员变量，而引用类型的成员对象并没有复制。下图为浅克隆示意图：
![浅克隆示意图](https://henleylee.github.io/medias/design_pattern/clone_shallow.jpg)

> 在 Java 语言中，通过覆盖 `Object` 类的 `clone()` 方法可以实现浅克隆。

### 深克隆 ###
在深克隆中，无论原型对象的成员变量是值类型还是引用类型，都将复制一份给克隆对象，深克隆将原型对象的所有引用对象也复制一份给克隆对象。简单来说，在深克隆中，除了对象本身被复制外，对象所包含的所有成员变量也将复制。下图为深克隆示意图：
![深克隆示意图](https://henleylee.github.io/medias/design_pattern/clone_deep.jpg)

> 在 Java 语言中，如果需要实现深克隆，可以通过`序列化(Serialization)`等方式来实现。序列化就是将对象写到流的过程，写到流中的对象是原有对象的一个拷贝，而原对象仍然存在于内存中。通过序列化实现的拷贝不仅可以复制对象本身，而且可以复制其引用的成员对象，因此通过序列化将对象写到一个流中，再从流里将其读出来，可以实现深克隆。

## 模式分析 ##
原型模式作为一种快速创建大量相同或相似对象的方式，在软件开发中应用较为广泛，很多软件提供的`复制(Ctrl + C)`和`粘贴(Ctrl + V)`操作就是原型模式的典型应用，下面对该模式的使用效果和适用情况进行简单的总结。

### 优点 ###
原型模式的主要优点如下：
 - 当创建新的对象实例较为复杂时，使用原型模式可以简化对象的创建过程，通过复制一个已有实例可以提高新实例的创建效率。
 - 扩展性较好，由于在原型模式中提供了抽象原型类，在客户端可以针对抽象原型类进行编程，而将具体原型类写在配置文件中，增加或减少产品类对原有系统都没有任何影响。
 - 原型模式提供了简化的创建结构，工厂方法模式常常需要有一个与产品类等级结构相同的工厂等级结构，而原型模式就不需要这样，原型模式中产品的复制是通过封装在原型类中的克隆方法实现的，无须专门的工厂类来创建产品。
 - 可以使用深克隆的方式保存对象的状态，使用原型模式将对象复制一份并将其状态保存起来，以便在需要的时候使用(如恢复到某一历史状态)，可辅助实现撤销操作。

### 缺点 ###
原型模式的主要缺点如下：
 - 需要为每一个类配备一个克隆方法，而且该克隆方法位于一个类的内部，当对已有的类进行改造时，需要修改源代码，违背了“开闭原则”。
 - 在实现深克隆时需要编写较为复杂的代码，而且当对象之间存在多重的嵌套引用时，为了实现深克隆，每一层对象对应的类都必须支持深克隆，实现起来可能会比较麻烦。

### 适用环境 ###
在以下情况下可以考虑使用原型模式：
 - 创建新对象成本较大(如初始化需要占用较长的时间，占用太多的CPU资源或网络资源)，新的对象可以通过原型模式对已有对象进行复制来获得，如果是相似对象，则可以对其成员变量稍作修改。
 - 如果系统要保存对象的状态，而对象的状态变化很小，或者对象本身占用内存较少时，可以使用原型模式配合备忘录模式来实现。
 - 需要避免使用分层次的工厂类来创建分层次的对象，并且类的实例对象只有一个或很少的几个组合状态，通过复制原型对象得到新实例可能比使用构造函数创建一个新实例更加方便。

### 拓展 ###
Java 语言提供的 `Cloneable` 接口和 `Serializable` 接口的代码非常简单，它们都是空接口，这种空接口也称为标识接口，标识接口中没有任何方法的定义，其作用是告诉 `JRE` 这些接口的实现类是否具有某个功能，如是否支持克隆、是否支持序列化等。

## 模式总结 ##
 - 原型模式使用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。
 - 原型模式的工作原理是将一个原型对象传给那个要发动创建的对象，这个要发动创建的对象通过请求原型对象拷贝自己来实现创建过程。
 - 原型模式通过克隆方法所创建的对象是全新的对象，它们在内存中拥有新的地址，通常对克隆所产生的对象进行修改对原型对象不会造成任何影响，每一个克隆对象都是相互独立的。
 - 原型模式可以简化对象的创建过程，通过复制一个已有实例可以提高新实例的创建效率。
 - 建造者模式适用情况包括：创建新对象成本较大；需要生成的产品对象的属性相互依赖，需要指定其生成顺序；系统要保存对象的状态，而对象的状态变化很小，或者对象本身占用内存较少时；需要避免使用分层次的工厂类来创建分层次的对象，并且类的实例对象只有一个或很少的几个组合状态。

## 致谢 ##
[Java Design Pattern](https://www.gitbook.com/book/quanke/design-pattern-java/)
