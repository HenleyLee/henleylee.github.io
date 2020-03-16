---
title: Java 集合之 LinkedList 源码分析
categories: Java
tags:
  - Java
abbrlink: 2d1165aa
date: 2019-08-06 15:56:35
---

`LinkedList` 是用链表结构存储数据的，比较适合数据的动态插入和删除，随机访问和遍历速度比较慢，还提供了 `List` 接口中没有定义的方法，专门用于操作表头和表尾的元素，所以可以当作堆栈、队列和双向队列来使用。

## 简介 ##
`java.util.LinkedList` 实现了 `java.util.List` 接口，继承 `java.util.AbstractSequentialList`。

`java.util.List` 是 Java 集合(Collection)中的一部分，是一个继承了 `java.util.Collection` 接口的接口，它有诸多特性，所以使用的场景会很多。下面先简单了解下它的特性：
 - 元素有序；
 - 元素可重复；
 - 每个元素都有自己的顺序索引。

`LinkedList` 有以下特点：
 - `LinkedList` 是一个继承于 `java.util.AbstractSequentialList` 的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。
 - `LinkedList` 实现 `java.util.List` 接口，能对它进行队列操作。
 - `LinkedList` 实现 `java.util.Deque` 接口，即能将 `LinkedList` 当作双端队列使用。
 - `LinkedList` 实现了 `java.lang.Cloneable` 接口，即覆盖了 `clone()` 方法，能克隆。
 - `LinkedList` 实现 `java.io.Serializable` 接口，这意味着 `LinkedList` 支持序列化，能通过序列化去传输。
 - `LinkedList` 是非线程安全的。

`LinkedList` 持有头节点和尾节点的引用，提供了两个构造函数：
 - `LinkedList()：`构造一个空 LinkedList。
 - `LinkedList(Collection<? extends E> c)：`构造一个包含指定 Collection 的元素的 LinkedList。

> `LinkedList` 是 `List` 接口的`双向链表`的实现，实现了所有可选链表操作，并允许包括 `null` 在内的所有元素。除了实现 `List` 接口外，还实现了 `Deque` 接口，是一个支持在两端插入和删除元素的线性集合。

## 实现原理 ##
`LinkedList` 实现了 `List` 接口，底层使用双向链表保存所有元素，其操作基本上是对双向链表的操作。

### 私有属性 ###
`ArrayList` 类只定义了两个私有属性：
```java
/** 包含的元素数量 */
transient int size = 0;

/** 指向第一个节点的指针 */
transient Node<E> first;

/** 指向最后一个节点的指针 */
transient Node<E> last;
```

`size`、`first` 和 `last` 用到了 Java 的关键字 `transient`，Java 中的 `transient` 关键字是短暂的意思。对于 `transient` 修饰的成员变量，在类实例的序列化处理过程中会被忽略。因此，`transient` 变量不会贯穿对象的序列化和反序列化，生命周期仅存于调用者的内存中而不会写到磁盘里持久化。

> `LinkedList` 底层的数据结构是基于`双向循环链表`的，且头结点中不存放数据。

### Node ###
`java.util.LinkedList.Node` 是 `LinkedList` 中的节点所对应的类：
```java
private static class Node<E> {
    /** 存放数据 */
    E item;
    /** 存放后节点信息 */
    Node<E> next;
    /** 存放前节点信息 */
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

### 集合操作 ###
#### set() ####
**`set(int index, E element)`**方法用于用指定的元素替代此链表中指定位置上的元素，并返回以前位于该位置上的元素。下面分析 `set(int index, E element)` 方法的源码： 
```java
public E set(int index, E element) {
    // 检查指定的索引是否在范围内
    checkElementIndex(index);
    // 获取指定索引处的非空节点
    Node<E> x = node(index);
    // 获取该索引处节点的数据
    E oldVal = x.item;
    // 用指定的元素替代此链表中指定位置上的元素
    x.item = element;
    return oldVal;
}
```

#### get() ####
**`get(int index)`**方法用于获取链表中指定位置上的元素。下面分析 `get(int index)` 方法的源码： 
```java
public E get(int index) {
    // 检查指定的索引是否在范围内
    checkElementIndex(index);
    // 返回指定索引的元素
    return node(index).item;
}
```

#### add() ####
**`add(E e)`**方法用于将指定的元素添加到此链表的尾部。下面分析 `add(E e)` 方法的源码： 
```java
public boolean add(E e) {
    // 将指定元素链接为最后一个元素
    linkLast(e);
    return true;
}
```

**`add(int index, E element)`**方法用于将指定的元素插入此链表中的指定位置。下面分析 `add(int index, E element)` 方法的源码： 
```java
public void add(int index, E element) {
    // 检查指定的索引是否在范围内
    checkPositionIndex(index);
    // 如果指定索引等于链表中的元素数量
    if (index == size)
        // 将指定元素链接为最后一个元素
        linkLast(element);
    else
        // 在指定索引处的非空节点之前插入指定元素
        linkBefore(element, node(index));
}
```

`linkLast()` 方法用于将指定元素链接为最后一个元素。下面分析  `linkLast()` 方法的源码：
```java
void linkLast(E e) {
    // 将最后一个节点赋值给l
    final Node<E> l = last;
    // 将最后一个节点作为前驱并创建新的节点
    final Node<E> newNode = new Node<>(l, e, null);
    // 将新创建的节点作为新的最后一个节点
    last = newNode;
    // 如果之前的最后一个节点为null(链表为空)
    if (l == null)
        // 将新创建的节点最为第一个节点
        first = newNode;
    else
        // 将新创建的节点最为之前最后一个节点的后继
        l.next = newNode;
    size++;
    modCount++;
}
```

`linkBefore()` 方法用于在指定索引处的非空节点之前插入指定元素。下面分析  `linkBefore()` 方法的源码：
```java
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    // 将指定索引的节点的前驱赋值给pred
    final Node<E> pred = succ.prev;
    // 将指定索引的节点的前驱作为前驱，指定索引的节点作为后继并创建新的节点
    final Node<E> newNode = new Node<>(pred, e, succ);
    // 将新创建的节点作为指定索引的节点的前驱
    succ.prev = newNode;
    // 如果之前指定索引的节点的前驱为空(之前指定索引的节点为头结点)
    if (pred == null)
        // 将新创建的节点最为第一个节点
        first = newNode;
    else
        // 将新创建的节点最为之前指定索引的节点的后继
        pred.next = newNode;
    size++;
    modCount++;
}
```

#### addAll() ####
**`addAll(Collection<? extends E> c)`**方法用于将指定 `Collection` 中的所有元素按顺序添加到此链表的尾部。下面分析 `addAll(Collection<? extends E> c)` 方法的源码： 
```java
public boolean addAll(Collection<? extends E> c) {
    // 将将指定Collection中的所有元素插入到链表的尾部
    return addAll(size, c);
}
```

**`addAll(int index, Collection<? extends E> c)`**方法用于将指定 `Collection` 中的所有元素按顺序从指定的位置开始添加到此链表中。下面分析 `addAll(int index, Collection<? extends E> c)` 方法的源码： 
```java
public boolean addAll(int index, Collection<? extends E> c) {
    // 检查指定的索引是否在范围内
    checkPositionIndex(index);
    // 将指定集合转换为数组
    Object[] a = c.toArray();
    // 获取转换后的数组长度
    int numNew = a.length;
    // 如果指定集合中的元素长度为0，则直接返回
    if (numNew == 0)
        return false;

    Node<E> pred, succ;
    // 如果指定索引等于链表中的元素数量
    if (index == size) {
        // 指定索引的元素为null，并将最后一个元素赋值给pred
        succ = null;
        pred = last;
    } else {
        // 获取指定索引的元素，并将它的前驱赋值给pred
        succ = node(index);
        pred = succ.prev;
    }
    // 遍历要插入元素的数组
    for (Object o : a) {
        // 将得到的pred作为前驱并创建新的节点
        @SuppressWarnings("unchecked") E e = (E) o;
        Node<E> newNode = new Node<>(pred, e, null);
        // 如果pred为null(链表为空)
        if (pred == null)
            // 将新创建的节点最为第一个节点
            first = newNode;
        else
            // 将新创建的节点最为pred的后继
            pred.next = newNode;
        // 将新创建的节点赋值给pred
        pred = newNode;
    }
    // 如果succ为null(直接在链表尾部插入)
    if (succ == null) {
        // 将pred作为最后一个节点
        last = pred;
    } else {
        // 将succ作为pred的后继
        pred.next = succ;
        // 将pred作为succ的前驱
        succ.prev = pred;
    }

    size += numNew;
    modCount++;
    return true;
}
```

#### remove() ####
**`remove(int index)`**方法用于移除此链表中指定位置上的元素。下面分析 `remove(int index)` 方法的源码： 
```java
public E remove(int index) {
    // 检查指定的索引是否在范围内
    checkElementIndex(index);
    // 获取指定索引处的节点并从链表中移除
    return unlink(node(index));
}
```

**`remove(Object o)`**方法用于移除此链表中首次出现的指定元素(如果存在)。下面分析 `remove(Object o)` 方法的源码： 
```java
public boolean remove(Object o) {
    // 如果要移除的节点为null
    if (o == null) {
        // 遍历链表
        for (Node<E> x = first; x != null; x = x.next) {
            // 如果当前节点的数据为null
            if (x.item == null) {
                // 移除当前节点
                unlink(x);
                return true;
            }
        }
    } else {
        // 遍历链表
        for (Node<E> x = first; x != null; x = x.next) {
            // 如果当前节点的数据为需要被移除的元素
            if (o.equals(x.item)) {
                // 移除当前节点
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

`unlink()` 方法用于从链表中移除指定的非空节点。
```java
E unlink(Node<E> x) {
    // assert x != null;
    // 将指定节点的数据赋值给element
    final E element = x.item;
    // 将指定节点的后继赋值给next
    final Node<E> next = x.next;
    // 将指定节点的前驱赋值给prev
    final Node<E> prev = x.prev;
    // 如果prev为null(指定节点为头结点)
    if (prev == null) {
        // 将next作为第一个节点
        first = next;
    } else {
        // 将next作为prev的后继
        prev.next = next;
        x.prev = null;
    }
    // 如果next为null(指定节点为尾结点)
    if (next == null) {
        // 将prev作为最后一个节点
        last = prev;
    } else {
        // 将prev作为next的前驱
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}
```

#### contains() ####
**`contains(Object o)`**方法用于判断链表中是否包含指定元素。下面分析 `contains(Object o)` 方法的源码： 
```java
public boolean contains(Object o) {
    // 判断指定元素在链表中首次出现的索引是否不等于-1
    return indexOf(o) != -1;
}
```

#### indexOf() ####
**`indexOf(Object o)`**方法用于返回指定元素在链表中首次出现的索引；如果链表不包含该元素，则返回-1。下面分析 `indexOf(Object o)` 方法的源码： 
```java
public int indexOf(Object o) {
    int index = 0;
    // 判断指定元素是否为null
    if (o == null) {
        // 遍历链表中的元素
        for (Node<E> x = first; x != null; x = x.next) {
            // 判断当前索引位置的元素是否为null
            if (x.item == null)
                // 返回当前索引
                return index;
            index++;
        }
    } else {
        // 遍历链表中的元素
        for (Node<E> x = first; x != null; x = x.next) {
            // 判断当前索引位置的元素是否为指定元素
            if (o.equals(x.item))
                // 返回当前索引
                return index;
            index++;
        }
    }
    return -1;
}
```

> 无论指定元素是否为 `null`，都需要遍历整个数组比较，所以效率是比较低的。

#### node() ####
**`node(int index)`**方法用于返回指定元素索引处的非空节点。下面分析 `node(int index)` 方法的源码： 
```java
Node<E> node(int index) {
    // assert isElementIndex(index);
    //如果下标在链表前半部分, 就从头开始查起
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    }
    //如果下标在链表后半部分, 就从尾开始查起
    else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

> 因此通过下标的查找和修改操作的时间复杂度是 `O(n/2)`，通过对双向链表的操作还可以实现单项队列，双向队列和栈的功能。

### 单向队列操作 ###
`LinkedList` 可被用作 `Queue`(单向队列)。

#### peek() ####
**`peek()`**方法用于检索但不删除链表的头结点。下面分析 `peek()` 方法的源码： 
```java
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}
```

#### element() ####
**`element()`**方法用于检索但不删除链表的头结点，如果链表为空则抛出异常。下面分析 `element()` 方法的源码： 
```java
public E element() {
    return getFirst();
}
```

#### poll() ####
**`poll()`**方法用于检索并删除链表的头结点。下面分析 `poll()` 方法的源码： 
```java
public E poll() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}
```

#### remove() ####
**`remove()`**方法用于检索并删除链表的头结点，如果链表为空则抛出异常。下面分析 `remove()` 方法的源码： 
```java
public E remove() {
    return removeFirst();
}
```

#### offer() ####
**`offer()`**方法用于将指定的元素添加到此链表的尾部。下面分析 `offer()` 方法的源码： 
```java
public boolean offer(E e) {
    return add(e);
}
```

### 双向队列操作 ###
`LinkedList` 可被用作 `Deque`(双向队列)。

#### offerFirst() ####
**`offerFirst()`**方法用于将指定的元素插入此链表的头部。下面分析 `offerFirst()` 方法的源码： 
```java
public boolean offerFirst(E e) {
    addFirst(e);
    return true;
}
```

#### offerLast() ####
**`offerLast()`**方法用于将指定的元素插入此链表的尾部。下面分析 `offerLast()` 方法的源码： 
```java
public boolean offerLast(E e) {
    addLast(e);
    return true;
}
```

#### peekFirst() ####
**`peekFirst()`**方法用于检索但不删除此列表的第一个元素，如果此列表为空，则返回 `null`。下面分析 `peekFirst()` 方法的源码： 
```java
public E peekFirst() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}
```

#### peekLast() ####
**`peekLast()`**方法用于检索但不删除此列表的最后一个元素，如果此列表为空，则返回 `null`。下面分析 `peekLast()` 方法的源码： 
```java
public E peekLast() {
    final Node<E> l = last;
    return (l == null) ? null : l.item;
}
```

#### pollFirst() ####
**`pollFirst()`**方法用于检索并删除此列表的第一个元素，如果此列表为空，则返回 `null`。下面分析 `pollFirst()` 方法的源码： 
```java
public E pollFirst() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}
```

#### pollLast() ####
**`pollLast()`**方法用于检索并删除此列表的最后一个元素，如果此列表为空，则返回 `null`。下面分析 `pollLast()` 方法的源码： 
```java
public E pollLast() {
    final Node<E> l = last;
    return (l == null) ? null : unlinkLast(l);
}
```

### 栈操作 ###
`LinkedList` 可被用作 `Stack`(堆栈)。

#### push() ####
**`push()`**方法用于将指定元素压入此列表表示的堆栈，即将指定元素插入此列表的头部。下面分析 `push()` 方法的源码： 
```java
public void push(E e) {
    addFirst(e);
}
```

#### pop() ####
**`pop()`**方法用于将指定元素弹出此列表表示的堆栈，即检索并删除此列表的第一个元素。下面分析 `pop()` 方法的源码： 
```java
public E pop() {
    return removeFirst();
}
```

## Fail-Fast 机制 ##
`Fail-Fast` 机制，即快速失败机制，是 `Java` 集合中的一种错误检测机制。当在迭代集合的过程中该集合在结构上发生改变的时候，就有可能会发生 `Fail-Fast`，即抛出 `ConcurrentModificationException` 异常。`Fail-Fast` 机制并不保证在不同步的修改下一定会抛出异常，它只是尽最大努力去抛出，所以这种机制一般仅用于检测 bug。

`LinkedList` 采用了快速失败的机制，通过记录 `modCount` 参数来实现。在面对并发的修改时，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。

## 总结 ##
**`LinkedList`**存储元素的数据结构是双向链表结构，由存储元素的结点连接而成，每一个节点都包含前一个节点的引用，后一个节点的引用和节点存储的值。当一个新节点插入时，只需要修改其中保持先后关系的节点的引用即可。所以 `LinkedList` 拥有着数组的特性：
 - `LinkedList` 是基于双向链表实现的，不论是增删改查方法还是队列和栈的实现，都可通过操作结点实现；
 - `LinkedList` 无需提前指定容量，因为基于链表操作，集合的容量随着元素的加入自动增加；
 - `LinkedList` 删除元素后集合占用的内存自动缩小，无需像 `ArrayList` 一样调用 `trimToSize()` 方法；
 - `LinkedList` 的所有方法没有进行同步，因此它不是线程安全的，应该避免在多线程环境下使用。

