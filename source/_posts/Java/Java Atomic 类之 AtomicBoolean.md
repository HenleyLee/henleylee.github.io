---
title: Java Atomic 类之 AtomicBoolean
categories: Java
tags:
  - Java
abbrlink: 842a4d70
date: 2019-04-19 18:32:36
---

在 `java.util.concurrent.atomic` 包下，有 `AtomicBoolean`、`AtomicInteger`、`AtomicLong`、`AtomicReference` 等原子类，它们的基本特性就是在多线程环境下，当有多个线程同时执行这些类的实例包含的方法时，具有排他性，即当某个线程进入方法，执行其中的指令时，不会被其他线程打断，而别的线程就像自旋锁一样，一直等到该方法执行完成，才由 JVM 从等待队列中选择一个另一个线程进入，这只是一种逻辑上的理解。实际上是借助硬件的相关指令来实现的，不会阻塞线程(或者说只是在硬件级别上阻塞了)。

## 前言 ##
`java.util.concurrent.atomic.AtomicBoolean` 类提供了可以原子读取和写入的底层 `boolean` 的操作，并且还包含高级原子操作。`AtomicBoolean` 支持基础 `boolean` 变量上的原子操作。其底层是通过 `volatile` 和 `CAS` 实现的，其中 `volatile` 保证了内存可见性，`CAS` 算法保证了原子性。

### volatile ###
`volatile` 是一种稍弱的同步机制，用来确保将变量的更新操作通知到其他线程。当把变量声明为 `volatile` 类型后，编译器与运行时都会注意到这个变量是共享的，因此不会将该变量上的操作与其他内存操作一起重排序。`volatile` 变量不会被缓存在寄存器或者对其他处理器不可见的地方，因此在读取 `volatile` 类型的变量时总返回最新写入的值。在访问 `volatile` 变量时不会执行加锁操作，因此也就不会使执行线程阻塞，因此 `volatile` 变量是一种比 `sychronized` 关键字更轻量级的同步机制。

### CAS ###
`CAS(Compare And Swap)` 即比较并交换，`CAS` 是乐观锁技术，当多个线程尝试使用 `CAS` 同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。它包含三个参数：内存值 V、预期值 A、要修改的新值 B。当且仅当预期值 A 和内存值 V 相同时，将内存值 V 修改为 B，否则什么都不做。原理如下图所示：
![CAS乐观锁原理](https://henleylee.github.io/medias/java/atomic_cas_process.png)

## 常用方法 ##
`AtomicBoolean` 类中常用的重要方法如下：

| 方法                                                               | 功能描述                                                 |
|--------------------------------------------------------------------|----------------------------------------------------------|
| `public boolean get()`                                             | 返回当前值                                               |
| `public void set(boolean newValue)`                                | 无条件地设置为给定值                                     |
| `public void lazySet(boolean newValue)`                            | 最终设置为给定值                                         |
| `public boolean getAndSet(boolean newValue)`                       | 以原子方式设置为给定值并返回之前的值                     |
| `public boolean compareAndSet(boolean expect, boolean update)`     | 如果当前值`==`期望值，则以原子方式将值设置为给定的更新值 |
| `public boolean weakCompareAndSet(boolean expect, boolean update)` | 如果当前值`==`期望值，则以原子方式将值设置为给定的更新值 |
| `public String toString()`                                         | 返回当前值的 `String` 表示形式                           |

## 源码分析 ##
`AtomicBoolean` 类保证了一系列的操作都是原子操作，不会受到多线程环境下的并发不安全问题，原理则是依赖神奇的 `sun.misc.Unsafe` 支持。下面简单分析一下 `AtomicBoolean` 的源码：
```java
public class AtomicBoolean implements Serializable {

    private static final long serialVersionUID = 4654671469794556979L;

    private static final Unsafe unsafe = Unsafe.getUnsafe(); // 调用指针类Unsafe
    private static final long valueOffset;                   // 变量value的内存偏移量
    private volatile int value;                              // volatile修饰的变量value

    static {
        try {
            valueOffset = unsafe.objectFieldOffset(AtomicBoolean.class.getDeclaredField("value"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }

    /**
     * 构造方法(使用初始值false创建一个新的AtomicBoolean)
     */
    public AtomicBoolean() {
    }

    /**
     * 构造方法(使用给定的初始值创建新的AtomicBoolean)
     *
     * @param initialValue 初始值
     */
    public AtomicBoolean(boolean initialValue) {
        value = initialValue ? 1 : 0;
    }

    /**
     * 返回当前值
     *
     * @return 当前值
     */
    public final boolean get() {
        return value != 0;
    }

    /**
     * 无条件地设置为给定值
     *
     * @param newValue 新值
     */
    public final void set(boolean newValue) {
        value = newValue ? 1 : 0;
    }

    /**
     * 最终设置为给定值
     *
     * @param newValue 新值
     * @since 1.6
     */
    public final void lazySet(boolean newValue) {
        unsafe.putOrderedInt(this, valueOffset, (newValue ? 1 : 0));
    }

    /**
     * 以原子方式设置为给定值并返回之前的值
     *
     * @param newValue 新值
     * @return 之前的值
     */
    public final boolean getAndSet(boolean newValue) {
        boolean prev;
        do {
            prev = get();
        } while (!compareAndSet(prev, newValue));
        return prev;
    }

    /**
     * 如果当前值==期望值，则以原子方式将值设置为给定的更新值
     *
     * @param expect 期望值
     * @param update 新值
     * @return 如果成功，则返回true；实际值不等于预期值则返回 false
     */
    public final boolean compareAndSet(boolean expect, boolean update) {
        return unsafe.compareAndSwapInt(this, valueOffset,
                (expect ? 1 : 0),
                (update ? 1 : 0));
    }

    /**
     * 如果当前值==期望值，则以原子方式将值设置为给定的更新值
     *
     * <p>
     * 可能意外失败并且不提供排序保证，因此几乎只是compareAndSet()方法的适当替代方法。
     *
     * @param expect 期望值
     * @param update 新值
     * @return 如果成功，则返回true
     */
    public boolean weakCompareAndSet(boolean expect, boolean update) {
        return unsafe.compareAndSwapInt(this, valueOffset,
                (expect ? 1 : 0),
                (update ? 1 : 0));
    }

    /**
     * 返回当前值的String表示形式
     *
     * @return 当前值的String表示形式
     */
    public String toString() {
        return Boolean.toString(get());
    }

}
```

## 总结 ##
使用 `java.util.concurrent.atomic` 包下的原子类，最大的好处就是可以避免多线程的优先级倒置和死锁情况的发生，提升在高并发处理下的性能。

