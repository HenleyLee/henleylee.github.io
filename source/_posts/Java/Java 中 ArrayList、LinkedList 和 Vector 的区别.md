---
title: Java 中 ArrayList、LinkedList 和 Vector 的区别
categories: Java
tags:
  - Java
abbrlink: 4eec1027
date: 2019-02-16 10:22:15
---

Java 中 `ArrayList`、`LinkedList`、`Vector` 是编程中经常使用的集合类，他们之间的区别也是经常在面试中会问到的问题。下面就`底层实现`、`执行效率`和`线程安全性`两个方法来看看三者的区别。

## 区别 ##
### 底层实现 ###
实现接口：
 - **`ArrayList`**和**`Vector`**实现了 `java.util.RandomAccess` 接口；
 - **`LinkedList`**实现了 `java.util.Deque` 接口。

底层实现：
 - **`ArrayList`**和**`Vector`**的底层是基于**`动态数组`**实现的；
 - **`LinkedList`**的底层是基于**`双向链表`**实现的。

### 执行效率 ###
#### ArrayList ####
**`ArrayList`**的底层是基于`动态数组`实现的，所以 `ArrayList` 拥有着数组的特性。
 - 优点：根据下标随机访问数组元素的效率高，向数组尾部添加元素的效率高。
 - 缺点：删除数组中的数据以及向数组中间添加数据效率低，因为需要移动数组；当长度大于初始长度的时候，每添加一个数，都会需要扩容。

　　
#### LinkedList ####
**`LinkedList`**的底层是基于`双向链表`实现的，所以 `LinkedList` 拥有着链表的特性。
 - 优点：添加、删除元素的效率高，只需要改变指针指向即可。
 - 缺点：访问元素的平均效率低，需要对链表进行遍历。

#### Vector ####
**`Vector`**和 `ArrayList` 一样，也是通过数组实现的，不同的是它支持线程的同步，即某一时刻只有一个线程能够写 `Vector`，但实现同步需要很高的花费，因此，访问它比访问`ArrayList`慢。 

### 线程安全性 ###
在多线程并发的时候，`ArrayList` 和 `LinkedList` 是非线程安全的，并且是不同步的。`Vector` 的所有方法都用了 `synchronized` 关键字修饰，是线程安全的，但是 `Vector` 中的方法组合起来使用不是线程安全的。

> 由 `Vector` 创建的 `Iterator`，虽然和 `ArrayList` 创建的 `Iterator` 是同一接口，但是，因为 `Vector` 是同步的，当一个 `Iterator` 被创建而且正在被使用，另一个线程改变了 `Vector` 的状态(例如，添加、删除元素)，这时调用 `Iterator` 的方法时将抛出 `ConcurrentModificationException`，因此必须捕获该异常。

## 总结 ##

| 类         | 底层实现 | 线程安全性 | 执行效率                                                     | 引进版本  |
|------------|----------|------------|--------------------------------------------------------------|-----------|
| ArrayList  | 动态数组 | 非线程安全 | 随机访问、向尾部添加元素的效率高；删除、向中间添加元素效率低 | JDK 1.2   |
| LinkedList | 双向链表 | 非线程安全 | 添加、删除元素效率高；访问元素的平均效率低                   | JDK 1.2   |
| Vector     | 动态数组 | 线程安全   | 同 `ArrayList`，但是比 `ArrayList` 效率低                    | JDK 1.0   |

 - `ArrayList` 和 `Vector` 的实现是基于动态数组，`LinkedList` 的实现是基于双向链表。 
 - 对于随机访问，`ArrayList` 优于 `LinkedList`，`ArrayList` 可以根据下标以 O(1) 时间复杂度对元素进行随机访问。而 `LinkedList` 的每一个元素都依靠地址指针和它后一个元素连接在一起，在这种情况下，查找某个元素的时间复杂度是 O(n)。
 - 对于插入和删除操作，`LinkedList` 优于 `ArrayList`，因为当元素被添加到 `LinkedList` 任意位置的时候，不需要像 `ArrayList` 那样重新计算大小或者是更新索引。 
 - `LinkedList` 比 `ArrayList` 更占内存，因为 `LinkedList` 的节点除了存储数据，还存储了两个引用，一个指向前一个元素，一个指向后一个元素。
 - 在多线程并发的时候，`ArrayList` 和 `LinkedList` 是非线程安全的，并且是不同步的，`Vector` 是线程安全的，但是 `Vector` 中的方法组合起来使用不是线程安全的。