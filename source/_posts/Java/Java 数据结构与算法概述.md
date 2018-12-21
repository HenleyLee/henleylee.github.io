---
title: Java 数据结构与算法概述
categories: Java
tags:
  - Java
  - 数据结构
  - 算法
abbrlink: 8359917f
date: 2018-11-20 18:32:28
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

## 算法 ##
算法(Algorithm)是指解题方案的准确而完整的描述，是一系列解决问题的清晰指令，算法代表着用系统的方法描述解决问题的策略机制。也就是说，能够对一定规范的输入，在有限时间内获得所要求的输出。如果一个算法有缺陷，或不适合于某个问题，执行这个算法将不会解决这个问题。不同的算法可能用不同的时间、空间或效率来完成同样的任务。一个算法的优劣可以用空间复杂度与时间复杂度来衡量。

在 Java 中，算法通常都是由类的方法来实现的。前面的数据结构，比如链表为啥插入、删除快，而查找慢，平衡的二叉树插入、删除、查找都快，这都是实现这些数据结构的算法所造成的。后面我们讲的各种排序实现也是算法范畴的重要领域。

### 算法的五个特征 ###
一个算法应该具有以下五个重要的特征：
 - **`有穷性(Finiteness)：`**算法的有穷性是指算法必须能在执行有限个步骤之后终止。
 - **`确定性(Definiteness)：`**在每种情况下所应执行的操作，在算法中都有确切的规定，使算法的执行者或阅读者都能明确其含义及如何执行，并且在任何条件下，算法都只有一条执行路径。
 - **`可行性(Effectiveness)：`**算法中执行的任何计算步骤都是可以被分解为基本的可执行的操作步，即每个计算步都可以在有限时间内完成(也称之为有效性)。
 - **`有输入(Input)：`**一个算法有0个或多个输入，以刻画运算对象的初始情况，所谓0个输入是指算法本身定出了初始条件。
 - **`有输出(Output)：`**个算法有一个或多个输出，以反映对输入数据加工后的结果，没有输出的算法是毫无意义的。

### 算法的设计原则 ###
一个算法应该具有以下四个重要的设计原则：
 - 正确性：算法至少应该具有输入、输出和加工处理无歧义性、能正确反映问题的需要、能够得到问题的正确答案。
> 算法的“正确”通常在用法上有很大的差别，大体分为以下4个层次：
> ① 算法程序没有语法错误；
> ② 算法程序能够根据正确的输入的值得到满足要求的输出结果；
> ③ 算法程序能够根据错误的输出的值满足规格说明的输出结果；
> ④ 算法程序对于精心设计、极其刁难的测试数据都能满足要求的输出结果。
> 对于这4层含义，层次 `①` 要求最低，因为仅仅没有语法错误实在谈不上是好的算法。而层次 `④` 是最困难的，人们几乎不可能逐一验证所有的输入都得到正确的结果。
> 因此，算法的正确性在大部分情况下都不可能用程序来证明，而是用数学方法证明的。证明一个复杂算法在所有层次上都是正确的，代价非常昂贵。所以一般情况下，人们把层次 `③` 作为一个算法是否正确的标准。
 - 可读性：算法为了人的阅读与交流，其次才是计算机执行。因此算法应该易于人的理解；另一方面，晦涩难懂的程序易于隐藏较多的错误而难以调试。
 - 健壮性：当输入的数据非法时，算法应当恰当的做出反应或进行相应处理，而不是产生莫名其妙的输出结果。并且，处理出错的方法不应是中断程序执行，而是应当返回一个表示错误或错误性质的值，以便在更高的抽象层次上进行处理。
 - 高效与低存储：通常算法效率值得是算法执行时间；存储量是指算法执行过程中所需要的最大存储空间，两者都与问题的规模有关。
> 在满足以上几点以后，还可以考虑对算法进程进一步优化，尽量满足时间效率高和空间存储量低的需求。

### 算法的设计步骤 ###
算法设计的一般过程可以归纳为以下几个步骤：
 - 建立数学模型；
 - 通过对问题进行详细的分析，抽象出相应的数学模型；
 - 确定数据结构与算法；
 - 确定使用的数据结构，并在此基础上设计对此数据结构实施各种操作的算法；
 - 选用语言；
 - 选用某种语言将算法转化成程序；
 - 调试并运行；
 - 调试并运行这些程序。

### 算法的基本思想 ###
#### 穷举算法思想 ####
穷举算法思想就是从所有的可能结果中一个一个的试验，知道试出正确的结果。具体的操作步骤如下：
1. 对每一种可能的结果，计算其结果；
2. 判断结果是否符合题目要求，如果符合则该结果正确，如果不符合则继续进行第1步骤。

> 穷举算法思想的经典例子为鸡兔同笼为题（又称龟鹤同笼问题）。

#### 递推算法思想 ####
递推算法算法就是根据已知条件，利用特定关系推导出中间推论，直到得到结果的算法。其执行过程如下：
1. 根据已知结果和关系，求解中间结果。
2. 判断是否达到要求，如果没有达到，则继续根据已知结果和关系求解中间结果。如果满足要求，则表示寻找到一个正确答案。

> 递推算法思想最经典的例子是斐波那契数列 : 1,1,2,3,5,8,13......

#### 递归算法思想 ####
递归算法思想是把大问题转换成同类问题的子问题，然后递归调用函数表示问题的解。在使用递归的时候一定要注意调回递归函数的终止条件。

递归算法的分类：
 - 直接递归：在函数中调用自身。
 - 间接递归：在函数中调用另外一个函数，然后在另外一个函数中再调用该函数(用得不多)。

> 递归算法比较经典的例子是求阶乘。

#### 分治算法思想 ####
分治算法思想就是把一个大问题分解成若干个规模较小的子问题，且这些子问题的都是相互独立的、与原问题性质一致。逐个求出这些子问题的解就能够得到原问题的解了。其执行过程如下：
1. 对于一个规模为 N 的问题，若该问题可以容易地解决（比如说规模 N 较小），则直接解决，否则执行下面的步骤。
2. 将该问题分解为 M 个规模较小的子问题，这些子问题互相独立，并且与原问题形式相同。
3. 递归的解子问题。
4. 然后，将各子问题的解合并到原问题的解。

> 分治算法思想有一个比较经典的例子就是查找假币问题。

#### 概率算法思想 ####
概率算法主要包括四种算法：
 - 数值概率算法：数值问题的求解，最优化问题的近似解。
 - 蒙特卡罗算法：判定问题的准确解，不一定正确。
 - 拉斯维加斯算法：不一定会得到解，但得到的解一定是正确解。
 - 舍伍德算法：总能求得一个解，且一定是正确解。

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

