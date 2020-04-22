---
title: Java 集合之 ArrayList 源码分析
categories: Java
tags:
  - Java
abbrlink: bc5813b9
date: 2019-08-15 18:26:15
---

`ArrayList` 不是线程安全的，只能用在单线程环境下，多线程环境下可以考虑用 `Collections.synchronizedList(List<T> list)` 函数返回一个线程安全的 `List` 类，也可以使用 `java.util.concurrent` 并发包下的 `CopyOnWriteArrayList` 类。

## 简介 ##
`java.util.ArrayList` 实现了 `java.util.List` 接口，继承 `java.util.AbstractList`。其中 `List` 接口定义了一个有序集合的插入、查询等规则，而 `AbstractList` 类提供 `List` 接口的骨干实现。

`java.util.List` 是 Java 集合(Collection)中的一部分，是一个继承了 `java.util.Collection` 接口的接口，它有诸多特性，所以使用的场景会很多。下面先简单了解下它的特性：
 - 元素有序；
 - 元素可重复；
 - 每个元素都有自己的顺序索引。

`ArrayList` 有以下特点：
 - `ArrayList` 是一个继承于 `java.util.AbstractSequentialList` 的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。
 - `ArrayList` 实现 `java.util.List` 接口，能对它进行队列操作(添加、删除、修改、遍历等)。
 - `ArrayList` 实现了 `java.lang.Cloneable` 接口，即覆盖了 `clone()` 方法，能克隆。
 - `ArrayList` 实现 `java.io.Serializable` 接口，这意味着 `Vector` 支持序列化，能通过序列化去传输。
 - `ArrayList` 是非线程安全的。

`java.util.ArrayList` 实现了 `java.io.Serializable` 接口，因此它支持序列化，能够通过序列化传输，实现了 `java.lang.Cloneable` 接口，能被克隆。

`ArrayList` 提供了三个构造函数：
 - `ArrayList()：`构造一个具有默认初始容量(10)的空 ArrayList。
 - `ArrayList(int initialCapacity)：`构造一个带指定初始容量的空 ArrayList。
 - `ArrayList(Collection<? extends E> c)：`构造一个包含指定 Collection 的元素的 ArrayList。

> `ArrayList` 是 `List` 接口的`可变数组`的实现，实现了所有可选列表操作，并允许包括 `null` 在内的所有元素。除了实现 `List` 接口外，此类还提供一些方法来操作内部用来存储列表的数组的大小。

## 实现原理 ##
`ArrayList` 实现了 `List` 接口，底层使用数组保存所有元素，其操作基本上是对数组的操作。

### 私有属性 ###
`ArrayList` 类只定义了两个私有属性：
```java
/** 存储元素的数组缓冲区 */
transient Object[] elementData; // non-private to simplify nested class access

/** 包含的元素数量 */
private int size;
```

`elementData` 用到了 Java 的关键字 `transient`，Java 中的 `transient` 关键字是短暂的意思。对于 `transient` 修饰的成员变量，在类实例的序列化处理过程中会被忽略。因此，`transient` 变量不会贯穿对象的序列化和反序列化，生命周期仅存于调用者的内存中而不会写到磁盘里持久化。

> `ArrayList` 是**`基于数组实现的，是一个动态数组`**，其容量能自动增长，类似于 `C 语言` 中的动态申请内存，动态增长内存。

### set() ###
**`set(int index, E element)`**方法用于用指定的元素替代此列表中指定位置上的元素，并返回以前位于该位置上的元素。下面分析 `set(int index, E element)` 方法的源码： 
```java
public E set(int index, E element) {
    // 检查指定的索引是否在范围内
    rangeCheck(index);
    // 获取指定索引的元素
    E oldValue = elementData(index);
    // 用指定的元素替代此列表中指定位置上的元素
    elementData[index] = element;
    // 返回指定位置的元素
    return oldValue;
}
```

### get() ###
**`get(int index)`**方法用于获取列表中指定位置上的元素。下面分析 `get(int index)` 方法的源码： 
```java
public E get(int index) {
    // 检查指定的索引是否在范围内
    rangeCheck(index);
    // 返回指定索引的元素
    return elementData(index);
}
```

### add() ###
**`add(E e)`**方法用于将指定的元素添加到此列表的尾部。下面分析 `add(E e)` 方法的源码： 
```java
public boolean add(E e) {
    // 如果数组长度不足，将进行扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 将指定元素添加列表的尾部
    elementData[size++] = e;
    return true;
}
```

**`add(int index, E element)`**方法用于将指定的元素插入此列表中的指定位置。下面分析 `add(int index, E element)` 方法的源码： 
```java
public void add(int index, E element) {
    // 检查指定的索引是否在范围内
    rangeCheckForAdd(index);
    // 如果数组长度不足，将进行扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 将当前位于该位置的元素以及所有后续元素右移一个位置
    System.arraycopy(elementData, index, elementData, index + 1,
            size - index);
    // 将指定的元素插入列表中的指定位置
    elementData[index] = element;
    size++;
}
```

### addAll() ###
**`addAll(Collection<? extends E> c)`**方法用于将指定 `Collection` 中的所有元素按顺序添加到此列表的尾部。下面分析 `addAll(Collection<? extends E> c)` 方法的源码： 
```java
public boolean addAll(Collection<? extends E> c) {
    // 将指定集合转换为数组
    Object[] a = c.toArray();
    // 获取转换后的数组长度
    int numNew = a.length;
    // 如果数组长度不足，将进行扩容
    ensureCapacityInternal(size + numNew);  // Increments modCount
    // 将转换后的数组复制到列表的尾部
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}
```

**`addAll(int index, Collection<? extends E> c)`**方法用于将指定 `Collection` 中的所有元素按顺序从指定的位置开始添加到此列表中。下面分析 `addAll(int index, Collection<? extends E> c)` 方法的源码： 
```java
public boolean addAll(int index, Collection<? extends E> c) {
    // 检查指定的索引是否在范围内
    rangeCheckForAdd(index);
    // 将指定集合转换为数组
    Object[] a = c.toArray();
    // 获取转换后的数组长度
    int numNew = a.length;
    // 如果数组长度不足，将进行扩容
    ensureCapacityInternal(size + numNew);  // Increments modCount
    // 需要移动的元素数量
    int numMoved = size - index;
    if (numMoved > 0)
        // 将当前位于该位置的元素以及所有后续元素右移一个位置
        System.arraycopy(elementData, index, elementData, index + numNew,
                numMoved);
    // 将转换后的数组复制到列表中的指定位置
    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```

### remove() ###
**`remove(int index)`**方法用于移除此列表中指定位置上的元素。下面分析 `remove(int index)` 方法的源码： 
```java
public E remove(int index) {
    // 检查指定的索引是否在范围内
    rangeCheck(index);
    modCount++;
    // 获取指定索引的元素
    E oldValue = elementData(index);
    // 需要移动的元素数量
    int numMoved = size - index - 1;
    if (numMoved > 0)
        // 将当前位于该位置的元素的所有后续元素左移一个位置
        System.arraycopy(elementData, index + 1, elementData, index,
                numMoved);
    // 将列表中最后一个元素置为null
    elementData[--size] = null; // clear to let GC do its work
    return oldValue;
}
```

**`remove(Object o)`**方法用于移除此列表中首次出现的指定元素(如果存在)。下面分析 `remove(Object o)` 方法的源码： 
```java
public boolean remove(Object o) {
    // 判断需要被移除的元素是否为null
    if (o == null) {
        // 遍历列表中的元素
        for (int index = 0; index < size; index++)
            // 判断当前索引位置的元素是否为null
            if (elementData[index] == null) {
                // 快速移除该索引位置的元素(跳过边界检查)
                fastRemove(index);
                return true;
            }
    } else {
        // 遍历列表中的元素
        for (int index = 0; index < size; index++)
            // 判断当前索引位置的元素是否为需要被移除的元素
            if (o.equals(elementData[index])) {
                // 快速移除该索引位置的元素(跳过边界检查)
                fastRemove(index);
                return true;
            }
    }
    return false;
}
```

> 从数组中移除元素的操作，也会导致被移除的元素以后的所有元素的向左移动一个位置。

### contains() ###
**`contains(Object o)`**方法用于判断列表中是否包含指定元素。下面分析 `contains(Object o)` 方法的源码： 
```java
public boolean contains(Object o) {
    // 判断指定元素在列表中首次出现的索引是否大于0
    return indexOf(o) >= 0;
}
```

### indexOf() ###
**`indexOf(Object o)`**方法用于返回指定元素在列表中首次出现的索引；如果列表不包含该元素，则返回-1。下面分析 `indexOf(Object o)` 方法的源码： 
```java
public int indexOf(Object o) {
    // 判断指定元素是否为null
    if (o == null) {
        // 遍历列表中的元素
        for (int i = 0; i < size; i++)
            // 判断当前索引位置的元素是否为null
            if (elementData[i] == null)
                // 返回当前索引
                return i;
    } else {
        // 遍历列表中的元素
        for (int i = 0; i < size; i++)
            // 判断当前索引位置的元素是否为指定元素
            if (o.equals(elementData[i]))
                // 返回当前索引
                return i;
    }
    return -1;
}
```

> 无论指定元素是否为 `null`，都需要遍历整个数组比较，所以效率是比较低的。

### ensureCapacity() ###
**`ensureCapacity(int minCapacity)`**方法用于当向列表中添加元素时，检查添加后元素的个数是否会超出当前数组的长度，如果超出，数组将会进行扩容，以满足添加数据的需求。下面分析 `ensureCapacity(int minCapacity)` 方法的源码： 
```java
public void ensureCapacity(int minCapacity) {
    // 最小扩容量
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;
    // 所需的最小容量大于最小扩容量
    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // overflow-conscious code
    // 所需的最小容量大于扩容前的数组长度
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    // 旧的容量为扩容前的数组长度
    int oldCapacity = elementData.length;
    // 得到新的容量
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 如果新的容量小于最小容量
    if (newCapacity - minCapacity < 0)
        // 取最小容量为新的容量
        newCapacity = minCapacity;
    // 如果新的容量大于数组的最大长度
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        // 如果所需最小容量大于数组的最大长度，则取Integer.MAX_VALUE，反之取数组的最大长度
        newCapacity = hugeCapacity(minCapacity);
    // 将数组中的元素全部复制新的数组中
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

> 数组进行扩容时，会将老数组中的元素重新拷贝一份到新的数组中，每次数组容量的增长的代价是很高的，因此在实际使用时，当我们可预知要保存的元素的多少时，要在构造 `ArrayList`实例时，就指定其容量，以避免数组扩容的发生。或者根据实际需求，通过调用 `ensureCapacity()` 方法来手动增加 `ArrayList` 实例的容量，以减少递增式再分配的数量。

### trimToSize() ###
**`trimToSize()`**方法用于将底层数组的容量调整为当前列表保存的实际元素的大小。下面分析 `trimToSize()` 方法的源码： 
```java
public void trimToSize() {
    modCount++;
    // 判断当前元素数量是否小于数组长度
    if (size < elementData.length) {
        // 将底层数组的容量调整为当前列表保存的实际元素的大小
        elementData = (size == 0)
                ? EMPTY_ELEMENTDATA
                : Arrays.copyOf(elementData, size);
    }
}
```

## Fail-Fast 机制 ##
`Fail-Fast` 机制，即快速失败机制，是 `Java` 集合中的一种错误检测机制。当在迭代集合的过程中该集合在结构上发生改变的时候，就有可能会发生 `Fail-Fast`，即抛出 `ConcurrentModificationException` 异常。`Fail-Fast` 机制并不保证在不同步的修改下一定会抛出异常，它只是尽最大努力去抛出，所以这种机制一般仅用于检测 bug。

`ArrayList` 采用了快速失败的机制，通过记录 `modCount` 参数来实现。在面对并发的修改时，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。

## 总结 ##
**`ArrayList`**的底层是基于`动态数组`实现的，所以 `ArrayList` 拥有着数组的特性：
 - 优点：根据下标随机访问数组元素的效率高，向数组尾部添加元素的效率高。
 - 缺点：删除数组中的数据以及向数组中间添加数据效率低，因为需要移动数组；当长度大于初始长度的时候，每添加一个数，都会需要扩容。

在初始化完成后如果想插入大量数据可以调用 `ensureCapacity()` 方法来提前对 `ArrayList` 进行扩容，而不是在不断插入数据的时候自身不断扩容，如果是数据频繁的增加删除 `LinkedList` 则是最佳选择。

