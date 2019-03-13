---
title: Java ThreadLocal 详解
categories: Java
tags:
  - Java
abbrlink: 27435a3e
date: 2019-01-31 12:36:25
---

## 简介 ##
`java.lang.ThreadLocal` 表示**线程本地存储区(Thread Local Storage，简称为 TLS)**，每个线程都有自己的私有的本地存储区域，不同线程之间彼此不能访问对方的 TLS 区域。`ThreadLocal` 是 JDK 1.2 开始提供的，该类为解决多线程的并发问题提供了一种新的思路。使用这个工具类可以很简洁地编写出优美的多线程程序。

当使用 `ThreadLocal` 维护变量时，`ThreadLocal` 为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。

从线程的角度看，目标变量就象是线程的本地变量，这也是类名中 `Local` 所要表达的意思。所以，在 Java 中编写线程局部变量的代码相对来说要笨拙一些，因此造成线程局部变量没有在 Java 开发者中得到很好的普及。

## 原理 ##
`Thread` 类中成员变量 `threadLocals`，它的类型是 `ThreadLocal.ThreadLocalMap`，每个线程都有一个这样的 `Map`，所以可以保存 N 个 `ThreadLocal` 键值对，并且不同线程的变量值不同。

`ThreadLocal` 连接 `ThreadLocalMap` 和 `Thread`，来处理 `Thread` 的 `threadLocals` 属性，包括初始化属性赋值、获取变量、设置变量等。通过当前线程，获取线程上的 `ThreadLocalMap`，对数据进行 `get`、`set` 等操作。

`ThreadLocalMap` 用来存储数据，存储了以 `ThreadLocal` 为 `key`，需要隔离的数据为 `value` 的 `Entry` 键值对数组结构。

## 源码解析 ##
`ThreadLocal` 的源码如下：
```java
public class ThreadLocal<T> {
    /**
     * ThreadLocals rely on per-thread linear-probe hash maps attached
     * to each thread (Thread.threadLocals and
     * inheritableThreadLocals).  The ThreadLocal objects act as keys,
     * searched via threadLocalHashCode.  This is a custom hash code
     * (useful only within ThreadLocalMaps) that eliminates collisions
     * in the common case where consecutively constructed ThreadLocals
     * are used by the same threads, while remaining well-behaved in
     * less common cases.
     */
    private final int threadLocalHashCode = nextHashCode();

    /**
     * The next hash code to be given out. Updated atomically. Starts at
     * zero.
     */
    private static AtomicInteger nextHashCode =
        new AtomicInteger();

    /**
     * The difference between successively generated hash codes - turns
     * implicit sequential thread-local IDs into near-optimally spread
     * multiplicative hash values for power-of-two-sized tables.
     */
    private static final int HASH_INCREMENT = 0x61c88647;

    /**
     * Returns the next hash code.
     */
    private static int nextHashCode() {
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }

    /**
     * Returns the current thread's "initial value" for this
     * thread-local variable.  This method will be invoked the first
     * time a thread accesses the variable with the {@link #get}
     * method, unless the thread previously invoked the {@link #set}
     * method, in which case the {@code initialValue} method will not
     * be invoked for the thread.  Normally, this method is invoked at
     * most once per thread, but it may be invoked again in case of
     * subsequent invocations of {@link #remove} followed by {@link #get}.
     *
     * <p>This implementation simply returns {@code null}; if the
     * programmer desires thread-local variables to have an initial
     * value other than {@code null}, {@code ThreadLocal} must be
     * subclassed, and this method overridden.  Typically, an
     * anonymous inner class will be used.
     *
     * @return the initial value for this thread-local
     */
    protected T initialValue() {
        return null;
    }

    /**
     * Creates a thread local variable. The initial value of the variable is
     * determined by invoking the {@code get} method on the {@code Supplier}.
     *
     * @param <S> the type of the thread local's value
     * @param supplier the supplier to be used to determine the initial value
     * @return a new thread local variable
     * @throws NullPointerException if the specified supplier is null
     * @since 1.8
     */
    public static <S> ThreadLocal<S> withInitial(Supplier<? extends S> supplier) {
        return new SuppliedThreadLocal<>(supplier);
    }

    /**
     * Creates a thread local variable.
     * @see #withInitial(java.util.function.Supplier)
     */
    public ThreadLocal() {
    }

    /**
     * Returns the value in the current thread's copy of this
     * thread-local variable.  If the variable has no value for the
     * current thread, it is first initialized to the value returned
     * by an invocation of the {@link #initialValue} method.
     *
     * @return the current thread's value of this thread-local
     */
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

    /**
     * Variant of set() to establish initialValue. Used instead
     * of set() in case user has overridden the set() method.
     *
     * @return the initial value
     */
    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }

    /**
     * Sets the current thread's copy of this thread-local variable
     * to the specified value.  Most subclasses will have no need to
     * override this method, relying solely on the {@link #initialValue}
     * method to set the values of thread-locals.
     *
     * @param value the value to be stored in the current thread's copy of
     *        this thread-local.
     */
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

    /**
     * Removes the current thread's value for this thread-local
     * variable.  If this thread-local variable is subsequently
     * {@linkplain #get read} by the current thread, its value will be
     * reinitialized by invoking its {@link #initialValue} method,
     * unless its value is {@linkplain #set set} by the current thread
     * in the interim.  This may result in multiple invocations of the
     * {@code initialValue} method in the current thread.
     *
     * @since 1.5
     */
     public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }

    /**
     * Get the map associated with a ThreadLocal. Overridden in
     * InheritableThreadLocal.
     *
     * @param  t the current thread
     * @return the map
     */
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

    /**
     * Create the map associated with a ThreadLocal. Overridden in
     * InheritableThreadLocal.
     *
     * @param t the current thread
     * @param firstValue value for the initial entry of the map
     */
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }

}
```

`ThreadLocal` 的核心方法有如下几个：
 - `T initialValue()：`返回该线程局部变量的初始值。该方法是一个 `protected` 的方法，显然是为了让子类重写而设计的。这个方法是一个延迟调用方法，在线程第一次调用 `get()` 方法时才执行，并且仅执行一次。`ThreadLocal` 中的缺省实现直接返回一个 `null`。
 - `ThreadLocal<S> withInitial(Supplier<? extends S> supplier)：`提供一个 `Supplier` 的 `lamda` 表达式用来当做初始值，该方法是 JDK 1.8 新增的方法。
 - `T setInitialValue()：`设置初始值。在调用 `get()` 方法没有对应的值时，调用此方法。该方法是一个 `private` 方法，防止被重写。
 - `T get()：`方法返回当前线程所对应的线程局部变量。先取出当前线程对应的 `threadLocalMap`，如果不存在则用创建一个，但是是用 `initialValue()` 方法的返回值作为 `value` 放入到 `map` 中，且返回 `initialValue()`，否则就直接从 `map` 取出 `this` 即 `ThreadLocal` 对应的 `value` 返回。
 - `void set(T value)：`设置当前线程对应的线程局部变量的值。先取出当前线程对应的 `threadLocalMap`，如果不存在则用创建一个，否则将 `value` 放入以 `this`(即 `ThreadLocal`)为 `key` 的映射的 `map` 中，其实 `ThreadLocalMap` 内部和 `HashMap` 机制一样，存储了 `Entry` 键值对数组。
 - `void remove()：`将当前线程局部变量的值删除，目的是为了减少内存的占用，该方法是 JDK 1.5 新增的方法。需要指出的是，当线程结束后，对应该线程的局部变量将自动被垃圾回收，所以显式调用该方法清除线程的局部变量并不是必须的操作，但它可以加快内存回收的速度。需要注意的是，如果 `remove()` 方法调用之后又调用了 `get()` 方法，会重新初始化一次，即再次调用 `initialValue()` 方法，除非在 `get()` 方法之前调用 `set()` 方法设置过值。

从上面的分析中，可以看到，`ThreadLocal` 的实现离不开 `ThreadLocalMap` 类，`ThreadLocalMap` 类是 `ThreadLocal` 的静态内部类。每个 `Thread` 维护一个 `ThreadLocalMap` 映射表，这个映射表的 `key` 是 `ThreadLocal` 实例本身，`value` 是真正需要存储的 `Object`。这样的设计主要有以下几点优势：
 - 这样设计之后每个 Map 的 Entry 数量变小了：之前是 Thread 的数量，现在是 ThreadLocal 的数量，能提高性能；
 - 当 Thread 销毁之后对应的 ThreadLocalMap 也就随之销毁了，能减少内存使用量。

`ThreadLocalMap` 看名字就知道是个 `map`，没错，这就是个 `HashMap` 机制实现的 `map`，用 `Entry` 数组来存储键值对，`key` 是 `ThreadLocal` 对象，`value` 则是具体的值。值得一提的是，为了方便 GC，`Entry` 继承了 `WeakReference`，也就是弱引用。里面有一些具体关于如何清理过期的数据、扩容等机制，思路基本和 `Hashmap` 差不多，有兴趣的可以自行阅读了解，一般只需知道大概的数据存储结构即可。`ThreadLocalMap` 的源码如下：
```java
static class ThreadLocalMap {

    /**
     * The entries in this hash map extend WeakReference, using
     * its main ref field as the key (which is always a
     * ThreadLocal object).  Note that null keys (i.e. entry.get()
     * == null) mean that the key is no longer referenced, so the
     * entry can be expunged from table.  Such entries are referred to
     * as "stale entries" in the code that follows.
     */
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /**
         * The value associated with this ThreadLocal.
         */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }

    /**
     * The initial capacity -- MUST be a power of two.
     */
    private static final int INITIAL_CAPACITY = 16;

    /**
     * The table, resized as necessary.
     * table.length MUST always be a power of two.
     */
    private Entry[] table;

    /**
     * The number of entries in the table.
     */
    private int size = 0;

    /**
     * The next size value at which to resize.
     */
    private int threshold; // Default to 0

    /**
     * Set the resize threshold to maintain at worst a 2/3 load factor.
     */
    private void setThreshold(int len) {
        threshold = len * 2 / 3;
    }

    /**
     * Construct a new map initially containing (firstKey, firstValue).
     * ThreadLocalMaps are constructed lazily, so we only create
     * one when we have at least one entry to put in it.
     */
    ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
        table = new Entry[INITIAL_CAPACITY];
        int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
        table[i] = new Entry(firstKey, firstValue);
        size = 1;
        setThreshold(INITIAL_CAPACITY);
    }

}
```

## 与同步机制的比较 ##
ThreadLocal 和线程同步机制相比有什么优势呢？
 - `Synchronized` 用于线程间的数据共享。在同步机制中，通过对象的锁机制保证同一时间只有一个线程访问变量。这时该变量是多个线程共享的，使用同步机制要求程序慎密地分析什么时候对变量进行读写，什么时候需要锁定某个对象，什么时候释放对象锁等繁杂的问题，程序设计和编写难度相对较大。
 - `ThreadLocal` 则用于线程间的数据隔离。ThreadLocal 则从另一个角度来解决多线程的并发访问。ThreadLocal 会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突。因为每一个线程都拥有自己的变量副本，从而也就没有必要对该变量进行同步了。ThreadLocal 提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的变量封装进 ThreadLocal。

概括起来说，对于多线程资源共享的问题，同步机制采用了“以时间换空间”的方式，而 ThreadLocal 采用了“以空间换时间”的方式。前者仅提供一份变量，让不同的线程排队访问，而后者为每一个线程都提供了一份变量，因此可以同时访问而互不影响。

## 总结 ##
`ThreadLocal` 是一个用于提供线程局部变量的工具类，它主要用于将私有线程和该线程存放的副本对象做一个映射，各个线程之间的变量互不干扰，在高并发场景下，可以实现无状态的调用，特别适用于各个线程依赖不通的变量值完成操作的场景。

`ThreadLocal` 是解决线程安全问题一个很好的思路，它通过为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突。在很多情况下，`ThreadLocal` 比直接使用 `synchronized` 同步机制解决线程安全问题更简单，更方便，且结果程序拥有更高的并发性。

