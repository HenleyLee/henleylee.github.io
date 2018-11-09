---
title: Java 设计模式之抽象工厂模式
date: 2018-09-11 18:25:50
categories: 设计模式
tags:
  - Java
  - 设计模式
---

## 模式定义 ##
**`抽象工厂模式(Abstract Factory Pattern)`**是围绕一个超级工厂创建其他工厂。该超级工厂又称为其他工厂的工厂，它提供了一种创建对象的最佳方式。

在抽象工厂模式中，接口是负责创建一个相关对象的工厂，不需要显式指定它们的类。每个生成的工厂都能按照工厂模式提供对象。

> 抽象工厂模式属于**`对象创建型模式`**。

## 模式结构 ##
### 参与角色 ###
抽象工厂模式包含如下角色：
 - **`AbstractProduct(抽象产品)：`**它为每种产品声明接口，在抽象产品中声明了产品所具有的业务方法。
 - **`ConcreteProduct(具体产品)：`**它定义具体工厂生产的具体产品对象，实现抽象产品接口中声明的业务方法。
 - **`AbstractFactory(抽象工厂)：`**它声明了一组用于创建一族产品的方法，每一个方法对应一种产品。
 - **`ConcreteFactory(具体工厂)：`**它实现了在抽象工厂中声明的创建产品的方法，生成一组具体产品，这些产品构成了一个产品族，每一个产品都位于某个产品等级结构中。

### 结构图 ###
![抽象工厂模式结构图](https://lyl873825813.github.io/medias/design_pattern/abatractfactory_uml.jpg)

### 时序图 ###
![抽象工厂模式时序图](https://lyl873825813.github.io/medias/design_pattern/abatractfactory_seq.jpg)

## 模式实现 ##
首先，是抽象的产品类和具体的产品类。接口 Button、ComboBox 和 TextField 充当抽象产品，其子类 SpringButton、SpringComboBox、SpringTextField 和 SummerButton、SummerComboBox、SummerTextField 充当具体产品。完整代码如下所示：
```java
/**
 * 按钮接口：抽象产品
 */
interface Button {
    public void display();
}

/**
 * Spring按钮类：具体产品
 */
class SpringButton implements Button {
    public void display() {
        System.out.println("显示浅绿色按钮。");
    }
}

/**
 * Summer按钮类：具体产品
 */
class SummerButton implements Button {
    public void display() {
        System.out.println("显示浅蓝色按钮。");
    }
}
```

```java
/**
 * 组合框接口：抽象产品
 */
interface ComboBox {
    public void display();
}

/**
 * Spring组合框类：具体产品
 */
class SpringComboBox implements ComboBox {
    public void display() {
        System.out.println("显示绿色边框组合框。");
    }
}

/**
 * Summer组合框类：具体产品
 */
class SummerComboBox implements ComboBox {
    public void display() {
        System.out.println("显示蓝色边框组合框。");
    }
}
```

```java
/**
 * 文本框接口：抽象产品
 */
interface TextField {
    public void display();
}

/**
 * Spring文本框类：具体产品
 */
class SpringTextField implements TextField {
    public void display() {
        System.out.println("显示绿色边框文本框。");
    }
}

/**
 * Summer文本框类：具体产品
 */
class SummerTextField implements TextField {
    public void display() {
        System.out.println("显示蓝色边框文本框。");
    }
}
```

其次，是抽象的工厂类和具体的工厂类。SkinFactory 接口充当抽象工厂，其子类 SpringSkinFactory 和 SummerSkinFactory 充当具体工厂，完整代码如下所示：
```java
/**
 * 界面皮肤工厂接口：抽象工厂
 */
interface SkinFactory {
    public Button createButton();

    public ComboBox createComboBox();

    public TextField createTextField();
}
```

```java
/**
 * Spring皮肤工厂：具体工厂
 */
class SpringSkinFactory implements SkinFactory {
    public Button createButton() {
        return new SpringButton();
    }

    public ComboBox createComboBox() {
        return new SpringComboBox();
    }

    public TextField createTextField() {
        return new SpringTextField();
    }
}
```

```java
/**
 * Summer皮肤工厂：具体工厂
 */
class SummerSkinFactory implements SkinFactory {
    public Button createButton() {
        return new SummerButton();
    }

    public ComboBox createComboBox() {
        return new SummerComboBox();
    }

    public TextField createTextField() {
        return new SummerTextField();
    }
}
```

然后，是工厂创造器/生成器类。FactoryProducer 类充当工厂创造器/生成器类，完整代码如下所示：
```java
/**
 * 工厂创造器/生成器类
 */
public class FactoryProducer {
    public static SkinFactory getFactory(String choice) {
        if (choice.equalsIgnoreCase("Spring")) {
            return new SpringSkinFactory();
        } else if (choice.equalsIgnoreCase("Summer")) {
            return new SummerSkinFactory();
        }
        return null;
    }
}
```

最后，是客户端场景类，完整代码如下所示：
```java
/**
 * 抽象工厂模式客户端场景类
 */
public class AbstractFactoryClient {

    public static void main(String args[]) {
        SkinFactory factory = FactoryProducer.getFactory("Spring");
        if (factory != null) {
            Button button = factory.createButton();
            ComboBox comboBox = factory.createComboBox();
            TextField textField = factory.createTextField();

            button.display();
            comboBox.display();
            textField.display();
        }
    }

}
```

## 模式分析 ##
抽象工厂模式是工厂方法模式的进一步延伸，由于它提供了功能更为强大的工厂类并且具备较好的可扩展性，在软件开发中得以广泛应用，尤其是在一些框架和 API 类库的设计中，例如在 Java 语言的 AWT(抽象窗口工具包)中就使用了抽象工厂模式，它使用抽象工厂模式来实现在不同的操作系统中应用程序呈现与所在操作系统一致的外观界面。抽象工厂模式也是在软件开发中最常用的设计模式之一。

### 优点 ###
抽象工厂模式的主要优点如下：
 - 抽象工厂模式隔离了具体类的生成，使得客户并不需要知道什么被创建。由于这种隔离，更换一个具体工厂就变得相对容易，所有的具体工厂都实现了抽象工厂中定义的那些公共接口，因此只需改变具体工厂的实例，就可以在某种程度上改变整个软件系统的行为。
 - 当一个产品族中的多个对象被设计成一起工作时，它能够保证客户端始终只使用同一个产品族中的对象。
 - 增加新的产品族很方便，无须修改已有系统，符合“开闭原则”。

### 缺点 ###
抽象工厂模式的主要缺点如下：
 - 增加新的产品等级结构麻烦，需要对原有系统进行较大的修改，甚至需要修改抽象层代码，这显然会带来较大的不便，违背了“开闭原则”。

### 适用环境 ###
在以下情况下可以考虑使用抽象工厂模式：
 - 一个系统不应当依赖于产品类实例如何被创建、组合和表达的细节，这对于所有类型的工厂模式都是很重要的，用户无须关心对象的创建过程，将对象的创建和使用解耦。
 - 系统中有多于一个的产品族，而每次只使用其中某一产品族。可以通过配置文件等方式来使得用户可以动态改变产品族，也可以很方便地增加新的产品族。
 - 属于同一个产品族的产品将在一起使用，这一约束必须在系统的设计中体现出来。同一个产品族中的产品可以是没有任何关系的对象，但是它们都具有一些共同的约束，如同一操作系统下的按钮和文本框，按钮与文本框之间没有直接关系，但它们都是属于某一操作系统的，此时具有一个共同的约束条件：操作系统的类型。
 - 产品等级结构稳定，设计完成之后，不会向系统中增加新的产品等级结构或者删除已有的产品等级结构。

## 模式总结 ##
 - 抽象工厂模式是围绕一个超级工厂创建其他工厂，它属于类创建型模式。在抽象工厂模式中，接口是负责创建一个相关对象的工厂，不需要显式指定它们的类。每个生成的工厂都能按照工厂模式提供对象。
 - 抽象工厂模式是工厂方法模式的进一步延伸，由于它提供了功能更为强大的工厂类并且具备较好的可扩展性，在软件开发中得以广泛应用，尤其是在一些框架和 API 类库的设计中。
 - 抽象工厂模式隔离了具体类的生成，使得客户并不需要知道什么被创建。由于这种隔离，更换一个具体工厂就变得相对容易，所有的具体工厂都实现了抽象工厂中定义的那些公共接口，因此只需改变具体工厂的实例，就可以在某种程度上改变整个软件系统的行为。
 - 抽象工厂模式增加新的产品族很方便，无须修改已有系统，符合“开闭原则”。。
 - 抽象工厂模式适用情况包括：系统中有多于一个的产品族，而每次只使用其中某一产品族。可以通过配置文件等方式来使得用户可以动态改变产品族，也可以很方便地增加新的产品族。

## 致谢 ##
[Java Design Pattern](https://www.gitbook.com/book/quanke/design-pattern-java/)

