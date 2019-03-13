---
title: Java 中 String、StringBuilder 和 StringBuffer 的区别
categories: Java
tags:
  - Java
abbrlink: c54ca406
date: 2019-02-12 18:32:28
---

Java 中 `String`、`StringBuilder`、`StringBuffer` 是编程中经常使用的字符串类，他们之间的区别也是经常在面试中会问到的问题。下面就`运算速度`(运算性能或执行效率)和`线程安全性`两个方法来看看三者的区别。

## 区别 ##
### 运算速度 ###
`String` 是 `final` 类不能被继承且为`字符串常量`，而 `StringBuilder` 和 `StringBuffer` 均为`字符串变量`。String 对象一旦创建便不可更改，而后两者是可更改的，它们只能通过构造函数来建立对象，且对象被建立以后将在内存中分配内存空间，并初始保存一个 `null`，通过 `append()` 方法向 `StringBuffer` 和 `StringBuilder` 中赋值。
> String 类中使用字符数组保存字符串；StringBuilder 和 StringBuffer 都继承自 AbstractStringBuilder 类，在 AbstractStringBuilder 中也是使用字符数组保存字符串。

请看如下示例代码：
```java
String str = "abc";
System.out.println(str);
str = str + "de";
System.out.println(str);
```

上述代码先创建一个 String 对象 str，并赋值 abc 给 str，然后运行到第三行，JVM 会再创建一个新的 str 对象，并将原有 str 的值和 de 加起来再赋值给新的 str。而第一个创建的 str 对象被 JVM 的垃圾回收机制(GC)回收掉。所以 str 实际上并没有被更改，即 String 对象一旦创建就不可更改。
> 因此，Java 中对 String 对象进行的操作实际上是一个不断创建并回收对象的过程，因此在运行速度上很慢。而 StringBuilder 和 StringBuffer 的对象是变量，对变量的操作是直接对该对象就行更改，因此不会进行反复的创建和回收。所以在运行速度上比较快。

在某些特殊的情况下，String 的效率比 StringBuilder 和 StringBuffer 要快。请看如下示例代码：

```java
String s1 = "123" + "456";
StringBuilder s2 = new StringBuilder().append("123").append("456");
System.out.println(s1);
System.out.println(s2.toString());
```

上述代码中 String 的操作速度反而要比 StringBuilder 快，这是因为在 JVM 眼里，第1行的代码操作和下列代码是完全一样的，所以很快。

```java
String s1 = "123456";
```

但如下的代码写法形式速度会很慢，JVM会不断地创建和回收对象来进行操作。

```java
String str1 = "123";
String str2 = "456";
String str = str1 + str2;
```

> 总结：三者在运行速度方面快慢顺序为：StringBuilder > StringBuffer > String。

### 线程安全性 ###
#### String(线程安全) ####
**`String`**中的对象是不可变的，也就可以理解为常量，显然**`线程安全`**。

#### StringBuffer(线程安全) ####
**`StringBuffer`**中的大部分方法由 `synchronized` 关键字修饰，在必要时可对方法进行同步，因此任意特定实例上的所有操作就好像是以串行顺序发生的，该顺序与所涉及的每个线程进行的方法调用顺序一致，所以是**`线程安全`**的。

#### StringBuilder(非线程安全) ####
**`StringBuilder`**中的方法没有 `synchronized` 关键字修饰，不能保证线程安全性，所以是**`非线程安全`**的。StringBuilder 是 JDK1.5 新增的，该类提供一个与 StringBuffer 兼容的 API，但不能保证同步，所以在性能上较高。该类被设计用作 StringBuffer 的一个简易替换，用在字符串缓冲区被单个线程使用的时候(这种情况很普遍)。如果可能，建议优先采用该类，因为在大多数实现中，它比 StringBuffer 要快，而且两者的方法基本相同。

## 总结 ##

| 类            | 描述       | 线程安全性 | 运算速度 | 引进版本  |
|---------------|------------|------------|----------|-----------|
| String        | 字符串常量 | 线程安全   | 最慢     | JDK 1.0   |
| StringBuffer  | 字符串变量 | 线程安全   | 慢       | JDK 1.0   |
| StringBuilder | 字符串变量 | 非线程安全 | 快       | JDK 1.5   |

 - **`String：`**适用于少量的字符串操作的情况；
 - **`StringBuffer：`**适用多线程下在字符缓冲区进行大量操作的情况；
 - **`StringBuilder：`**适用于单线程下在字符缓冲区进行大量操作的情况。

> 在编译期就能够确定的字符串常量，没有必要创建 String、StringBuilder 或 StringBuffer 对象。直接使用字符串常量的**`+`**连接操作效率最高。
