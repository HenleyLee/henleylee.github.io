---
title: Java 线程池详解
categories: Java
tags:
  - Java
abbrlink: cf2a801
date: 2019-04-09 18:52:50
---

Java 中，使用线程来异步执行任务。Java 线程的创建与销毁需要一定的开销，如果为每一个任务创建一个新线程来执行，这些线程的创建和销毁将消耗大量的计算资源。针对这种情况，通常需要使用线程池来管理线程，使用线程池有以下几个好处：
 - 降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
 - 提高响应速度。当任务到达时，任务可以不需要等到线程创建就能立即执行。
 - 提高线程的可管理性。线程是稀缺资源，不能无限制创建，否则不但会消耗资源，还会降低系统的稳定性，而使用线程池可以进行统一分配、调优和监控。

## 线程池的创建 ##
Java 中创建线程池很简单，只需要通过线程池工厂类 `Executors` 的提供的静态方法即可。`Executors` 提供四种线程池，分别为：
 - **`newFixedThreadPool()：`**创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
 - **`newCachedThreadPool()：`**创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
 - **`newScheduledThreadPool()：`**创建一个定长线程池，支持定时和周期性任务执行。
 - **`newSingleThreadExecutor()：`**创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。
 - **`newSingleThreadScheduledExecutor()：`**创建一个单线程化的线程池，支持定时和周期性任务执行。
 - **`newWorkStealingPool()：`**JDK 1.8 新增，创建一个维护足够线程的线程池来支持给定的并行级别，并通过使用多个队列，减少竞争，它需要穿一个并行级别的参数，如果不传，则被设定为默认的 CPU 数量。

根据 `Executors` 提供的方法可知：线程池分为 `ThreadPoolExecutor`、`ScheduledThreadPoolExecutor` 和 `ForkJoinPool` 3 种。

## ThreadPoolExecutor ##
`java.util.concurrent.ThreadPoolExecutor` 类是线程池中最核心的一个类，因此如果要透彻地了解 Java 中的线程池，必须先了解这个类。

### 构造方法 ###
`ThreadPoolExecutor` 类提供了四个构造方法：
```java
public class ThreadPoolExecutor extends AbstractExecutorService {

    public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime,
                              TimeUnit unit, BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }

    public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime,
                              TimeUnit unit, BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             threadFactory, defaultHandler);
    }

    public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime,
                              TimeUnit unit, BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), handler);
    }

    public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime,
                              TimeUnit unit, BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory, RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }

}
```

从上面的代码可以得知，`ThreadPoolExecutor` 继承了 `AbstractExecutorService` 类，并提供了四个构造器，事实上，通过观察每个构造器的源码具体实现，发现前面三个构造器都是调用的第四个构造器进行的初始化工作。

下面解释下一下构造器中各个参数的含义：
 - **`corePoolSize：`**核心线程的数量。默认情况下，在创建了线程池后，线程池中的线程数为0。当提交一个任务到线程池时，线程池会创建一个核心线程来执行任务，线程池中的线程数目达到线程池核心线程的数量时就不再创建，就会把后续到达的任务放到缓存队列当中。
 - **`maximumPoolSize：`**线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。所以只有队列满了的时候，这个参数才有意义。因此当你使用了无界任务队列的时候，这个参数就没有效果了。
 - **`keepAliveTime：`**线程活动保持时间，即当线程池的工作线程空闲后，保持存活的时间。默认情况下，只有当线程池中的线程数大于核心线程数量时，keepAliveTime 才会起作用，直到线程池中的线程数不大于核心线程数量，即当线程池中的线程数大于核心线程数量时，如果一个线程空闲的时间达到 keepAliveTime，则会终止，直到线程池中的线程数不超过核心线程数量。但是如果调用了 allowCoreThreadTimeOut(boolean) 方法，在线程池中的线程数不大于核心线程数量时，keepAliveTime 参数也会起作用，直到线程池中的线程数为 0。
 - **`unit：`**线程活动保持时间的单位：可选的单位有7种，分别为：天(DAYS)、小时(HOURS)、分钟(MINUTES)、毫秒(MILLISECONDS)、微秒(MICROSECONDS，千分之一毫秒)和纳秒(NANOSECONDS，千分之一微秒)
 - **`workQueue：`**用来保存等待执行任务的阻塞队列。workQueue 的类型为 BlockingQueue<Runnable>，通常可以取下面三种类型：
  - `ArrayBlockingQueue：`基于数组的先进先出队列，此队列创建时必须指定大小。
  - `LinkedBlockingQueue：`基于链表的先进先出队列，如果创建时没有指定此队列大小，则默认为 Integer.MAX_VALUE。
  - `SynchronousQueue：`这个队列比较特殊，它不会保存提交的任务，而是将直接新建一个线程来执行新来的任务。
 - **`threadFactory：`**线程工厂，主要用来创建线程。可以通过线程工厂给每个创建出来的线程设置更加有意义的名字。
 - **`handler：`**拒绝处理任务时策略，可以理解为饱和策略。当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。默认提供了以下四种策略：
  - `AbortPolicy：`丢弃任务并抛出 RejectedExecutionException 异常。
  - `DiscardPolicy：`丢弃任务，但是不抛出异常。
  - `DiscardOldestPolicy：`丢弃队列最前面的任务，并执行当前任务。
  - `CallerRunsPolicy：`由调用线程处理该任务。

### 线程池状态 ###
在 `ThreadPoolExecutor` 中定义几个常量用来表示线程池的各个状态：
```java
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;
```

> 当线程池处于 `SHUTDOWN` 或 `STOP` 状态，并且所有工作线程已经销毁，任务缓存队列已经清空或执行结束后，线程池被设置为 `TERMINATED` 状态。

### execute() ###
`execute()` 方法用于向线程池提交任务，该方法是 `java.util.concurrent.Executor` 接口中声明的方法，在 `ThreadPoolExecutor` 进行了具体的实现，这个方法是 `ThreadPoolExecutor` 的核心方法，通过这个方法可以向线程池提交一个任务，交由线程池去执行。
```java
public void execute(Runnable command) {
    // 判断需要提交的任务是否为null
    if (command == null)
        // 如果需要提交的任务为null，则抛出空指针异常
        throw new NullPointerException();
    int c = ctl.get();
    // 判断线程池的当前线程数量是否小于核心线程数量
    if (workerCountOf(c) < corePoolSize) {
        // 直接将任务加入worker启动运行
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    // 运行线程数量大于核心线程数量时，当前状态为RUNNING并且将任务加入缓冲队列成功
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        // 如果线程池没有处于RUNNING状态，则从缓冲队列中删除任务，执行reject()方法处理任务
        if (!isRunning(recheck) && remove(command))
            reject(command);
        // 如果线程池处于RUNNING状态，但是没有线程，则将任务加入worker
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 往线程池中创建新的线程失败，则执行reject()方法处理任务
    else if (!addWorker(command, false))
        reject(command);
}
```

### submit() ###
`submit()` 方法用于向线程池提交任务，该方法是 `java.util.concurrent.ExecutorService` 接口中声明的方法，在 `AbstractExecutorService` 就已经有了具体的实现，在 `ThreadPoolExecutor` 中并没有对其进行重写，这个方法也是用来向线程池提交任务的，但是它和 `execute()` 方法不同，它能够返回任务执行的结果，去看 `submit()` 方法的实现，会发现它实际上还是调用的 `execute()` 方法，只不过它利用了 `java.util.concurrent.Future` 来获取任务执行结果。
```java
public <T> Future<T> submit(Runnable task, T result) {
    if (task == null) throw new NullPointerException();
    // 将提交的任务会封装成一个FutureTask对象
    RunnableFuture<T> ftask = newTaskFor(task, result);
    // 执行execute()方法提交任务
    execute(ftask);
    // 返回FutureTask对象
    return ftask;
}
```

> 不管 `submit(callable)` 还是 `submit(runnable)` 都是提交一个任务是指是创建了一个 `FutureTask`，被 `FutureTask` 构造函数统一适配为自己的成员 `callable`，最终都会执行 `execute(futureTask)` 方法。

### shutdown() ###
`shutdown()` 方法将线程池里的线程状态设置成 `SHUTDOWN` 状态，此时线程池不能够接受新的任务，但是会等待所有任务执行完毕后才终止。

### shutdownNow() ###
`shutdownNow()` 方法将线程池里的线程状态设置成 `STOP` 状态，此时线程池不能接受新的任务，并且会去尝试终止正在执行的任务，同时清空任务缓存队列，返回尚未执行的任务。

## ScheduledThreadPoolExecutor ##
`java.util.concurrent.ScheduledThreadPoolExecutor` 类也是线程池中最核心的一个类，它继承 `ThreadPoolExecutor` 来重用线程池的功能，并且还支持周期性任务的调度。

### 构造方法 ###
`ScheduledThreadPoolExecutor` 类提供了四个构造方法：
```java
public class ScheduledThreadPoolExecutor extends ThreadPoolExecutor implements ScheduledExecutorService {

    public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE, DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
              new DelayedWorkQueue());
    }

    public ScheduledThreadPoolExecutor(int corePoolSize, ThreadFactory threadFactory) {
        super(corePoolSize, Integer.MAX_VALUE, DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
              new DelayedWorkQueue(), threadFactory);
    }

    public ScheduledThreadPoolExecutor(int corePoolSize, RejectedExecutionHandler handler) {
        super(corePoolSize, Integer.MAX_VALUE, DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
              new DelayedWorkQueue(), handler);
    }

    public ScheduledThreadPoolExecutor(int corePoolSize, ThreadFactory threadFactory,
                                       RejectedExecutionHandler handler) {
        super(corePoolSize, Integer.MAX_VALUE, DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
              new DelayedWorkQueue(), threadFactory, handler);
    }

}
```

> 因为 `ScheduledThreadPoolExecutor` 继承自 `ThreadPoolExecutor`，所以这里都是调用的 `ThreadPoolExecutor` 类的构造方法。

### 常用方法 ###
```java
// 创建一个Runnable定时任务，并在delay时间后执行
public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit);
// 创建一个Callable定时任务，并在delay时间后执行
public <V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit);
// 创建一个Runnable周期任务，在initialDelay时间开始执行，每隔period时间再执行一次
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDela,y long period, TimeUnit unit);
// 创建一个Runnable周期任务，在initialDelay时间开始执行，以一次任务结束的时间为起点，每隔delay时间再执行一次
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit);
```

## ForkJoinPool ##
`java.util.concurrent.ForkJoinPool` 类和 `ThreadPoolExecutor` 一样它也是 `ExecutorService` 接口的实现类，是 Java 7 提供的一个用于并行执行任务的特殊线程池，将一个大任务拆分成多个小任务后，使用 fork 可以将小任务分发给其他线程同时处理，使用 join 可以将多个线程处理的结果进行汇总；这实际上就是分治思想的并行版本。

### 构造方法 ###
`ForkJoinPool` 类提供了四个构造方法：
public class ForkJoinPool extends AbstractExecutorService {

    public ForkJoinPool() {
        this(Math.min(MAX_CAP, Runtime.getRuntime().availableProcessors()),
             defaultForkJoinWorkerThreadFactory, null, false);
    }

    public ForkJoinPool(int parallelism) {
        this(parallelism, defaultForkJoinWorkerThreadFactory, null, false);
    }

    public ForkJoinPool(int parallelism, ForkJoinWorkerThreadFactory factory,
                        UncaughtExceptionHandler handler, boolean asyncMode) {
        this(checkParallelism(parallelism), checkFactory(factory), handler,
             asyncMode ? FIFO_QUEUE : LIFO_QUEUE, "ForkJoinPool-" + nextPoolId() + "-worker-");
        checkPermission();
    }

    private ForkJoinPool(int parallelism, ForkJoinWorkerThreadFactory factory,
                         UncaughtExceptionHandler handler, int mode,
                         String workerNamePrefix) {
        this.workerNamePrefix = workerNamePrefix;
        this.factory = factory;
        this.ueh = handler;
        this.config = (parallelism & SMASK) | mode;
        long np = (long)(-parallelism); // offset ctl counts
        this.ctl = ((np << AC_SHIFT) & AC_MASK) | ((np << TC_SHIFT) & TC_MASK);
    }

}

从上面的代码可以得知，`ForkJoinPool` 继承了 `AbstractExecutorService` 类，并提供了四个构造器，事实上，通过观察每个构造器的源码具体实现，发现前面三个构造器都是调用的第四个构造器进行的初始化工作。

下面解释下一下构造器中各个参数的含义：
 - **`parallelism：`**并行级别。通常默认为 JVM 可用的处理器个数 Runtime.getRuntime().availableProcessors()。
 - **`factory：`**线程工程，主要用来创建 ForkJoinPool 中使用的线程。
 - **`handler：`**用于处理工作线程未处理的异常，默认为 null。
 - **`mode：`**用于控制 WorkQueue 的工作模式，有 FIFO_QUEUE 和 LIFO_QUEUE 两个值。
 - **`workerNamePrefix：`**创建的线程的名称前缀。

### execute() ###
`execute()` 方法用于提交任务到 ForkJoinPool，最终是调用的 `externalSubmit()` 方法。
```java
public void execute(ForkJoinTask<?> task) {
    externalSubmit(task);
}

public void execute(Runnable task) {
    if (task == null)
        throw new NullPointerException();
    ForkJoinTask<?> job;
    if (task instanceof ForkJoinTask<?>) // avoid re-wrap
        // 避免二次包装
        job = (ForkJoinTask<?>) task;
    else
        // 将要提交的任务包装成ForkJoinTask
        job = new ForkJoinTask.RunnableExecuteAction(task);
    externalSubmit(job);
}
```

### submit() ###
`submit()` 方法也是用于提交任务到 ForkJoinPool 的，但是它和 `execute()` 方法不同，它能够返回任务执行的结果，去看 `submit()` 方法的实现，会发现它实际上也是调用的 `externalSubmit()` 方法，只不过它利用了 `java.util.concurrent.ForkJoinTask` 来获取任务执行结果。
```java
public <T> ForkJoinTask<T> submit(ForkJoinTask<T> task) {
    return externalSubmit(task);
}

public <T> ForkJoinTask<T> submit(Callable<T> task) {
    return externalSubmit(new ForkJoinTask.AdaptedCallable<T>(task));
}

public <T> ForkJoinTask<T> submit(Runnable task, T result) {
    return externalSubmit(new ForkJoinTask.AdaptedRunnable<T>(task, result));
}

public ForkJoinTask<?> submit(Runnable task) {
    if (task == null)
        throw new NullPointerException();
    ForkJoinTask<?> job;
    if (task instanceof ForkJoinTask<?>) // avoid re-wrap
        // 避免二次包装
        job = (ForkJoinTask<?>) task;
    else
        // 将要提交的任务包装成ForkJoinTask
        job = new ForkJoinTask.AdaptedRunnableAction(task);
    return externalSubmit(job);
}
```

### invoke() ###
`invoke()` 方法也是用于提交任务到 ForkJoinPool 的，只不过该方法是同步提交的，等待完成后返回结果。
```java
public <T> T invoke(ForkJoinTask<T> task) {
    if (task == null)
        throw new NullPointerException();
    externalSubmit(task);
    return task.join();
}
```

### ForkJoinTask ###
#### 核心方法 ####
大多数情况下，都是直接提交 `ForkJoinTask` 对象到 `ForkJoinPool` 中，因为 `ForkJoinTask` 有以下三个核心方法：
 - **`fork()：`**在任务执行过程中将大任务划分为多个小的子任务，调用子任务的 `fork()` 方法可以将任务放到线程池中异步调度。
 - **`join()：`**调用子任务的 `join()` 方法等待任务返回的结果。这个方法类似于 `Thread.join()`，区别在于前者不受线程中断机制的影响。如果子任务中有运行时异常，`join()` 方法会抛出异常，`quietlyJoin()` 方法不会抛出异常也不会返回结果，需要开发者调用 `getException()` 或 `getRawResult()` 自己去处理异常和结果。
 - **`invoke()：`**在当前线程同步执行该任务。该方法也不受中断机制影响。如果子任务中有运行时异常，`invoke()` 方法会抛出异常，`quietlyInvoke()` 方法不会抛出异常也不会返回结果，需要开发者调用 `getException()` 或 `getRawResult()` 自己去处理异常和结果。

> `ForkJoinTask` 中 `join()` 和 `invoke()` 方法都不受中断机制影响，内部调用 `externalAwaitDone()` 方法实现。
> 如果是在 `ForkJoinTask` 内部调用 `get()` 方法，本质上和 `join()` 方法一样都是调用 `externalAwaitDone()`。
> 但如果是在 `ForkJoinTask` 外部调用 `get()` 方法，这时会受线程中断机制影响，因为内部是通过调用 `externalInterruptibleAwaitDone()` 方法实现的。

#### 静态方法 ####
`ForkJoinTask` 由上面三个方法衍生出了几个静态方法：
```java
public static void invokeAll(ForkJoinTask<?> t1, ForkJoinTask<?> t2)
public static void invokeAll(ForkJoinTask<?>... tasks)
public static <T extends ForkJoinTask<?>> Collection<T> invokeAll(Collection<T> tasks)
```

> 上面几个方法都是让第一个任务同步执行，其他任务异步执行(注意：其他任务先 fork，第一个任务再 invoke)。

#### 任务状态 ####
`ForkJoinTask` 内部维护了四个状态：
```java
/** The run status of this task */
// 默认等于0，小于0表示任务已经执行过，大于0说明任务没执行完
volatile int status; // accessed directly by pool and workers
static final int DONE_MASK   = 0xf0000000;  // mask out non-completion bits
// NORMAL,CANCELLED,EXCEPTIONAL均小于0
static final int NORMAL      = 0xf0000000;  // must be negative
static final int CANCELLED   = 0xc0000000;  // must be < NORMAL
static final int EXCEPTIONAL = 0x80000000;  // must be < CANCELLED
static final int SIGNAL      = 0x00010000;  // must be >= 1 << 16

static final int SMASK       = 0x0000ffff;  // short bits for tags
```

#### 抽象子类 ####
通常不会直接使用 `ForkJoinTask`，而是使用它的两个抽象子类：
 - **`RecursiveAction：`**没有返回值的任务；
 - **`RecursiveTask：`**有返回值的任务。

