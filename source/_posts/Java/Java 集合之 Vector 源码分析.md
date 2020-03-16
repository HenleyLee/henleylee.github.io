---
title: Java 集合之 Vector 源码分析
categories: Java
tags:
  - Java
abbrlink: 83ddd470
date: 2019-08-16 19:06:25
---

`Vector` 和 `ArrayList` 有一些相似，其内部都是通过一个容量能够动态增长的数组来实现的。不同点是 `Vector` 是线程安全的，因为其内部有很多同步代码快来保证线程安全。

## 简介 ##
`java.util.Vector` 实现了 `java.util.List` 接口，继承 `java.util.AbstractList`。其中 `List` 接口定义了一个有序集合的插入、查询等规则，而 `AbstractList` 类提供 `List` 接口的骨干实现。

`java.util.List` 是 Java 集合(Collection)中的一部分，是一个继承了 `java.util.Collection` 接口的接口，它有诸多特性，所以使用的场景会很多。下面先简单了解下它的特性：
 - 元素有序；
 - 元素可重复；
 - 每个元素都有自己的顺序索引。

`Vector` 有以下特点：
 - `Vector` 是一个继承于 `java.util.AbstractSequentialList` 的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。
 - `Vector` 实现 `java.util.List` 接口，能对它进行队列操作(添加、删除、修改、遍历等)。
 - `Vector` 实现了 `java.lang.Cloneable` 接口，即覆盖了 `clone()` 方法，能克隆。
 - `Vector` 实现 `java.io.Serializable` 接口，这意味着 `Vector` 支持序列化，能通过序列化去传输。
 - `Vector` 是线程安全的。

`Vector` 提供了三个构造函数：
 - `Vector()：`构造一个具有默认初始容量(10)和默认溢出时容量增量值(0)的空 Vector。
 - `Vector(int initialCapacity)：`构造一个带指定初始容量和默认溢出时容量增量值(0)的空 Vector。
 - `Vector(int initialCapacity, int capacityIncrement)：`构造一个带指定初始容量和溢出时容量增量值的空 Vector。
 - `Vector(Collection<? extends E> c)：`构造一个包含指定 Collection 的元素的 Vector。

> `Vector` 是 `List` 接口的`可变数组`的实现，实现了所有可选列表操作，并允许包括 `null` 在内的所有元素。除了实现 `List` 接口外，此类还提供一些方法来操作内部用来存储列表的数组的大小。

## 实现原理 ##
`Vector` 实现了 `Vector` 接口，底层使用数组保存所有元素，其操作基本上是对数组的操作。

### 私有属性 ###
`Vector` 类只定义了两个私有属性：
```java
/** 存储元素的数组缓冲区 */
protected Object[] elementData;

/** 包含的元素数量 */
protected int elementCount;

/** 容量溢出时容量增量值 */
protected int capacityIncrement;
```

> `Vector` 是**`基于数组实现的，是一个动态数组`**，其容量能自动增长，类似于 `C 语言` 中的动态申请内存，动态增长内存。

### set() ###
**`set(int index, E element)`**方法用于用指定的元素替代此列表中指定位置上的元素，并返回以前位于该位置上的元素。下面分析 `set(int index, E element)` 方法的源码： 
```java
public synchronized E set(int index, E element) {
    // 检查指定的索引是否在范围内
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);
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
public synchronized E get(int index) {
    // 检查指定的索引是否在范围内
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);
    // 调用elementData()方法获取指定索引的元素
    return elementData(index);
}

E elementData(int index) {
    // 返回指定索引的元素
    return (E) elementData[index];
}
```

### add() ###
**`add(E e)`**方法用于将指定的元素添加到此列表的尾部。下面分析 `add(E e)` 方法的源码： 
```java
public synchronized boolean add(E e) {
    modCount++;
    // 如果数组长度不足，将进行扩容
    ensureCapacityHelper(elementCount + 1);
    // 将指定元素添加列表的尾部
    elementData[elementCount++] = e;
    return true;
}
```

**`add(int index, E element)`**方法用于将指定的元素插入此列表中的指定位置。下面分析 `add(int index, E element)` 方法的源码： 
```java
public void add(int index, E element) {
    // 直接调用insertElementAt()方法在指定位置插入元素
    insertElementAt(element, index);
}
```

### addAll() ###
**`addAll(Collection<? extends E> c)`**方法用于将指定 `Collection` 中的所有元素按顺序添加到此列表的尾部。下面分析 `addAll(Collection<? extends E> c)` 方法的源码： 
```java
public synchronized boolean addAll(Collection<? extends E> c) {
    modCount++;
    // 将指定集合转换为数组
    Object[] a = c.toArray();
    // 获取转换后的数组长度
    int numNew = a.length;
    // 如果数组长度不足，将进行扩容
    ensureCapacityHelper(elementCount + numNew);
    // 将转换后的数组复制到列表的尾部
    System.arraycopy(a, 0, elementData, elementCount, numNew);
    elementCount += numNew;
    return numNew != 0;
}
```

**`addAll(int index, Collection<? extends E> c)`**方法用于将指定 `Collection` 中的所有元素按顺序从指定的位置开始添加到此列表中。下面分析 `addAll(int index, Collection<? extends E> c)` 方法的源码： 
```java
public synchronized boolean addAll(int index, Collection<? extends E> c) {
    modCount++;
    // 检查指定的索引是否在范围内
    if (index < 0 || index > elementCount)
        throw new ArrayIndexOutOfBoundsException(index);
    // 将指定集合转换为数组
    Object[] a = c.toArray();
    // 获取转换后的数组长度
    int numNew = a.length;
    // 如果数组长度不足，将进行扩容
    ensureCapacityHelper(elementCount + numNew);
    // 需要移动的元素数量
    int numMoved = elementCount - index;
    if (numMoved > 0)
        // 将当前位于该位置的元素以及所有后续元素右移一个位置
        System.arraycopy(elementData, index, elementData, index + numNew,
                numMoved);
    // 将转换后的数组复制到列表中的指定位置
    System.arraycopy(a, 0, elementData, index, numNew);
    elementCount += numNew;
    return numNew != 0;
}
```

### remove() ###
**`remove(int index)`**方法用于移除此列表中指定位置上的元素。下面分析 `remove(int index)` 方法的源码： 
```java
public synchronized E remove(int index) {
    modCount++;
    // 检查指定的索引是否在范围内
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);
    // 获取指定索引的元素
    E oldValue = elementData(index);
    // 需要移动的元素数量
    int numMoved = elementCount - index - 1;
    if (numMoved > 0)
        // 将当前位于该位置的元素的所有后续元素左移一个位置
        System.arraycopy(elementData, index + 1, elementData, index,
                numMoved);
    // 将列表中最后一个元素置为null
    elementData[--elementCount] = null; // Let gc do its work
    return oldValue;
}
```

**`remove(Object o)`**方法用于移除此列表中首次出现的指定元素(如果存在)。下面分析 `remove(Object o)` 方法的源码： 
```java
public boolean remove(Object o) {
    // 直接调用removeElement()方法移除指定元素
    return removeElement(o);
}
```

> 从数组中移除元素的操作，也会导致被移除的元素以后的所有元素的向左移动一个位置。

### contains() ###
**`contains(Object o)`**方法用于判断列表中是否包含指定元素。下面分析 `contains(Object o)` 方法的源码： 
```java
public boolean contains(Object o) {
    return indexOf(o, 0) >= 0;
}
```

### indexOf() ###
**`indexOf(Object o)`**方法用于返回指定元素在列表中首次出现的索引；如果列表不包含该元素，则返回-1。下面分析 `indexOf(Object o)` 方法的源码： 
```java
public int indexOf(Object o) {
    return indexOf(o, 0);
}

public synchronized int indexOf(Object o, int index) {
    // 判断指定元素是否为null
    if (o == null) {
        // 遍历列表中的元素
        for (int i = index; i < elementCount; i++)
            // 判断当前索引位置的元素是否为null
            if (elementData[i] == null)
                // 返回当前索引
                return i;
    } else {
        // 遍历列表中的元素
        for (int i = index; i < elementCount; i++)
            // 判断当前索引位置的元素是否为指定元素
            if (o.equals(elementData[i]))
                // 返回当前索引
                return i;
    }
    return -1;
}
```

> 无论指定元素是否为 `null`，都需要遍历整个数组比较，所以效率是比较低的。

### elementAt() ###
**`elementAt(int index)`**方法用于获取列表中指定位置上的元素。下面分析 `elementAt(int index)` 方法的源码： 
```java
public synchronized E elementAt(int index) {
    // 检查指定的索引是否在范围内
    if (index >= elementCount) {
        throw new ArrayIndexOutOfBoundsException(index + " >= " + elementCount);
    }
    // 调用elementData()方法获取指定索引的元素
    return elementData(index);
}
```

### firstElement() ###
**`firstElement()`**方法用于获取列表中第一个元素，如果列表为空则抛出异常。下面分析 `firstElement()` 方法的源码： 
```java
public synchronized E firstElement() {
    // 检查元素数量是否为0(空集合)
    if (elementCount == 0) {
        throw new NoSuchElementException();
    }
    // 调用elementData()方法获取第一个元素
    return elementData(0);
}
```

### lastElement() ###
**`lastElement()`**方法用于获取列表中最后一个元素，如果列表为空则抛出异常。下面分析 `lastElement()` 方法的源码： 
```java
public synchronized E lastElement() {
    // 检查元素数量是否为0(空集合)
    if (elementCount == 0) {
        throw new NoSuchElementException();
    }
    // 调用elementData()方法获取最后一个元素
    return elementData(elementCount - 1);
}
```

### setElementAt() ###
**`setElementAt(E obj, int index)`**方法用于用指定的元素替代此列表中指定位置上的元素。下面分析 `setElementAt(E obj, int index)` 方法的源码： 
```java
public synchronized void setElementAt(E obj, int index) {
    // 检查指定的索引是否在范围内
    if (index >= elementCount) {
        throw new ArrayIndexOutOfBoundsException(index + " >= " +
                elementCount);
    }
    // 用指定的元素替代此列表中指定位置上的元素
    elementData[index] = obj;
}
```

### removeElement() ###
**`removeElement(Object obj)`**方法用于移除此列表中首次出现的指定元素(如果存在)。下面分析 `removeElement(Object obj)` 方法的源码： 
```java
public synchronized boolean removeElement(Object obj) {
    modCount++;
    // 获取指定元素在列表中首次出现的索引
    int i = indexOf(obj);
    // 判断索引是否大于0(指定元素是否存在)
    if (i >= 0) {
        // 调用removeElementAt()方法移除指定索引位置的元素
        removeElementAt(i);
        return true;
    }
    return false;
}
```

### removeElementAt() ###
**`removeElementAt(int index)`**方法用于移除此列表中指定索引出的元素。下面分析 `removeElementAt(int index)` 方法的源码： 
```java
public synchronized void removeElementAt(int index) {
    modCount++;
    // 检查指定的索引是否在范围内
    if (index >= elementCount) {
        throw new ArrayIndexOutOfBoundsException(index + " >= " +
                elementCount);
    } else if (index < 0) {
        throw new ArrayIndexOutOfBoundsException(index);
    }
    // 需要移动的元素数量
    int j = elementCount - index - 1;
    if (j > 0) {
        // 将当前位于该位置的元素的所有后续元素左移一个位置
        System.arraycopy(elementData, index + 1, elementData, index, j);
    }
    elementCount--;
    // 将列表中最后一个元素置为null
    elementData[elementCount] = null; /* to let gc do its work */
}
```

> 从数组中移除元素的操作，也会导致被移除的元素以后的所有元素的向左移动一个位置。

### insertElementAt() ###
**`insertElementAt(E obj, int index)`**方法用于将指定的元素插入此列表中的指定位置。下面分析 `insertElementAt(E obj, int index)` 方法的源码： 
```java
public synchronized void insertElementAt(E obj, int index) {
    modCount++;
    // 检查指定的索引是否在范围内
    if (index > elementCount) {
        throw new ArrayIndexOutOfBoundsException(index
                + " > " + elementCount);
    }
    // 如果数组长度不足，将进行扩容
    ensureCapacityHelper(elementCount + 1);
    // 将当前位于该位置的元素以及所有后续元素右移一个位置
    System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);
    // 将指定的元素插入列表中的指定位置
    elementData[index] = obj;
    elementCount++;
}
```

### addElement() ###
**`addElement(E obj)`**方法用于将指定的元素添加到此列表的尾部。下面分析 `addElement(E obj)` 方法的源码： 
```java
public synchronized void addElement(E obj) {
    modCount++;
    // 如果数组长度不足，将进行扩容
    ensureCapacityHelper(elementCount + 1);
    // 将指定的元素插入列表中的指定位置
    elementData[elementCount++] = obj;
}
```

### removeAllElements() ###
**`removeAllElements()`**方法用于移除列表中的所有元素。下面分析 `removeAllElements()` 方法的源码： 
```java
public synchronized void removeAllElements() {
    modCount++;
    // Let gc do its work
    // 遍历数组
    for (int i = 0; i < elementCount; i++)
        // 将元素置为null
        elementData[i] = null;
    // 将元素数量置为0
    elementCount = 0;
}
```

### ensureCapacity() ###
**`ensureCapacity(int minCapacity)`**方法用于当向列表中添加元素时，检查添加后元素的个数是否会超出当前数组的长度，如果超出，数组将会进行扩容，以满足添加数据的需求。下面分析 `ensureCapacity(int minCapacity)` 方法的源码： 
```java
public synchronized void ensureCapacity(int minCapacity) {
    if (minCapacity > 0) {
        modCount++;
        // 调用ensureCapacityHelper()进行扩容
        ensureCapacityHelper(minCapacity);
    }
}

private void ensureCapacityHelper(int minCapacity) {
    // overflow-conscious code
    // 判断所需的最小容量是否大于扩容前的数组长度
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    // 旧的容量为扩容前的数组长度
    int oldCapacity = elementData.length;
    // 得到新的容量
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
            capacityIncrement : oldCapacity);
    // 如果新的容量小于最小容量
    if (newCapacity - minCapacity < 0)
        // 取最小容量为新的容量
        newCapacity = minCapacity;
    // 如果新的容量大于数组的最大长度
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        // 如果所需最小容量大于数组的最大长度，则取Integer.MAX_VALUE，反之取数组的最大长度
        newCapacity = hugeCapacity(minCapacity);
    // 将数组中的元素全部复制新的数组中
}
```

`Vector` 构造时可以指定增长因子，如果该增长因子指定了，那么扩容的时候会每次新的数组大小会在原数组的大小基础上加上增长因子；如果不指定增长因子，那么就给原数组大小*2。

> 数组进行扩容时，会将老数组中的元素重新拷贝一份到新的数组中，每次数组容量的增长的代价是很高的，因此在实际使用时，当我们可预知要保存的元素的多少时，要在构造 `Vector`实例时，就指定其容量，以避免数组扩容的发生。或者根据实际需求，通过调用 `ensureCapacity()` 方法来手动增加 `Vector` 实例的容量，以减少递增式再分配的数量。

### trimToSize() ###
**`trimToSize()`**方法用于将底层数组的容量调整为当前列表保存的实际元素的大小。下面分析 `trimToSize()` 方法的源码： 
```java
public synchronized void trimToSize() {
    modCount++;
    int oldCapacity = elementData.length;
    // 判断当前元素数量是否小于数组长度
    if (elementCount < oldCapacity) {
        // 将底层数组的容量调整为当前列表保存的实际元素的大小
        elementData = Arrays.copyOf(elementData, elementCount);
    }
}
```

## Fail-Fast 机制 ##
`Fail-Fast` 机制，即快速失败机制，是 `Java` 集合中的一种错误检测机制。当在迭代集合的过程中该集合在结构上发生改变的时候，就有可能会发生 `Fail-Fast`，即抛出 `ConcurrentModificationException` 异常。`Fail-Fast` 机制并不保证在不同步的修改下一定会抛出异常，它只是尽最大努力去抛出，所以这种机制一般仅用于检测 bug。

`Vector` 采用了快速失败的机制，通过记录 `modCount` 参数来实现。在面对并发的修改时，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。

## 总结 ##
**`Vector`**和 `ArrayList` 一样，都是是通过数组实现的，不同的是 `Vector` 的所有方法都用了 `synchronized` 关键字修饰，是线程安全的，即某一时刻只有一个线程能够写 `Vector`，但实现同步需要很高的花费，因此，访问它比访问 `ArrayList` 慢。 

由 `Vector` 创建的 `Iterator`，虽然和 `ArrayList` 创建的 `Iterator` 是同一接口，但是，因为 `Vector` 是同步的，当一个 `Iterator` 被创建而且正在被使用，另一个线程改变了 `Vector` 的状态(例如，添加、删除元素)，这时调用 `Iterator` 的方法时将抛出 `ConcurrentModificationException`，因此必须捕获该异常。

