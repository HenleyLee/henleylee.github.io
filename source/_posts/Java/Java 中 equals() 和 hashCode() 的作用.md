---
title: Java 中 equals() 和 hashCode() 的作用
categories: Java
tags:
  - Java
abbrlink: cc1a57ba
date: 2019-02-18 18:32:46
---

`java.lang.Object` 类中定义了 `equals()` 和 `hashCode()` 方法，`Object` 类是所有 Java 类的基类，所以所有的 Java 类都实现了这两个方法。

## equals() ##
`equals()` 方法的作用是**`用来判断两个对象是否相等`**。

`equals()` 方法定义在 `java.lang.Object` 类中。通过判断两个对象的地址是否相等(即，是否是同一个对象)来区分它们是否相等。源码如下：
```java
public boolean equals() {
    return (this == obj);
}
```

既然 `Object` 类中定义了 `equals()` 方法，这就意味着所有的 Java 类都实现了 equals() 方法，所有的类都可以通过 `equals()` 方法去比较两个对象是否相等。但是，使用默认的 `equals()` 方法，等价于 `==`。因此，通常需要重写 `equals()` 方法：若两个对象的内容相等，则 `equals()` 方法返回 `true`；否则，返回 `fasle`。  

`equals()` 方法的使用分为以下两种情况：
 - 若某个类没有重写 `equals()` 方法，当它通过 `equals()` 方法比较两个对象时，实际上是比较两个对象是不是同一个对象。这时，等价于通过 `==` 去比较这两个对象。
 - 可以重写类的 `equals()` 方法，来让 `equals()` 方法通过其它方式比较两个对象是否相等。通常的做法是：若两个对象的内容相等，则 `equals()` 方法返回 `true`；否则，返回 `fasle`。

> `==` 的作用是判断两个对象的地址是否相等(即，判断两个对象是不是同一个对象)；`equals()` 的作用是判断两个对象是否相等。

## hashCode() ##
`hashCode()` 方法的作用是**`获取哈希码`**，也称为散列码；它实际上是返回一个 int 值。这个哈希码的作用是确定该对象在哈希表中的索引位置。

`hashCode()` 方法定义在 `java.lang.Object` 类中，这就意味着所有的 Java 类都实现了 `hashCode()` 方法。

虽然，每个 Java 类都实现了 `hashCode()` 方法，但是，类的 `hashCode()` 方法一般仅用在散列表中(用于确定该类的每一个对象在散列表中的位置)。
> 散列表：Java 集合中本质是散列表的类，如 HashMap、Hashtable、HashSet 等。也就是说：hashCode() 方法在散列表中才有用，在其它情况下没用。在散列表中 hashCode() 方法的作用是获取对象的散列码，进而确定该对象在散列表中的位置。

## equals() 和 hashCode() 的关系 ##
Java 对于 `eqauls()` 方法和 `hashCode()` 方法是这样规定的： 
 - 同一对象上多次调用 hashCode() 方法，总是返回相同的整型值。
 - 如果 a.equals(b)，则一定有 a.hashCode() 一定等于 b.hashCode()。 
 - 如果 !a.equals(b)，则 a.hashCode() 不一定等于 b.hashCode()。此时如果 a.hashCode() 总是不等于 b.hashCode()，会提高 hashtables 的性能。
 - a.hashCode()==b.hashCode() 则 a.equals(b) 可真可假
 - a.hashCode()！= b.hashCode() 则 a.equals(b) 为假。 

上面结论简记：
 - 如果两个对象 equals，Java 运行时环境会认为他们的 hashCode 一定相等。 
 - 如果两个对象不 equals，他们的 hashCode 有可能相等。 
 - 如果两个对象 hashCode 相等，他们不一定 equals。 
 - 如果两个对象 hashCode 不相等，他们一定不 equals。 

## 关于 equals() 和 hashCode() 的重要规范 ##
 - 规范1：若重写 `equals()` 方法，有必要重写 `hashcode()` 方法，确保通过 `equals()` 方法判断结果为 `true` 的两个对象具备相等的 `hashcode()` 方法返回值。说得简单点就是：“如果两个对象相同，那么他们的 hashCode 应该相等”。不过请注意：这个只是规范，如果非要写一个类让 `equals()` 方法返回 `true` 而 `hashCode()` 方法返回两个不相等的值，编译和运行都是不会报错的。不过这样违反了 Java 规范，程序也就埋下了 BUG。 
 - 规范2：如果 `equals()` 方法返回 `false`，即两个对象“不相同”，并不要求对这两个对象调用 `hashCode()` 方法得到两个不相同的数。说的简单点就是：“如果两个对象不相同，他们的 hashCode 可能相同”。 

## 为什么重写 equals() 时总要重写 hashCode() ## 
一个很常见的错误根源在于没有重写 hashCode() 方法。在每个重写了 `equals()` 方法的类中，也必须重写 `hashCode()` 方法。如果不这样做的话，就会违反 Object.hashCode 的通用约定，从而导致该类无法结合所有基于散列的集合一起正常运作，这样的集合包括 HashMap、HashSet 和 Hashtable。
 - 在应用程序的执行期间，只要对象的 `equals()` 方法的比较操作所用到的信息没有被修改，那么对这同一个对象调用多次，`hashCode()` 方法都必须始终如一地返回同一个整数。在同一个应用程序的多次执行过程中，每次执行所返回的整数可以不一致。
 - 如果两个对象根据 `equals()` 方法比较是相等的，那么调用这两个对象中任意一个对象的 `hashCode()` 方法都必须产生同样的整数结果。
 - 如果两个对象根据 `equals()` 方法比较是不相等的，那么调用这两个对象中任意一个对象的 `hashCode()` 方法，则不一定要产生相同的整数结果。但是程序员应该知道，给不相等的对象产生截然不同的整数结果，有可能提高散列表的性能。

## 总结 ## 
 - `equals()` 方法用于比较对象的内容是否相等(重写以后)；
 - `hashCode()` 方法只有在集合中用到；
 - 当重写了 `equals()` 方法时，比较对象是否相等将通过重写后的 `equals()` 方法进行比较(判断对象的内容是否相等)；
 - 将对象放入到集合中时，首先判断要放入对象的 `hashCode()` 值与集合中的任意一个元素的 `hashCode()` 值是否相等，如果不相等直接将该对象放入集合中。如果 `hashCode()` 值相等，然后再通过 `equals()` 方法判断要放入对象与集合中的任意一个对象是否相等，如果 `equals()` 判断不相等，直接将该元素放入到集合中，否则不放入。

