---
title: Java ConcurrentHashMap 的实现原理
categories: Java
tags:
  - Java
abbrlink: 9cd25f5c
date: 2019-07-21 18:36:55
---

在多线程环境下，使用 `HashMap` 进行 `put` 操作时存在丢失数据的情况，为了避免这种 bug 的隐患，强烈建议使用 `ConcurrentHashMap` 代替 `HashMap`。

`HashTable` 是一个线程安全的类，它使用 `synchronized` 来锁住整张 `Hash` 表来实现线程安全，即每次锁住整张表让线程独占，相当于所有线程进行读写时都去竞争一把锁，导致效率非常低下。

`ConcurrentHashMap` 可以做到读取数据不加锁，并且其内部的结构可以让其在进行写操作的时候能够将锁的粒度保持地尽量地小，允许多个修改操作并发进行，其关键在于使用了锁分离技术。它使用了多个锁来控制对 `hash` 表的不同部分进行的修改。`ConcurrentHashMap` 内部使用 `Segment` 来表示这些不同的部分，每个段其实就是一个小的 `Hashtable`，它们有自己的锁。只要多个修改操作发生在不同的段上，它们就可以并发进行。

## ConcurrentHashMap 简介 ##
`java.util.concurrent.ConcurrentHashMap` 实现了 `java.util.Map` 接口，继承 `java.util.AbstractMap`。其中 `Map` 接口定义了键映射到值的规则，而 `AbstractMap` 类提供 `Map` 接口的骨干实现。

`ConcurrentHashMap` 提供了三个构造函数：
 - `HashMap()：`构造一个具有默认初始容量 (16) 和默认加载因子 (0.75) 的空 HashMap。
 - `HashMap(int initialCapacity)：`构造一个带指定初始容量和默认加载因子 (0.75) 的空 HashMap。
 - `HashMap(int initialCapacity, float loadFactor)：`构造一个带指定初始容量和加载因子的空 HashMap。

`ConcurrentHashMap` 采用了非常精妙的**`分段锁`**策略，`ConcurrentHashMap` 的主干是个 `Segment` 数组。`Segment` 继承了 `ReentrantLock`，所以它就是一种可重入锁(ReentrantLock)。在 `ConcurrentHashMap` 中，一个 `Segment`就是一个子哈希表，`Segment` 里维护了一个 `HashEntry` 数组，并发环境下，对于不同 `Segment` 的数据进行操作是不用考虑锁竞争的。

## ConcurrentHashMap 实现原理 ##
`ConcurrentHashMap` 为了提高本身的并发能力，在内部采用了一个叫做 `Segment` 的结构，一个 `Segment` 其实就是一个类 `HashTable` 的结构，`Segment` 内部维护了一个链表数组，我们用下面这一幅图来看下 `ConcurrentHashMap` 的内部结构，从下面的结构我们可以了解到，`ConcurrentHashMap` 定位一个元素的过程需要进行两次 `Hash` 操作，第一次 `Hash` 定位到 `Segment`，第二次 `Hash` 定位到元素所在的链表的头部，因此这一种结构的带来的副作用是 `Hash` 的过程要比普通的 `HashMap` 要长，但是带来的好处是写操作的时候可以只对元素所在的 `Segment` 进行操作即可，不会影响到其他的 `Segment`，这样，在最理想的情况下，`ConcurrentHashMap` 可以最高同时支持 `Segment` 数量大小的写操作(刚好这些写操作都非常平均地分布在所有的 `Segment` 上)，所以，通过这一种结构，`ConcurrentHashMap` 的并发能力可以大大的提高。我们用下面这一幅图来看下 `ConcurrentHashMap` 的内部结构详情图，如下：
![ConcurrentHashMap的数据结构](https://henleylee.github.io/medias/java/concurrent_hashmap_structure.png)

不难看出，`ConcurrentHashMap` 采用了二次 `hash` 的方式，第一次 `hash` 将 `key` 映射到对应的 `segment`，而第二次 `hash` 则是映射到 `segment` 的不同桶(bucket)中。

为什么要用二次 `Hash`，主要原因是为了构造分离锁，使得对于 `Map` 的修改不会锁住整个容器，提高并发能力。当然，没有一种东西是绝对完美的，二次 `Hash` 带来的问题是整个 `Hash` 的过程比 `HashMap` 单次 `Hash` 要长，所以，如果不是并发情形，不要使用 `ConcurrentHashMap`。

JDK 1.7 之前 `ConcurrentHashMap` 主要采用锁机制，在对某个 `Segment` 进行操作时，将该 `Segment` 锁定，不允许对其进行非查询操作，而在 JDK 1.8 之后采用 `CAS` 无锁算法，这种乐观操作在完成前进行判断，如果符合预期结果才给予执行，对并发操作提供良好的优化。

### JDK 1.7 的实现原理 ###
`ConcurrentHashMap` 是由 `Segment` 数组、`HashEntry` 组成，和 `HashMap` 一样，仍然是数组加链表。

`ConcurrentHashMap` 类中包含两个静态内部类 `HashEntry` 和 `Segment`。`HashEntry` 用来封装映射表的键/值对；`Segment` 用来充当锁的角色，每个 `Segment` 对象守护整个散列映射表的若干个桶。每个桶是由若干个 `HashEntry` 对象链接起来的链表。一个 `ConcurrentHashMap` 实例中包含由若干个 `Segment` 对象组成的数组。

#### HashEntry ####
`HashEntry` 用来封装散列映射表中的键值对。在 `HashEntry` 类中，`key`、`hash` 和 `next` 域都被声明为 `final` 型，`value` 域被声明为 `volatile` 型。
```java
static final class HashEntry<K,V> { 
       final K key;                  // 声明 key 为 final 型
       final int hash;               // 声明 hash 值为 final 型 
       volatile V value;             // 声明 value 为 volatile 型
       final HashEntry<K,V> next;    // 声明 next 为 final 型 
 
       HashEntry(K key, int hash, HashEntry<K,V> next, V value) { 
           this.key = key; 
           this.hash = hash; 
           this.next = next; 
           this.value = value; 
       } 
}
```

在 `ConcurrentHashMap` 中，在散列时如果产生“碰撞”，将采用“分离链接法”来处理“碰撞”：把“碰撞”的 `HashEntry` 对象链接成一个链表。由于 `HashEntry` 的 `next` 域为 `final` 型，所以新节点只能在链表的表头处插入。由于只能在表头插入，所以链表中节点的顺序和插入的顺序相反。

#### Segment ####
`Segment` 类继承于 `ReentrantLock` 类，从而使得 `Segment` 对象能充当锁的角色。每个 `Segment` 对象用来守护其(成员对象 `table` 中)包含的若干个桶。
```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {

       /** 在本segment范围内，包含的HashEntry元素的个数 */ 
       transient volatile int count; 
 
       /** table被更新的次数 */ 
       transient int modCount; 
 
       /** 当table中包含的HashEntry元素的个数超过本变量值时，触发table的再散列 */ 
       transient int threshold; 
 
       /** table是由HashEntry对象组成的数组 */ 
       transient volatile HashEntry<K,V>[] table; 
 
       /** 装载因子 */ 
       final float loadFactor; 
 
       Segment(int initialCapacity, float lf) { 
           loadFactor = lf; 
           setTable(HashEntry.<K,V>newArray(initialCapacity)); 
       } 
 
       /** 
        * 设置table引用到这个新生成的HashEntry数组
        */ 
       void setTable(HashEntry<K,V>[] newTable) { 
           // 计算临界阀值为新数组的长度与装载因子的乘积
           threshold = (int)(newTable.length * loadFactor); 
           table = newTable; 
       } 
 
       /** 
        * 根据key的散列值，找到table中对应的那个桶(table数组的某个数组成员)
        */ 
       HashEntry<K,V> getFirst(int hash) { 
           HashEntry<K,V>[] tab = table; 
           // 把散列值与 table 数组长度减 1 的值相“与”，
           // 得到散列值对应的 table 数组的下标
           // 然后返回 table 数组中此下标对应的 HashEntry 元素
           return tab[hash & (tab.length - 1)]; 
       }

}
```

`table` 是一个由 `HashEntry` 对象组成的数组。`table` 数组的每一个数组成员就是散列映射表的一个桶。

`count` 变量是一个计数器，它表示每个 `Segment` 对象管理的 `table` 数组(若干个 `HashEntry` 组成的链表)包含的 `HashEntry` 对象的个数。每一个 `Segment` 对象都有一个 `count` 对象来表示本 `Segment` 中包含的 `HashEntry` 对象的总数。注意，之所以在每个 `Segment` 对象中包含一个计数器，而不是在 `ConcurrentHashMap` 中使用全局的计数器，是为了避免出现“热点域”而影响 `ConcurrentHashMap` 的并发性。

#### ConcurrentHashMap ####
```java
public class ConcurrentHashMap<K, V> extends AbstractMap<K, V> 
       implements ConcurrentMap<K, V>, Serializable { 
 
   /** 默认初始容量 */ 
   static final int DEFAULT_INITIAL_CAPACITY= 16; 
 
   /** 默认装载因子 */ 
   static final float DEFAULT_LOAD_FACTOR= 0.75f; 
 
   /** 散列表的默认并发级别为 16，该值表示当前更新线程的估计数 */ 
   static final int DEFAULT_CONCURRENCY_LEVEL= 16; 
 
   /** segments 的掩码值，key 的散列码的高位用来选择具体的 segment */ 
   final int segmentMask; 
 
   /** 偏移量 */ 
   final int segmentShift; 
 
   /** 由 Segment 对象组成的数组 */ 
   final Segment<K,V>[] segments; 

   /** 
    * 创建一个带有默认初始容量(16)、默认加载因子(0.75) 和 默认并发级别(16)的空散列映射表。
    */ 
   public ConcurrentHashMap() { 
       // 使用三个默认参数，调用上面重载的构造函数来创建空散列映射表
       this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR, DEFAULT_CONCURRENCY_LEVEL);
   }

   /** 
    * 创建一个带有指定初始容量、默认加载因子(0.75)和默认并发级别(16)的空散列映射表。
    */ 
   public ConcurrentHashMap(int initialCapacity) { 
       // 使用三个默认参数，调用上面重载的构造函数来创建空散列映射表
       this(initialCapacity, DEFAULT_LOAD_FACTOR, DEFAULT_CONCURRENCY_LEVEL);
   }

   /** 
    * 创建一个带有指定初始容量、指定加载因子和默认并发级别(16)的空散列映射表。
    */ 
   public ConcurrentHashMap(int initialCapacity, float loadFactor) { 
       // 使用三个默认参数，调用上面重载的构造函数来创建空散列映射表
       this(initialCapacity, loadFactor, DEFAULT_CONCURRENCY_LEVEL);
   }

   /** 
    * 创建一个带有指定初始容量、加载因子和并发级别的新的空映射。
    */ 
   public ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel) { 
       if(!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0) 
           throw new IllegalArgumentException(); 
 
       if(concurrencyLevel > MAX_SEGMENTS) 
           concurrencyLevel = MAX_SEGMENTS; 
 
       // 寻找最佳匹配参数（不小于给定参数的最接近的2次幂） 
       int sshift = 0; 
       // segment数组的长度是由concurrentLevel计算来的，segment数组的长度是2的N次方，
       // 默认concurrencyLevel = 16, 所以ssize在默认情况下也是16，此时 sshift = 4
       // sshift相当于ssize从1向左移的次数
       int ssize = 1; 
       while(ssize < concurrencyLevel) { 
           ++sshift; 
           ssize <<= 1; 
       }
       // 段偏移量，默认值情况下此时segmentShift = 28
       segmentShift = 32 - sshift;
       // 散列算法的掩码，默认值情况下segmentMask = 15
       segmentMask = ssize - 1;
       // 创建ssize长度的Segment数组
       this.segments = Segment.newArray(ssize);
 
       if (initialCapacity > MAXIMUM_CAPACITY) 
           initialCapacity = MAXIMUM_CAPACITY;
       int c = initialCapacity / ssize; 
       if(c * ssize < initialCapacity) 
           ++c; 
       int cap = 1; 
       while(cap < c) 
           cap <<= 1; 
 
       // 依次遍历每个数组元素
       for(int i = 0; i < this.segments.length; ++i) 
           // 初始化每个数组元素引用的 Segment 对象
           this.segments[i] = new Segment<K,V>(cap, loadFactor); 
   } 
 
}
```

其中，`concurrencyLevel` 一经指定，不可改变，后续如果 `ConcurrentHashMap` 的元素数量增加导致 `ConrruentHashMap` 需要扩容，`ConcurrentHashMap` 不会增加 `Segment` 的数量，而只会增加 `Segment` 中链表数组的容量大小，这样的好处是扩容过程不需要对整个 `ConcurrentHashMap` 做 `rehash`，而只需要对 `Segment` 里面的元素做一次 `rehash` 就可以了。

整个 `ConcurrentHashMap` 的初始化方法还是非常简单的，先是根据 `concurrencyLevel` 来 `new` 出 `Segment`，这里 `Segment` 的数量是不大于 `concurrencyLevel` 的最大的2的指数，就是说 `Segment` 的数量永远是2的指数个，这样的好处是方便采用移位操作来进行 `hash`，加快 `hash`的过程。接下来就是根据 `intialCapacity` 确定 `Segment` 的容量的大小，每一个 `Segment` 的容量大小也是2的指数，同样使为了加快 `hash` 的过程。

注意一下两个变量 `segmentShift` 和 `segmentMask`，这两个变量在后面将会起到很大的作用，假设构造函数确定了 `Segment` 的数量是2的n次方，那么 `segmentShift` 就等于32减去 n，而 `segmentMask` 就等于2的 n 次方减一。

#### put() 方法 ####
**`put()`**方法的过程是先计算 `hash`，然后通过 `hash` 找到对应的 `Segment`，最后将键值对保存到对应的 `Segment` 中。下面分析 `put()` 方法的源码：
```java
public V put(K key, V value) {
       // ConcurrentHashMap 中不允许用 null 作为映射值
       if (value == null)
           throw new NullPointerException(); 
       // 计算键对应的hash值
       int hash = hash(key.hashCode());
       // 根据hash值找到对应的Segment并将键值对保存到对应的Segment中
       return segmentFor(hash).put(key, hash, value, false); 
}
```

`segmentFor()` 方法的作用是根据 `hash` 值找到对应的 `Segment`，具体实现如下：
```java
final Segment<K,V> segmentFor(int hash) { 
       // 将散列值右移 segmentShift 个位，并在高位填充 0 
       // 然后把得到的值与 segmentMask 相“与”
       // 从而得到 hash 值对应的 segments 数组的下标值
       // 最后根据下标值返回散列码对应的 Segment 对象
       return segments[(hash >>> segmentShift) & segmentMask]; 
}
```

通过 `key` 定位到 `Segment` 之后，在对应的 `Segment` 中进行具体的 `put` 操作。下面分析 `Segment.put()` 方法的源码：
```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    // 尝试获取锁，如果获取失败肯定就有其他线程存在竞争，
    // 则利用 scanAndLockForPut() 自旋获取锁
    HashEntry<K, V> node = tryLock() ? null : scanAndLockForPut(key, hash, value);
    V oldValue;
    try {
        // 每一个Segment对应一个HashEntry[ ]数组
        HashEntry<K, V>[] tab = table;
        // 把hash值与table数组的长度减 1 的值相“与”
        // 得到该hash值对应的table数组的下标值
        int index = (tab.length - 1) & hash;
        // 找到hash值对应的具体的那个桶
        HashEntry<K, V> first = entryAt(tab, index);
        // 遍历链表
        for (HashEntry<K, V> e = first; ; ) {
            if (e != null) {
                K k;
                if ((k = e.key) == key ||
                        (e.hash == hash && key.equals(k))) {
                    oldValue = e.value;
                    if (!onlyIfAbsent) {
                        e.value = value;
                        ++modCount;
                    }
                    break;
                }
                e = e.next;
            }
            // 如果链表为空（即表头为空）
            else {
                if (node != null)
                    // 将新节点插入到链表作为链表头
                    node.setNext(first);
                else
                    // 根据key和value 创建结点并插入链表
                    node = new HashEntry<K, V>(hash, key, value, first);
                int c = count + 1;
                // 判断元素个数是否超过了阈值或者segment中数组的长度超过了MAXIMUM_CAPACITY
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    // 如果满足条件则rehash扩容
                    rehash(node);
                else
                    // 不需要扩容时，将node放到数组（HashEntry[]）中对应的位置
                    setEntryAt(tab, index, node);
                ++modCount;
                count = c;
                oldValue = null;
                break;
            }
        }
    } finally {
        // 最后释放锁
        unlock();
    }
    return oldValue;
}
```
总的来说，`put()` 的流程如下：
 - 将当前 `Segment` 中的 `table` 通过 `key` 的 `hashcode` 定位到 `HashEntry`。
 - 遍历该 `HashEntry`，如果不为空则判断传入的 `key` 和当前遍历的 `key` 是否相等，相等则覆盖旧的 `value`。
 - 不为空则需要新建一个 `HashEntry` 并加入到 `Segment` 中，同时会先判断是否需要扩容。
 - 最后会解除在所获取当前 `Segment` 的锁。

#### get() 方法
**`get()`**方法的过程是先计算 `hash`，然后通过 `hash` 定位到具体的 `Segment`，然后再通过一次 `hash` 定位到 `Segment` 中具体的元素上。下面分析 `get()` 方法的源码：
```java
V get(Object key, int hash) {
    // 首先读 count 变量
    if (count != 0) {
        HashEntry<K, V> e = getFirst(hash);
        while (e != null) {
            if (e.hash == hash && key.equals(e.key)) {
                V v = e.value;
                if (v != null)
                    return v;
                // 如果读到 value 域为 null，说明发生了重排序，加锁后重新读取
                return readValueUnderLock(e);
            }
            e = e.next;
        }
    }
    return null;
}

V readValueUnderLock(HashEntry<K, V> e) {
    lock();
    try {
        return e.value;
    } finally {
        unlock();
    }
}
```

#### rehash() 方法 ####
**`rehash()`** 方法用于当元素个数超过阈值或者 `Segment` 中数组的长度超过最大容量时进行扩容。下面分析 `rehash()` 方法的源码：
```java
private void rehash(HashEntry<K, V> node) {
    HashEntry<K, V>[] oldTable = table;
    int oldCapacity = oldTable.length;
    // 扩大1倍（左移一位）
    int newCapacity = oldCapacity << 1;
    // 计算新的阈值
    threshold = (int) (newCapacity * loadFactor);
    // 创建新的数组
    HashEntry<K, V>[] newTable =
            (HashEntry<K, V>[]) new HashEntry[newCapacity];
    // mask
    int sizeMask = newCapacity - 1;
    // 遍历旧数组数据
    for (int i = 0; i < oldCapacity; i++) {
        // 对应一个链表的表头结点
        HashEntry<K, V> e = oldTable[i];
        if (e != null) {
            HashEntry<K, V> next = e.next;
            // 计算e对应的这条链表在新数组中对应的下标
            int idx = e.hash & sizeMask;
            if (next == null)
                // 只有一个结点时直接放入（新的）数组中
                newTable[idx] = e;
            else {
                // 链表有多个结点时，就链表的表头结点做为新链表的尾结点
                HashEntry<K, V> lastRun = e;
                int lastIdx = idx;
                for (HashEntry<K, V> last = next;
                     last != null;
                     last = last.next) {
                    // 旧数组中一个链表中的数据并不一定在新数组中属于同一个链表
                    // 所以这里需要每次都重新计算
                    int k = last.hash & sizeMask;
                    if (k != lastIdx) {
                        lastIdx = k;
                        lastRun = last;
                    }
                }
                // lastRun（和之后的元素）插入数组中。
                newTable[lastIdx] = lastRun;
                // 从（旧链表）头结点向后遍历，遍历到最后一组不同于前面hash值的组头。
                for (HashEntry<K, V> p = e; p != lastRun; p = p.next) {
                    V v = p.value;
                    int h = p.hash;
                    int k = h & sizeMask;
                    HashEntry<K, V> n = newTable[k];
                    // 拼接链表
                    newTable[k] = new HashEntry<K, V>(h, p.key, v, n);
                }
            }
        }
    }
    // 将之前的旧数据都添加到新的结构中之后，才会插入新的结点（依旧是插入表头）
    int nodeIndex = node.hash & sizeMask;
    node.setNext(newTable[nodeIndex]);
    newTable[nodeIndex] = node;
    table = newTable;
}
```

### JDK 1.8 的实现原理 ###
JDK 1.7 已经解决了并发问题，并且能支持 N 个 `Segment` 这么多次数的并发，但依然存在 `HashMap` 在 1.7 版本中的问题。那么是什么问题呢？很明显那就是查询遍历链表效率太低。

因此 JDK 1.8 做了一些数据结构上的调整。在 JDK 1.8 中它摒弃了 `Segment` 的概念，而是启用了一种全新的方式实现，利用 `CAS` 算法。底层依然由“数组”+链表+红黑树的方式思想，但是为了做到并发，又增加了很多辅助的类，例如 `TreeBin`、`Traverser` 等对象内部类。

`ConcurrentHashMap` 包含了很多内部类，其中主要的内部类框架图如下图所示：
![JDK1.8中ConcurrentHashMap的框架](https://henleylee.github.io/medias/java/concurrent_hashmap_jdk_1_8_structure.png)

`ConcurrentHashMap` 的内部类非常的庞大，并且在 JDK 1.8 时新增加了很多的内部类，如下图所示：
![JDK1.8中ConcurrentHashMap新增内部类](https://henleylee.github.io/medias/java/concurrent_hashmap_jdk_1_8_add_class.png)

下面对其中主要的内部类进行分析和讲解：
 - **`Node：`**主要用于存储具体键值对，其子类有 `ForwardingNode`、`ReservationNode`、`TreeNode` 和 `TreeBin` 四个子类。四个子类具体的代码在之后的具体例子中进行分析讲解。
 - **`Traverser：`**主要用于遍历操作，其子类有 `BaseIterator`、`KeySpliterator`、`ValueSpliterator`、`EntrySpliterator` 四个类，`BaseIterator` 用于遍历操作。`KeySplitertor`、`ValueSpliterator`、`EntrySpliterator` 则用于键、值、键值对的划分。
 - **`CollectionView：`**是一个主要定义了视图操作的抽象类，其子类 `KeySetView`、`ValueSetView`、`EntrySetView` 分别表示键视图、值视图、键值对视图。对视图均可以进行操作。
 - **`Segment：`**在 JDK1.8 中与之前的版本的 JDK 作用存在很大的差别，JDK 1.8 下，其在普通的 `ConcurrentHashMap` 操作中已经没有失效，其在序列化与反序列化的时候会发挥作用。
 - **`CounterCell：`**主要用于对 `baseCount` 的计数。

#### 类中常量 ####
```java
public class ConcurrentHashMap<K, V> extends AbstractMap<K, V>
        implements ConcurrentMap<K, V>, Serializable {
    private static final long serialVersionUID = 7249069246763182397L;
    /** 表的最大容量 */
    private static final int MAXIMUM_CAPACITY = 1 << 30;
    /** 默认表的大小 */
    private static final int DEFAULT_CAPACITY = 16;
    /** 最大数组大小 */
    static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    /** 默认并发数 */
    private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
    /** 装载因子 */
    private static final float LOAD_FACTOR = 0.75f;
    /** 转化为红黑树的阈值 */
    static final int TREEIFY_THRESHOLD = 8;
    /** 由红黑树转化为链表的阈值 */
    static final int UNTREEIFY_THRESHOLD = 6;
    /** 转化为红黑树的表的最小容量 */
    static final int MIN_TREEIFY_CAPACITY = 64;
    /** 每次进行转移的最小值
    private static final int MIN_TRANSFER_STRIDE = 16;
    /** 生成sizeCtl所使用的bit位数 */
    private static int RESIZE_STAMP_BITS = 16;
    /** 进行扩容所允许的最大线程数 */
    private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;
    /** 记录sizeCtl中的大小所需要进行的偏移位数 */
    private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;
    /** 一系列的标识 */
    static final int MOVED = -1; // hash for forwarding nodes
    static final int TREEBIN = -2; // hash for roots of trees
    static final int RESERVED = -3; // hash for transient reservations
    static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash
    /** 获取可用的CPU个数 */
    static final int NCPU = Runtime.getRuntime().availableProcessors();

    /** 进行序列化的属性 */
    private static final ObjectStreamField[] serialPersistentFields = {
            new ObjectStreamField("segments", Segment[].class),
            new ObjectStreamField("segmentMask", Integer.TYPE),
            new ObjectStreamField("segmentShift", Integer.TYPE)
    };

    /** 表 */
    transient volatile Node<K, V>[] table;
    /** 下一个表 */
    private transient volatile Node<K, V>[] nextTable;
    /** 基本计数 */
    private transient volatile long baseCount;
    /** 对表初始化和扩容控制 */
    private transient volatile int sizeCtl;
    /** 扩容下另一个表的索引 */
    private transient volatile int transferIndex;
    /** 旋转锁 */
    private transient volatile int cellsBusy;
    /** counterCell表 */
    private transient volatile CounterCell[] counterCells;

    // views
    private transient KeySetView<K, V> keySet;
    private transient ValuesView<K, V> values;
    private transient EntrySetView<K, V> entrySet;

    // Unsafe mechanics
    private static final sun.misc.Unsafe U;
    private static final long SIZECTL;
    private static final long TRANSFERINDEX;
    private static final long BASECOUNT;
    private static final long CELLSBUSY;
    private static final long CELLVALUE;
    private static final long ABASE;
    private static final int ASHIFT;

    static {
        try {
            U = sun.misc.Unsafe.getUnsafe();
            Class<?> k = ConcurrentHashMap.class;
            SIZECTL = U.objectFieldOffset
                    (k.getDeclaredField("sizeCtl"));
            TRANSFERINDEX = U.objectFieldOffset
                    (k.getDeclaredField("transferIndex"));
            BASECOUNT = U.objectFieldOffset
                    (k.getDeclaredField("baseCount"));
            CELLSBUSY = U.objectFieldOffset
                    (k.getDeclaredField("cellsBusy"));
            Class<?> ck = CounterCell.class;
            CELLVALUE = U.objectFieldOffset
                    (ck.getDeclaredField("value"));
            Class<?> ak = Node[].class;
            ABASE = U.arrayBaseOffset(ak);
            int scale = U.arrayIndexScale(ak);
            if ((scale & (scale - 1)) != 0)
                throw new Error("data type scale not a power of two");
            ASHIFT = 31 - Integer.numberOfLeadingZeros(scale);
        } catch (Exception e) {
            throw new Error(e);
        }
    }

}
```

> `ConcurrentHashMap` 的属性很多，其中不少属性在 `HashMap` 中就已经介绍过，而对于 `ConcurrentHashMap` 而言，添加了 `sun.misc.Unsafe` 实例，主要用于反射获取对象相应的字段。

#### 构造函数 ####
```java
    /**
     * 该构造函数用于创建一个带有默认初始容量(16)、加载因子(0.75)和concurrencyLevel(16)的空映射
     */
    public ConcurrentHashMap() {
    }

    /**
     * 该构造函数用于创建一个带有指定初始容量、默认加载因子(0.75)和concurrencyLevel(16)的空映射
     */
    public ConcurrentHashMap(int initialCapacity) {
        // 初始容量小于0，抛出异常
        if (initialCapacity < 0)
            throw new IllegalArgumentException();
        // 找到最接近该容量的2的幂次方数
        int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
                MAXIMUM_CAPACITY :
                tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
        // 初始化
        this.sizeCtl = cap;
    }

    /**
     * 该构造函数用于构造一个与给定映射具有相同映射关系的新映射
     */
    public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
        this.sizeCtl = DEFAULT_CAPACITY;
        // 将集合m的元素全部放入
        putAll(m);
    }

    /**
     * 该构造函数用于创建一个带有指定初始容量、加载因子和默认 concurrencyLevel(1)的空映射
     */
    public ConcurrentHashMap(int initialCapacity, float loadFactor) {
        this(initialCapacity, loadFactor, 1);
    }

    /**
     * 该构造函数用于创建一个带有指定初始容量、加载因子和并发级别的空映射
     */
    public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
        // 合法性判断
        if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
            throw new IllegalArgumentException();
        if (initialCapacity < concurrencyLevel)   // Use at least as many bins
            initialCapacity = concurrencyLevel;   // as estimated threads
        long size = (long) (1.0 + (long) initialCapacity / loadFactor);
        int cap = (size >= (long) MAXIMUM_CAPACITY) ?
                MAXIMUM_CAPACITY : tableSizeFor((int) size);
        this.sizeCtl = cap;
    }
```

> 对于构造函数而言，会根据输入的 `initialCapacity` 的大小来确定一个最小的且大于等于 `initialCapacity` 大小的2的n次幂，如 `initialCapacity` 为15，则 `sizeCtl` 为16，若 `initialCapacity` 为16，则 `sizeCtl` 为16。若 `initialCapacity` 大小超过了允许的最大值，则 `sizeCtl` 为最大值。值得注意的是，构造函数中的 `concurrencyLevel` 参数已经在 JDK 1.8 中的意义发生了很大的变化，其并不代表所允许的并发数，其只是用来确定 `sizeCtl` 大小，在 JDK 1.8 中的并发控制都是针对具体的桶而言，即有多少个桶就可以允许多少个并发数。

#### put() 方法 ####
```java
public V put(K key, V value) {
    // 直接调用putVal()方法
    return putVal(key, value, false);
}

/** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    // 键或值为空，抛出异常
    if (key == null || value == null) throw new NullPointerException();
    // 键的hash值经过计算获得hash值
    int hash = spread(key.hashCode());
    int binCount = 0;
    // 无限循环
    for (Node<K, V>[] tab = table; ; ) {
        Node<K, V> f;
        int n, i, fh;
        // 表为空或者表的长度为0
        if (tab == null || (n = tab.length) == 0)
            // 初始化表
            tab = initTable();
        // 表不为空并且表的长度大于0，并且该桶不为空
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // 比较并且交换值，如tab的第i项为空则用新生成的node替换
            if (casTabAt(tab, i, null, new Node<K, V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        // 该结点的hash值为MOVED
        else if ((fh = f.hash) == MOVED)
            // 进行结点的转移（在扩容的过程中）
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            // 加锁同步
            synchronized (f) {
                // 找到table表下标为i的节点
                if (tabAt(tab, i) == f) {
                    // 该table表中该结点的hash值大于0
                    if (fh >= 0) {
                        // binCount赋值为1
                        binCount = 1;
                        for (Node<K, V> e = f; ; ++binCount) {
                            K ek;
                            // 结点的hash值相等并且key也相等
                            if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                            (ek != null && key.equals(ek)))) {
                                // 保存该结点的val值
                                oldVal = e.val;
                                // 进行判断
                                if (!onlyIfAbsent)
                                    // 将指定的value保存至结点，即进行了结点值的更新
                                    e.val = value;
                                break;
                            }
                            // 保存当前结点
                            Node<K, V> pred = e;
                            // 当前结点的下一个结点为空，即为最后一个结点
                            if ((e = e.next) == null) {
                                // 新生一个结点并且赋值给next域
                                pred.next = new Node<K, V>(hash, key,
                                        value, null);
                                // 退出循环
                                break;
                            }
                        }
                    }
                    // 结点为红黑树结点类型
                    else if (f instanceof TreeBin) {
                        Node<K, V> p;
                        // binCount赋值为2
                        binCount = 2;
                        // 将hash、key、value放入红黑树
                        if ((p = ((TreeBin<K, V>) f).putTreeVal(hash, key,
                                value)) != null) {
                            // 保存结点的val
                            oldVal = p.val;
                            // 进行判断
                            if (!onlyIfAbsent)
                                // 赋值结点value值
                                p.val = value;
                        }
                    }
                }
            }
            // binCount不为0
            if (binCount != 0) {
                // 如果binCount大于等于转化为红黑树的阈值
                if (binCount >= TREEIFY_THRESHOLD)
                    // 进行转化
                    treeifyBin(tab, i);
                // 旧值不为空
                if (oldVal != null)
                    // 返回旧值
                    return oldVal;
                break;
            }
        }
    }
    // 增加binCount的数量
    addCount(1L, binCount);
    return null;
}
```

**`put()`**方法底层调用了 `putVal()` 方法进行数据的插入，对于 `putVal()` 方法的流程大体如下：
 - ①、判断存储的 `key`、`value` 是否为空，若为空，则抛出异常，否则进入步骤②
 - ②、计算 `key` 的 `hash` 值，随后进入无限循环，该无限循环可以确保成功插入数据，若 `table` 表为空或者长度为 0，则初始化 `table` 表，否则进入步骤③
 - ③、根据 `key` 的 `hash` 值取出 `table` 表中的结点元素，若取出的结点为空(该桶为空)，则使用 `CAS` 将 `key`、`value`、`hash` 值生成的结点放入桶中，否则进入步骤④
 - ④、若该结点的的 `hash` 值为 `MOVED`，则对该桶中的结点进行转移，否则进入步骤⑤
 - ⑤、对桶中的第一个结点(即 `table` 表中的结点)进行加锁，对该桶进行遍历，桶中的结点的 `hash` 值与 `key` 值与给定的 `hash` 值和 `key` 值相等，则根据标识选择是否进行更新操作(用给定的 `value` 值替换该结点的 `value` 值)，若遍历完桶仍没有找到 `hash` 值与 `key` 值和指定的 `hash` 值与 `key` 值相等的结点，则直接新生一个结点并赋值为之前最后一个结点的下一个结点，进入步骤⑥
 - ⑥、若 `binCount` 值达到红黑树转化的阈值，则将桶中的结构转化为红黑树存储，最后增加 `binCount` 的值。

在 `putVal()` 方法中会涉及到如下几个函数：`initTable()`、`tabAt()`、`casTabAt()`、`helpTransfer()`、`putTreeVal()`、`treeifyBin()`、`addCount()` 方法，下面对其中涉及到的函数进行分析。

##### initTable() #####
`initTable()` 方法用于初始化 `table` 表。`initTable()` 方法的源码如下：
```java
private final Node<K, V>[] initTable() {
    Node<K, V>[] tab;
    int sc;
    // 无限循环
    while ((tab = table) == null || tab.length == 0) {
        // sizeCtl小于0，则进行线程让步等待
        if ((sc = sizeCtl) < 0)
            Thread.yield(); // lost initialization race; just spin
            // 比较sizeCtl的值与sc是否相等，相等则用-1替换
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                // table表为空或者大小为0
                if ((tab = table) == null || tab.length == 0) {
                    // sc的值是否大于0，若是，则n为sc，否则，n为默认初始容量
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    // 新生结点数组
                    @SuppressWarnings("unchecked")
                    Node<K, V>[] nt = (Node<K, V>[]) new Node<?, ?>[n];
                    // 赋值给table
                    table = tab = nt;
                    // sc为 n * 3/4
                    sc = n - (n >>> 2);
                }
            } finally {
                // 设置sizeCtl的值
                sizeCtl = sc;
            }
            break;
        }
    }
    // 返回table表
    return tab;
}
```
> 对于 `table` 的大小，会根据 `sizeCtl` 的值进行设置，如果没有设置 `szieCtl` 的值，那么默认生成的 `table` 大小为16，否则，会根据 `sizeCtl` 的大小设置 `table` 大小。

##### tabAt() #####
`tabAt()` 方法用于返回 `table` 数组中下标为 `i` 的结点。`tabAt()` 方法的源码如下：
```java
static final <K, V> Node<K, V> tabAt(Node<K, V>[] tab, int i) {
    return (Node<K, V>) U.getObjectVolatile(tab, ((long) i << ASHIFT) + ABASE);
}
```
> 可以看到是通过 `Unsafe`对象通过反射获取的，`getObjectVolatile()` 方法的第二项参数为下标为 `i` 的偏移地址。

##### casTabAt() #####
`casTabAt()` 方法用于比较 `table` 数组下标为 `i` 的结点是否为 `c`，若为 `c`，则用 `v` 交换操作，否则不进行交换操作。`casTabAt()` 方法的源码如下：
```java
static final <K, V> boolean casTabAt(Node<K, V>[] tab, int i,
                                     Node<K, V> c, Node<K, V> v) {
    return U.compareAndSwapObject(tab, ((long) i << ASHIFT) + ABASE, c, v);
}
```

##### helpTransfer() #####
`helpTransfer()` 方法用于在扩容时将 `table` 表中的结点转移到 `nextTable` 中。`helpTransfer()` 方法的源码如下：
```java
final Node<K, V>[] helpTransfer(Node<K, V>[] tab, Node<K, V> f) {
    Node<K,V>[] nextTab; int sc;
    // table表不为空并且结点类型使ForwardingNode类型，并且结点的nextTable不为空
    if (tab != null && (f instanceof ForwardingNode) &&
            (nextTab = ((ForwardingNode<K, V>) f).nextTable) != null) {
        int rs = resizeStamp(tab.length);
        // 条件判断
        while (nextTab == nextTable && table == tab &&
                (sc = sizeCtl) < 0) {
            if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || transferIndex <= 0)
                break;
            // 比较并交换
            if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                // 将table的结点转移到nextTab中
                transfer(tab, nextTab);
                break;
            }
        }
        return nextTab;
    }
    return table;
}
```

##### putTreeVal() #####
`putTreeVal()` 方法用于将指定的 `hash`、`key`、`value` 值添加到红黑树中，若已经添加则返回 `null`，否则返回该结点。`putTreeVal()` 方法的源码如下：
```java
final TreeNode<K, V> putTreeVal(int h, K k, V v) {
    Class<?> kc = null;
    boolean searched = false;
    for (TreeNode<K, V> p = root; ; ) {
        int dir, ph;
        K pk;
        if (p == null) {
            first = root = new TreeNode<K, V>(h, k, v, null, null);
            break;
        } else if ((ph = p.hash) > h)
            dir = -1;
        else if (ph < h)
            dir = 1;
        else if ((pk = p.key) == k || (pk != null && k.equals(pk)))
            return p;
        else if ((kc == null &&
                (kc = comparableClassFor(k)) == null) ||
                (dir = compareComparables(kc, k, pk)) == 0) {
            if (!searched) {
                TreeNode<K, V> q, ch;
                searched = true;
                if (((ch = p.left) != null &&
                        (q = ch.findTreeNode(h, k, kc)) != null) ||
                        ((ch = p.right) != null &&
                                (q = ch.findTreeNode(h, k, kc)) != null))
                    return q;
            }
            dir = tieBreakOrder(k, pk);
        }

        TreeNode<K, V> xp = p;
        if ((p = (dir <= 0) ? p.left : p.right) == null) {
            TreeNode<K, V> x, f = first;
            first = x = new TreeNode<K, V>(h, k, v, f, xp);
            if (f != null)
                f.prev = x;
            if (dir <= 0)
                xp.left = x;
            else
                xp.right = x;
            if (!xp.red)
                x.red = true;
            else {
                lockRoot();
                try {
                    root = balanceInsertion(root, x);
                } finally {
                    unlockRoot();
                }
            }
            break;
        }
    }
    assert checkInvariants(root);
    return null;
}
```

##### treeifyBin() #####
`treeifyBin()` 方法用于将桶中的数据结构转化为红黑树，其中值得注意的是，当 `table` 的长度未达到阈值时，会进行一次扩容操作，该操作会使得触发 `treeifyBin()` 操作的某个桶中的所有元素进行一次重新分配，这样可以避免某个桶中的结点数量太大。`treeifyBin()` 方法的源码如下：
```java
private final void treeifyBin(Node<K, V>[] tab, int index) {
    Node<K,V> b; int n, sc;
    // 表不为空
    if (tab != null) {
        // table表的长度小于最小的长度
        if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
            // 进行扩容，调整某个桶中结点数量过多的问题
            // 由于某个桶中结点数量超出了阈值，则触发treeifyBin
            tryPresize(n << 1);
        // 桶中存在结点并且结点的hash值大于等于0
        else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
            // 对桶中第一个结点进行加锁
            synchronized (b) {
                // 第一个结点没有变化
                if (tabAt(tab, index) == b) {
                    TreeNode<K, V> hd = null, tl = null;
                    // 遍历桶中所有结点
                    for (Node<K, V> e = b; e != null; e = e.next) {
                        // 新生一个TreeNode结点
                        TreeNode<K, V> p =
                                new TreeNode<K, V>(e.hash, e.key, e.val,
                                        null, null);
                        // 该结点前驱为空
                        if ((p.prev = tl) == null)
                            // 设置p为头结点
                            hd = p;
                        else
                            // 尾节点的next域赋值为p
                            tl.next = p;
                        // 尾节点赋值为p
                        tl = p;
                    }
                    // 设置table表中下标为index的值为hd
                    setTabAt(tab, index, new TreeBin<K, V>(hd));
                }
            }
        }
    }
}
```

##### addCount() #####
`addCount()` 方法用于完成 `binCount` 的值加1的操作。`addCount()` 方法的源码如下：
```java
private final void addCount(long x, int check) {
    CounterCell[] as; long b, s;
    // counterCells不为空或者比较交换失败
    if ((as = counterCells) != null ||
            !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
        CounterCell a;
        long v;
        int m;
        // 无竞争标识
        boolean uncontended = true;
        if (as == null || (m = as.length - 1) < 0 ||
                (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
                !(uncontended =
                        U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
            fullAddCount(x, uncontended);
            return;
        }
        if (check <= 1)
            return;
        s = sumCount();
    }
    if (check >= 0) {
        Node<K, V>[] tab, nt;
        int n, sc;
        while (s >= (long) (sc = sizeCtl) && (tab = table) != null &&
                (n = tab.length) < MAXIMUM_CAPACITY) {
            int rs = resizeStamp(n);
            if (sc < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            } else if (U.compareAndSwapInt(this, SIZECTL, sc,
                    (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
            s = sumCount();
        }
    }
}
```

#### get() 方法 ####
**`get()`**方法根据 `key` 的 `hash` 值来计算在哪个桶中，再遍历桶，查找元素，若找到则返回该结点，否则返回 `null`。`get()` 方法的源码如下：
```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    // 计算key的hash值
    int h = spread(key.hashCode());
    // 表不为空并且表的长度大于0并且key所在的桶不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) {
        // 表中的元素的hash值与key的hash值相等
        if ((eh = e.hash) == h) {
            // 键相等
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                // 返回值
                return e.val;
        }
        // 结点hash值小于0
        else if (eh < 0)
            // 在桶（链表/红黑树）中查找
            return (p = e.find(h, key)) != null ? p.val : null;
        // 对于结点hash值大于0的情况
        while ((e = e.next) != null) {
            if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

#### remove() 方法 ####
**`remove()`**方法实现结点的删除。`remove()` 方法的源码如下：
```java
public V remove(Object key) {
    return replaceNode(key, null, null);
}

final V replaceNode(Object key, V value, Object cv) {
    // 计算key的hash值
    int hash = spread(key.hashCode());
    // 无限循环
    for (Node<K, V>[] tab = table; ; ) {
        Node<K, V> f;
        int n, i, fh;
        // table表为空或者表长度为0或者key所对应的桶为空
        if (tab == null || (n = tab.length) == 0 ||
                (f = tabAt(tab, i = (n - 1) & hash)) == null)
            // 跳出循环
            break;
            // 桶中第一个结点的hash值为MOVED
        else if ((fh = f.hash) == MOVED)
            // 转移
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            boolean validated = false;
            // 加锁同步
            synchronized (f) {
                // 桶中的第一个结点没有发生变化
                if (tabAt(tab, i) == f) {
                    // 结点hash值大于0
                    if (fh >= 0) {
                        validated = true;
                        // 无限循环
                        for (Node<K, V> e = f, pred = null; ; ) {
                            K ek;
                            // 结点的hash值与指定的hash值相等，并且key也相等
                            if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                            (ek != null && key.equals(ek)))) {
                                V ev = e.val;
                                // cv为空或者与结点value相等或者不为空并且相等
                                if (cv == null || cv == ev ||
                                        (ev != null && cv.equals(ev))) {
                                    // 保存该结点的val值
                                    oldVal = ev;
                                    // value不为null
                                    if (value != null)
                                        // 设置结点value值
                                        e.val = value;
                                    // 前驱不为空
                                    else if (pred != null)
                                        // 前驱的后继为e的后继，即删除了e结点
                                        pred.next = e.next;
                                    else
                                        // 设置table表中下标为index的值为e.next
                                        setTabAt(tab, i, e.next);
                                }
                                break;
                            }
                            pred = e;
                            if ((e = e.next) == null)
                                break;
                        }
                    }
                    // 为红黑树结点类型
                    else if (f instanceof TreeBin) {
                        validated = true;
                        // 类型转化
                        TreeBin<K, V> t = (TreeBin<K, V>) f;
                        TreeNode<K, V> r, p;
                        // 根节点不为空并且存在与指定hash和key相等的结点
                        if ((r = t.root) != null &&
                                (p = r.findTreeNode(hash, key, null)) != null) {
                            // 保存p结点的value
                            V pv = p.val;
                            // cv为空或者与结点value相等或者不为空并且相等
                            if (cv == null || cv == pv ||
                                    (pv != null && cv.equals(pv))) {
                                oldVal = pv;
                                if (value != null)
                                    p.val = value;
                                // 移除p结点
                                else if (t.removeTreeNode(p))
                                    setTabAt(tab, i, untreeify(t.first));
                            }
                        }
                    }
                }
            }
            if (validated) {
                if (oldVal != null) {
                    if (value == null)
                        // baseCount值减一
                        addCount(-1L, -1);
                    return oldVal;
                }
                break;
            }
        }
    }
    return null;
}
```

## 总结 ##
`ConcurrentHashMap` 是一个并发散列映射表的实现，它允许完全并发的读取，并且支持给定数量的并发更新。相比于 `HashTable` 和用同步包装器(`Collections.synchronizedMap(new HashMap())`)包装的 `HashMap`，`ConcurrentHashMap` 拥有更高的并发性。在 `HashTable` 和由同步包装器包装的 `HashMap` 中，使用一个全局的锁来同步不同线程间的并发访问。同一时间点，只能有一个线程持有锁，也就是说在同一时间点，只能有一个线程能访问容器。这虽然保证多线程间的安全并发访问，但同时也导致对容器的访问变成串行化的了。

在使用锁来协调多线程间并发访问的模式下，减小对锁的竞争可以有效提高并发性。有两种方式可以减小对锁的竞争：
 - 减小请求同一个锁的频率。
 - 减少持有锁的 时间。

`ConcurrentHashMap` 的高并发性主要来自于三个方面：
 - 用分离锁实现多个线程间的更深层次的共享访问。
 - 用 `HashEntery` 对象的不变性来降低执行读操作的线程在遍历链表期间对加锁的需求。
 - 通过对同一个 `volatile` 变量的写/读访问，协调不同线程间读/写操作的内存可见性。

通过 `HashEntery` 对象的不变性及对同一个 `volatile` 变量的读/写来协调内存可见性，使得读操作大多数时候不需要加锁就能成功获取到需要的值。由于散列映射表在实际应用中大多数操作都是成功的读操作，所以 2 和 3 既可以减少请求同一个锁的频率，也可以有效减少持有锁的时间。

通过使用分离锁，减小请求同一个锁的频率和尽量减少持有锁的时间，使得 `ConcurrentHashMap` 的并发性相对于 `HashTable` 和用同步包装器包装的 `HashMap` 有了质的提高。



