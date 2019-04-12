---
title: Java 设计模式之解释器模式
categories: 设计模式
tags:
  - Java
  - 设计模式
abbrlink: 4976d8cc
date: 2018-10-24 19:20:22
---

## 模式动机 ##
在某些情况下，为了更好地描述某一些特定类型的问题，我们可以创建一种新的语言，这种语言拥有自己的表达式和结构，即文法规则，这些问题的实例将对应为该语言中的句子。此时，可以使用解释器模式来设计这种新的语言。对解释器模式的学习能够加深我们对面向对象思想的理解，并且掌握编程语言中文法规则的解释过程。这就是解释器模式的动机。

## 模式定义 ##
**`解释器模式(Interpreter Pattern)`**定义一个语言的文法，并且建立一个解释器来解释该语言中的句子，这里的“语言”是指使用规定格式和语法的代码。

> 解释器模式是一种**`类行为型模式`**。

## 模式结构 ##
### 角色组成 ###
解释器模式包含如下角色：
 - **`AbstractExpression(抽象表达式)：`**在抽象表达式中声明了抽象的解释操作，它是所有终结符表达式和非终结符表达式的公共父类。
 - **`TerminalExpression(终结符表达式)：`**终结符表达式是抽象表达式的子类，它实现了与文法中的终结符相关联的解释操作，在句子中的每一个终结符都是该类的一个实例。通常在一个解释器模式中只有少数几个终结符表达式类，它们的实例可以通过非终结符表达式组成较为复杂的句子。
 - **`NonterminalExpression(非终结符表达式)：`**非终结符表达式也是抽象表达式的子类，它实现了文法中非终结符的解释操作，由于在非终结符表达式中可以包含终结符表达式，也可以继续包含非终结符表达式，因此其解释操作一般通过递归的方式来完成。
 - **`Context(环境类)：`**环境类又称为上下文类，它用于存储解释器之外的一些全局信息，通常它临时存储了需要解释的语句。

### 结构图 ###
![解释器模式结构图](https://henleylee.github.io/medias/design_pattern/interpreter_uml.jpg)

## 示例代码 ##
首先，是抽象表达式。AbstractExpression 抽象类充当抽象表达式，完整代码如下所示：
```java
/**
 * 抽象表达式
 */
abstract class AbstractExpression {

    public abstract int interpret(Context context);

}
```
> 在解释器模式中，每一种终结符和非终结符都有一个具体类与之对应，正因为使用类来表示每一条文法规则，所以系统将具有较好的灵活性和可扩展性。对于所有的终结符和非终结符，我们首先需要抽象出一个公共父类，即抽象表达式类。

其次，是终结符表达式和非终结符表达式。TerminalExpression 类充当终结符表达式，NonterminalExpression 类充当非终结符表达式，完整代码如下所示：
```java
/**
 * 终结符表达式
 */
class TerminalExpression extends AbstractExpression {

    public int interpret(Context context) {
        // 终结符表达式的解释操作
        return context.lookup(this);
    }

}
```

```java
/**
 * 非终结符表达式
 */
class NonterminalExpression extends AbstractExpression {

    private AbstractExpression left;
    private AbstractExpression right;

    public NonterminalExpression(AbstractExpression left, AbstractExpression right) {
        this.left = left;
        this.right = right;
    }

    public int interpret(Context context) {
        // 调用每一个组成部分的interpret()方法，在调用时指定组成部分的连接方式，即非终结符的功能
        return left.interpret(context) + right.interpret(context);
    }

}
```
> 终结符表达式和非终结符表达式类都是抽象表达式类的子类。对于终结符表达式，其代码很简单，主要是对终结符元素的处理；对于非终结符表达式，其代码相对比较复杂，因为可以通过非终结符将表达式组合成更加复杂的结构。

然后，是环境类。Context 类充当环境类，完整代码如下所示：
```java
/**
 * 环境类
 */
public class Context {

    private HashMap<AbstractExpression, Integer> values = new HashMap<AbstractExpression, Integer>();

    public void assign(AbstractExpression key, Integer value) {
        // 往环境类中设值
        values.put(key, value);
    }

    public Integer lookup(AbstractExpression key) {
        // 获取存储在环境类中的值
        return values.get(key);
    }

}
```
> 环境类用于存储一些全局信息，通常在 Context 中包含了一个 HashMap 或 ArrayList 等类型的集合对象（也可以直接由 HashMap 等集合类充当环境类），存储一系列公共信息，如变量名与值的映射关系（key/value）等，用于在进行具体的解释操作时从中获取相关信息。
> 环境类通常作为参数被传递到所有表达式的解释方法 interpret() 中，可以在 Context 对象中存储和访问表达式解释器的状态，向表达式解释器提供一些全局的、公共的数据，此外还可以在 Context 中增加一些所有表达式解释器都共有的功能，减轻解释器的职责。

最后，是客户端场景类，完整代码如下所示：
```java
public class InterpreterClient {

    public static void main(String[] args) {
        Context context = new Context();
        TerminalExpression expressionA = new TerminalExpression();
        TerminalExpression expressionB = new TerminalExpression();
        context.assign(expressionA, 100);
        context.assign(expressionB, 1000);

        AbstractExpression expression = new NonterminalExpression(expressionA, expressionB);
        System.out.println(expression.interpret(context));
    }

}
```

## 模式分析 ##
 - 由于表达式可分为终结符表达式和非终结符表达式，因此解释器模式的结构与组合模式的结构有些类似，但在解释器模式中包含更多的组成元素。
 - 解释器模式为自定义语言的设计和实现提供了一种解决方案，它用于定义一组文法规则并通过这组文法规则来解释语言中的句子。

### 优点 ###
解释器模式的主要优点如下：
 - 易于改变和扩展文法。由于在解释器模式中使用类来表示语言的文法规则，因此可以通过继承等机制来改变或扩展文法。
 - 每一条文法规则都可以表示为一个类，因此可以方便地实现一个简单的语言。
 - 实现文法较为容易。在抽象语法树中每一个表达式节点类的实现方式都是相似的，这些类的代码编写都不会特别复杂，还可以通过一些工具自动生成节点类代码。
 - 增加新的解释表达式较为方便。如果用户需要增加新的解释表达式只需要对应增加一个新的终结符表达式或非终结符表达式类，原有表达式类代码无须修改，符合“开闭原则”。

### 缺点 ###
解释器模式的主要缺点如下：
 - 对于复杂文法难以维护。在解释器模式中，每一条规则至少需要定义一个类，因此如果一个语言包含太多文法规则，类的个数将会急剧增加，导致系统难以管理和维护，此时可以考虑使用语法分析程序等方式来取代解释器模式。
 - 执行效率较低。由于在解释器模式中使用了大量的循环和递归调用，因此在解释较为复杂的句子时其速度很慢，而且代码的调试过程也比较麻烦。

### 适用环境 ###
在以下情况下可以使用解释器模式：
 - 可以将一个需要解释执行的语言中的句子表示为一个抽象语法树。
 - 一些重复出现的问题可以用一种简单的语言来进行表达。
 - 一个语言的文法较为简单。
 - 执行效率不是关键问题。

## 总结 ##
 - 解释器模式定义一个语言的文法，并且建立一个解释器来解释该语言中的句子，这里的“语言”是指使用规定格式和语法的代码。解释器模式是一种类行为型模式。
 - 在解释器模式中，每一种终结符和非终结符都有一个具体类与之对应，正因为使用类来表示每一条文法规则，所以系统将具有较好的灵活性和可扩展性。
 - 由于表达式可分为终结符表达式和非终结符表达式，因此解释器模式的结构与组合模式的结构有些类似，但在解释器模式中包含更多的组成元素。
 - 解释器模式为自定义语言的设计和实现提供了一种解决方案，它用于定义一组文法规则并通过这组文法规则来解释语言中的句子。
 - 易于改变和扩展文法。由于在解释器模式中使用类来表示语言的文法规则，因此可以通过继承等机制来改变或扩展文法。
 - 每一条文法规则都可以表示为一个类，因此可以方便地实现一个简单的语言。

## 致谢 ##
[Java Design Pattern](https://www.gitbook.com/book/quanke/design-pattern-java/)

