---
title: Java 数据结构
categories: Java
tags:
  - 数据结构
abbrlink: cbe5dc46
date: 2018-11-18 18:32:28
---

## 数据结构 ##
数据结构是计算机存储、组织数据的方式。数据结构是指相互之间存在一种或多种特定关系的数据元素的集合。通常情况下，精心选择的数据结构可以带来更高的运行或者存储效率。数据结构往往同高效的检索算法和索引技术有关。

### 数据结构的基本功能 ###
不同的数据结构其操作集不同，但下列基本功能必不可缺：
 - 如何插入一条新的数据项；
 - 如何寻找某一特定的数据项；
 - 如何删除某一特定的数据项；
 - 如何迭代的访问各个数据项，以便进行显示或其他操作。

### 常见的数据结构 ###
常见的数据结构包含：数组(Array)、栈(Stack)、队列(Queue)、链表(Linked List)、树(Tree)、哈希表(Hash)、堆(Heap)、图(Graph)。

下表示常见的几种数据结构的特性：

| 数据结构 | 优点                                                       | 缺点                                                     |
|----------|------------------------------------------------------------|----------------------------------------------------------|
| 数组     | 插入快，如果知道下标，可以非常快地存取                     | 查找慢，删除慢，大小固定，只能存储单一元素               |
| 有序数组 | 比无序的数据查找快                                         | 插入慢，删除慢，大小固定，只能存储单一元素               |
| 栈       | 提供后进先出方式的存取                                     | 存取其他项很慢                                           |
| 队列     | 提供先进先出方式的存取                                     | 存取其他项很慢                                           |
| 链表     | 插入快，删除快                                             | 查找慢                                                   |
| 二叉树   | 如果树是平衡的，查找、插入、删除都快                       | 删除算法复杂                                             |
| 红黑树   | 查找、插入、删除都快。树总是平衡的                         | 算法复杂                                                 |
| 2-3-4树  | 查找、插入、删除都快。树总是平衡的。类似的树对磁盘存储有用 | 算法复杂                                                 |
| 哈希表   | 如果关键字已知则存取极快。插入快                           | 删除慢，如果不知道关键字则存取很慢，对存储空间使用不充分 |
| 堆       | 插入、删除快，对最大数据项的存取很快                       | 对其他数据项存取慢                                       |
| 图       | 对现实世界建模                                             | 有些算法且复杂                                           |

## Java 数据结构 ##
Java 工具包提供了强大的数据结构。在 Java 中的数据结构主要包括以下几种接口和类：
 - **`枚举(Enumeration)：`**Enumeration 接口中定义了一些方法，通过这些方法可以枚举(一次获得一个)对象集合中的元素。
 - **`位集合(BitSet)：`**BitSet 实现了一组可以单独设置和清除的位或标志。
 - **`向量(Vector)：`**Vector 类实现了一个动态数组。
 - **`栈(Stack)：`**Stack 实现了一个后进先出(LIFO)的数据结构。
 - **`字典(Dictionary)：`**Dictionary 定义了键映射到值的数据结构。
 - **`哈希表(Hashtable)：`**Hashtable 提供了一种在用户定义键结构的基础上来组织数据的手段。
 - **`属性(Properties)：`**Properties 表示一个持久的属性集。

### 枚举(Enumeration) ###
**`枚举(Enumeration)`**接口虽然它本身不属于数据结构，但它在其他数据结构的范畴里应用很广。 枚举(Enumeration)接口中定义了一些方法，通过这些方法可以枚举(一次获得一个)对象集合中的元素。
这种传统接口已被迭代器取代，虽然 Enumeration 还未被遗弃，但在现代代码中已经被很少使用了。尽管如此，它还是使用在诸如 Vector 和 Properties 这些传统类所定义的方法中，除此之外，还用在一些 API 类，并且在应用程序中也广泛被使用。 下表总结了一些 Enumeration 声明的方法：

| 方法                        | 描述                                                           |
|-----------------------------|----------------------------------------------------------------|
| `boolean hasMoreElements()` | 判断此枚举是否包含更多的元素                                   |
| `Object nextElement()`      | 如果此枚举对象至少还有一个可提供的元素，则返回下一个元素的枚举 |

> 枚举可用作`常量`、`swicth 语句`，枚举中也可以`添加参数和方法`，每个枚举对象可以用作一个带方法和属性的实例对象。

### 位集合(BitSet) ###
**`位集合(BitSet)`**实现了一组可以单独设置和清除的位或标志。该类在处理一组布尔值的时候非常有用，只需要给每个值赋值一"位"，然后对位进行适当的设置或清除，就可以对布尔值进行操作了。

一个 Bitset 类创建一种特殊类型的数组来保存位值。BitSet 中数组大小会随需要增加。

BitSet 定义了两个构造方法：
 - 第一个构造方法创建一个默认的对象：
    ```java
    BitSet()
    ```

 - 第二个方法允许用户指定初始大小，所有位初始化为0：
    ```java
    BitSet(int size)
    ```

BitSet 中实现了 Cloneable 接口中定义的方法如下表所列：

| 方法                                                | 描述                                                                                     |
|-----------------------------------------------------|------------------------------------------------------------------------------------------|
| `void and(BitSet set)`                              | 对此目标位 set 和参数位 set 执行逻辑与操作                                               |
| `void andNot(BitSet set)`                           | 清除此 BitSet 中所有的位，其相应的位在指定的 BitSet 中已设置                             |
| `int cardinality()`                                 | 返回此 BitSet 中设置为 true 的位数                                                       |
| `void clear()`                                      | 将此 BitSet 中的所有位设置为 false                                                       |
| `void clear(int index)`                             | 将索引指定处的位设置为 false                                                             |
| `void clear(int startIndex, int endIndex)`          | 将指定的 fromIndex(包括)到指定的 toIndex(不包括)范围内的位设置为 false                   |
| `Object clone()`                                    | 复制此 BitSet，生成一个与之相等的新 BitSet                                               |
| `boolean equals(Object bitSet)`                     | 将此对象与指定的对象进行比较                                                             |
| `void flip(int index)`                              | 将指定索引处的位设置为其当前值的补码                                                     |
| `void flip(int startIndex, int endIndex)`           | 将指定的 fromIndex(包括)到指定的 toIndex(不包括)范围内的每个位设置为其当前值的补码       |
| `boolean get(int index)`                            | 返回指定索引处的位值                                                                     |
| `BitSet get(int startIndex, int endIndex)`          | 返回一个新的 BitSet，它由此 BitSet 中从 fromIndex(包括)到 toIndex(不包括)范围内的位组成  |
| `boolean intersects(BitSet bitSet)`                 | 如果指定的 BitSet 中有设置为 true 的位，并且在此 BitSet 中也将其设置为 true，则返回 true |
| `boolean isEmpty()`                                 | 如果此 BitSet 中没有包含任何设置为 true 的位，则返回 true                                |
| `int length()`                                      | 返回此 BitSet 的"逻辑大小"：BitSet 中最高设置位的索引加 1                                |
| `int nextClearBit(int startIndex)`                  | 返回第一个设置为 false 的位的索引，这发生在指定的起始索引或之后的索引上                  |
| `int nextSetBit(int startIndex)`                    | 返回第一个设置为 true 的位的索引，这发生在指定的起始索引或之后的索引上                   |
| `void or(BitSet bitSet)`                            | 对此位 set 和位 set 参数执行逻辑或操作                                                   |
| `void set(int index)`                               | 将指定索引处的位设置为 true                                                              |
| `void set(int index, boolean v)`                    | 将指定索引处的位设置为指定的值                                                           |
| `void set(int startIndex, int endIndex)`            | 将指定的 fromIndex(包括)到指定的 toIndex(不包括)范围内的位设置为 true                    |
| `void set(int startIndex, int endIndex, boolean v)` | 将指定的 fromIndex(包括)到指定的 toIndex(不包括)范围内的位设置为指定的值                 |
| `int size()`                                        | 返回此 BitSet 表示位值时实际使用空间的位数                                               |
| `void xor(BitSet bitSet)`                           | 对此位 set 和位 set 参数执行逻辑异或操作                                                 |
| `void xor(BitSet bitSet)`                           | 对此位 set 和位 set 参数执行逻辑异或操作                                                 |
| `String toString()`                                 | 返回此位 set 的字符串表示形式                                                            |

### 向量(Vector) ###
**`向量(Vector)`**类实现了一个动态数组。和ArrayList和相似，但是两者是不同的：
 - Vector 是同步访问的。
 - Vector 包含了许多传统的方法，这些方法不属于集合框架。

Vector 主要用在事先不知道数组的大小，或者只是需要一个可以改变大小的数组的情况。

Vector 类支持4种构造方法：
 - 第一种构造方法创建一个默认的向量，默认大小为10：
    ```java
    Vector()
    ```

 - 第二种构造方法创建指定大小的向量：
    ```java
    Vector(int size)
    ```

 - 第三种构造方法创建指定大小的向量，并且增量用 incr 指定. 增量表示向量每次增加的元素数目：
    ```java
    Vector(int size,int incr)
    ```

 - 第四种构造方法创建一个包含集合 c 元素的向量：
    ```java
    Vector(Collection c)
    ```

除了从父类继承的方法外 Vector 还定义了以下方法：

| 方法                                                     | 描述                                                                                                   |
|----------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| `void add(int index, Object element)`                    | 在此向量的指定位置插入指定的元素                                                                       |
| `boolean add(Object o)`                                  | 将指定元素添加到此向量的末尾                                                                           |
| `boolean addAll(Collection c)`                           | 将指定 Collection 中的所有元素添加到此向量的末尾，按照指定 collection 的迭代器所返回的顺序添加这些元素 |
| `boolean addAll(int index, Collection c)`                | 在指定位置将指定 Collection 中的所有元素插入到此向量中                                                 |
| `void addElement(Object obj)`                            | 将指定的组件添加到此向量的末尾，将其大小增加 1                                                         |
| `int capacity()`                                         | 返回此向量的当前容量                                                                                   |
| `void clear()`                                           | 从此向量中移除所有元素                                                                                 |
| `Object clone()`                                         | 返回向量的一个副本                                                                                     |
| `boolean contains(Object elem)`                          | 如果此向量包含指定的元素，则返回 true                                                                  |
| `boolean containsAll(Collection c)`                      | 如果此向量包含指定 Collection 中的所有元素，则返回 true                                                |
| `void copyInto(Object[] anArray)`                        | 将此向量的组件复制到指定的数组中                                                                       |
| `Object elementAt(int index)`                            | 返回指定索引处的组件                                                                                   |
| `Enumeration elements()`                                 | 返回此向量的组件的枚举                                                                                 |
| `void ensureCapacity(int minCapacity)`                   | 增加此向量的容量(如有必要)，以确保其至少能够保存最小容量参数指定的组件数                               |
| `boolean equals(Object o)`                               | 比较指定对象与此向量的相等性                                                                           |
| `Object firstElement()`                                  | 返回此向量的第一个组件(位于索引 0) 处的项)                                                             |
| `Object get(int index)`                                  | 返回向量中指定位置的元素                                                                               |
| `int hashCode()`                                         | 返回此向量的哈希码值                                                                                   |
| `int indexOf(Object elem)`                               | 返回此向量中第一次出现的指定元素的索引，如果此向量不包含该元素，则返回 -1                              |
| `int indexOf(Object elem, int index)`                    | 返回此向量中第一次出现的指定元素的索引，从 index 处正向搜索，如果未找到该元素，则返回 -1               |
| `void insertElementAt(Object obj, int index)`            | 将指定对象作为此向量中的组件插入到指定的 index 处                                                      |
| `boolean isEmpty()`                                      | 测试此向量是否不包含组件                                                                               |
| `Object lastElement()`                                   | 返回此向量的最后一个组件                                                                               |
| `int lastIndexOf(Object elem)`                           | 返回此向量中最后一次出现的指定元素的索引；如果此向量不包含该元素，则返回 -1                            |
| `int lastIndexOf(Object elem, int index)`                | 返回此向量中最后一次出现的指定元素的索引，从 index 处逆向搜索，如果未找到该元素，则返回 -1             |
| `Object remove(int index)`                               | 移除此向量中指定位置的元素                                                                             |
| `boolean remove(Object o)`                               | 移除此向量中指定元素的第一个匹配项，如果向量不包含该元素，则元素保持不变                               |
| `boolean removeAll(Collection c)`                        | 从此向量中移除包含在指定 Collection 中的所有元素                                                       |
| `void removeAllElements()`                               | 从此向量中移除全部组件，并将其大小设置为零                                                             |
| `boolean removeElement(Object obj)`                      | 从此向量中移除变量的第一个(索引最小的)匹配项                                                           |
| `void removeElementAt(int index)`                        | 删除指定索引处的组件                                                                                   |
| `protected void removeRange(int fromIndex, int toIndex)` | 从此 List 中移除其索引位于 fromIndex(包括)与 toIndex(不包括)之间的所有元素                             |
| `boolean retainAll(Collection c)`                        | 在此向量中仅保留包含在指定 Collection 中的元素                                                         |
| `Object set(int index, Object element)`                  | 用指定的元素替换此向量中指定位置处的元素                                                               |
| `void setElementAt(Object obj, int index)`               | 将此向量指定 index 处的组件设置为指定的对象                                                            |
| `void setSize(int newSize)`                              | 设置此向量的大小                                                                                       |
| `int size()`                                             | 返回此向量中的组件数                                                                                   |
| `List subList(int fromIndex, int toIndex)`               | 返回此 List 的部分视图，元素范围为从 fromIndex(包括)到 toIndex(不包括)                                 |
| `Object[] toArray()`                                     | 返回一个数组，包含此向量中以恰当顺序存放的所有元素                                                     |
| `Object[] toArray(Object[] a)`                           | 返回一个数组，包含此向量中以恰当顺序存放的所有元素；返回数组的运行时类型为指定数组的类型               |
| `String toString()`                                      | 返回此向量的字符串表示形式，其中包含每个元素的 String 表示形式                                         |
| `void trimToSize()`                                      | 对此向量的容量进行微调，使其等于向量的当前大小                                                         |

### 栈(Stack) ###
**`栈(Stack)`**是 Vector 的一个子类，实现了一个后进先出(LIFO)的数据结构。

可以把栈理解为对象的垂直分布的栈，当添加一个新元素时，就将新元素放在其他元素的顶部。当从栈中取元素的时候，就从栈顶取一个元素。换句话说，最后进栈的元素最先被取出。

堆栈只定义了默认构造函数，用来创建一个空栈：
```java
Stack()
```

除了由 Vector 定义的所有方法，Stack 还定义了以下方法：：

| 方法                          | 描述                                           |
|-------------------------------|------------------------------------------------|
| `boolean empty()`             | 测试堆栈是否为空                               |
| `Object peek()`               | 查看堆栈顶部的对象，但不从堆栈中移除它         |
| `Object pop()`                | 移除堆栈顶部的对象，并作为此函数的值返回该对象 |
| `Object push(Object element)` | 把项压入堆栈顶部                               |
| `int search(Object element)`  | 返回对象在堆栈中的位置，以 1 为基数            |

### 字典(Dictionary) ###
**`字典(Dictionary)`**类是一个抽象类，用来存储键/值对，作用和 Map 类相似。

Dictionary 定义的抽象方法如下表所示：

| 方法                                   | 描述                                          |
|----------------------------------------|-----------------------------------------------|
| `Enumeration elements()`               | 返回此 dictionary 中值的枚举                  |
| `Object get(Object key)`               | 返回此 dictionary 中该键所映射到的值          |
| `boolean isEmpty()`                    | 测试此 dictionary 是否不存在从键到值的映射    |
| `Enumeration keys()`                   | 返回此 dictionary 中的键的枚举                |
| `Object put(Object key, Object value)` | 将指定 key 映射到此 dictionary 中指定 value   |
| `Object remove(Object key)`            | 从此 dictionary 中移除 key (及其相应的 value) |
| `int size()`                           | 返回此 dictionary 中条目(不同键)的数量        |

> Dictionary 类已经过时了。在实际开发中，你可以实现 Map 接口来获取键/值的存储功能。

### 哈希表(Hashtable) ###
**`哈希表(Hashtable)`**类提供了一种在用户定义键结构的基础上来组织数据的手段。Hashtable 是原始的 java.util 的一部分，是一个 Dictionary 具体的实现。

哈希表键的具体含义完全取决于哈希表的使用情景和它包含的数据。Hashtable 现在集成到了集合框架中。它和 HashMap 类很相似，但是它支持同步。

像 HashMap 一样，Hashtable 在哈希表中存储键/值对。当使用一个哈希表，要指定用作键的对象，以及要链接到该键的值。然后，该键经过哈希处理，所得到的散列码被用作存储在该表中值的索引。

Hashtable 定义了四个构造方法：
 - 第一个是默认构造方法：
    ```java
    Hashtable()
    ```

 - 第二个构造函数创建指定大小的哈希表：
    ```java
    Hashtable(int size)
    ```

 - 第三个构造方法创建了一个指定大小的哈希表，并且通过 fillRatio 指定填充比例(填充比例必须介于0.0和1.0之间，它决定了哈希表在重新调整大小之前的充满程度)：
    ```java
    Hashtable(int size,float fillRatio)
    ```

 - 第四个构造方法创建了一个以M中元素为初始化元素的哈希表(哈希表的容量被设置为M的两倍)：
    ```java
    Hashtable(Map m)
    ```

Hashtable 中除了从 Map 接口中定义的方法外，还定义了以下方法：

| 方法                                   | 描述                                                                 |
|----------------------------------------|----------------------------------------------------------------------|
| `void clear()`                         | 将此哈希表清空，使其不包含任何键                                     |
| `Object clone()`                       | 创建此哈希表的浅表副本                                               |
| `boolean contains(Object value)`       | 测试此映射表中是否存在与指定值关联的键                               |
| `boolean containsKey(Object key)`      | 测试指定对象是否为此哈希表中的键                                     |
| `boolean containsValue(Object value)`  | 如果此 Hashtable 将一个或多个键映射到此值，则返回 true               |
| `Enumeration elements()`               | 返回此哈希表中的值的枚举                                             |
| `Object get(Object key)`               | 返回指定键所映射到的值，如果此映射不包含此键的映射，则返回 null      |
| `boolean isEmpty()`                    | 测试此哈希表是否没有键映射到值                                       |
| `Enumeration keys()`                   | 返回此哈希表中的键的枚举                                             |
| `Object put(Object key, Object value)` | 将指定 key 映射到此哈希表中的指定 value                              |
| `void rehash()`                        | 增加此哈希表的容量并在内部对其进行重组，以便更有效地容纳和访问其元素 |
| `Object remove(Object key)`            | 从哈希表中移除该键及其相应的值                                       |
| `int size()`                           | 返回此哈希表中的键的数量                                             |

### 属性(Properties) ###
**`属性(Properties)`**继承于 Hashtable。Properties 类表示了一个持久的属性集。属性列表中每个键及其对应值都是一个字符串。

Properties 类被许多 Java 类使用。例如，在获取环境变量时它就作为 System.getProperties() 方法的返回值。

Properties 定义如下实例变量，这个变量持有一个 Properties 对象相关的默认属性列表：
```java
Properties defaults;
```

Properties 类定义了两个构造方法：
 - 第一个构造方法没有默认值：
    ```java
    Properties()
    ```
 - 第二个构造方法使用propDefault 作为默认值：
    ```java
    Properties(Properties propDefault)
    ```

除了从 Hashtable 中所定义的方法，Properties 定义了以下方法：

| 方法                                                     | 描述                                                                                                                |
|----------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------|
| `String getProperty(String key)`                         | 用指定的键在此属性列表中搜索属性                                                                                    |
| `String getProperty(String key, String defaultProperty)` | 用指定的键在属性列表中搜索属性                                                                                      |
| `void list(PrintStream streamOut)`                       | 将属性列表输出到指定的输出流                                                                                        |
| `void list(PrintWriter streamOut)`                       | 将属性列表输出到指定的输出流                                                                                        |
| `void load(InputStream streamIn)`                        | 从输入流中读取属性列表(键和元素对)                                                                                  |
| `Enumeration propertyNames()`                            | 按简单的面向行的格式从输入字符流中读取属性列表(键和元素对)                                                          |
| `Object setProperty(String key, String value)`           | 调用 Hashtable 的方法 put                                                                                           |
| `void store(OutputStream streamOut, String description)` | 以适合使用  load(InputStream)方法加载到 Properties 表中的格式，将此 Properties 表中的属性列表(键和元素对)写入输出流 |

