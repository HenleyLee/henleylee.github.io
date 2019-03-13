---
title: Android 中的 Handler 消息机制详解
categories: Android
tags:
  - Android
abbrlink: 9669634f
date: 2019-03-05 18:52:35
---

Android 应用程序是通过消息来驱动的，Android 某种意义上也可以说成是一个以消息驱动的系统，UI、事件、生命周期都和消息处理机制息息相关，并且消息处理机制在整个 Android 知识体系中也是尤其重要。

## 消息处理机制 ##
**`消息处理机制`**本质是一个线程开启循环模式持续监听并依次处理其他线程给它发的消息。

简单的说：一个线程开启一个无限循环模式，不断遍历自己的消息列表，如果有消息就挨个拿出来做处理，如果列表没消息，自己就堵塞(相当于 wait，让出 CPU 资源给其他线程)，其他线程如果想让该线程做什么事，就往该线程的消息队列插入消息，该线程会不断从队列里拿出消息做处理。

## Handler 简介 ##
**`android.os.Handler`**是 Android 中相当经典的异步消息机制，在 Android 发展的历史长河中扮演着很重要的角色，无论是我们直接面对的应用层还是 FrameWork 层，使用的场景还是相当的多。

### Handler 的引入 ###
在 Android 开发中，UI 操作是线程不安全的，为了保证 UI 操作是线程安全的，规定了只允许在 UI 线程更新 UI 组件，如果在子线程中尝试进行 UI 操作，程序就有可能会崩溃。

在多线程的应用场景中，将工作线程中需更新 UI 的操作信息传递到 UI 线程，从而实现多个线程并发更新 UI 的同时并保证线程安全，最终实现异步消息的处理，因此引入了 **`Handler 消息传递机制`**。

### Handler 的作用 ###
`Handler` 能够发送和处理 `Message` 和 `Runnable`，每个 `Handler` 对象对应一个 `Thread` 和 `Thread 的消息队列`。每当创建一个 `Handler`时，它就和所在线程的消息队列绑定在一起，然后就可以传递 `Message` 和 `Runnable` 到消息队列中，执行消息后就从消息队列中退出。

`Handler` 有以下两个主要用途：
 - 执行定时任务：将未来某个时间点将要执行的 `Message` 或 `Runnable` 加入到消息队列；
 - 线程间的通信：在子线程把需要在另一个线程执行的操作加入到消息队列中去。

## Handler 消息机制 ##
**`android.os.Handler`**是 Android 类库提供的用于接受、传递和处理 `Message`或 `Runnable`对象的处理类，它结合 `Looper`、`Message` 和 `MessageQueue` 以及当前线程实现了一个消息循环机制，用于实现任务的异步加载和处理。

`Handler` 消息处理机制的流程图如下图所示：
![Handler消息处理机制流程图](https://lyl873825813.github.io/medias/android/handler_process.png)

> `Handler` 消息机制是 Android 的两大消息机制之一，另一个是 `Binder IPC` 机制。

### 消息机制架构 ###
`Handler` 消息机制的架构图如下图所示：
![Handler消息机制架构图](https://lyl873825813.github.io/medias/android/handler_architecture.png)

从 `Handler` 消息机制的架构图中可以看到：
 - `Looper` 有一个 `MessageQueue` 消息队列；
 - `MessageQueue` 有一组待处理的 `Message`；
 - `Message` 中有一个用于处理消息的 `Handler`；
 - `Handler` 中有 `Looper` 和 `MessageQueue`。

### 消息机制核心类 ###
`Handler` 消息机制中有3个核心类：
 - **`Handler：`**处理程序
 - **`MessageQueue：`**消息队列
 - **`Looper：`**循环器

它们之间的关系如下图所示：
![Handler消息机制核心类关系](https://lyl873825813.github.io/medias/android/handler_core_classes.png)

### 消息机制典型实例 ###
下面展示一个典型的关于 `Handler/Looper` 的线程：

```java
class LooperThread extends Thread {
    
    public Handler mHandler;

    public void run() {
        Looper.prepare();

        mHandler = new Handler() {
            public void handleMessage(Message msg) {
                // process incoming messages here
            }
        };

        Looper.loop();
    }
}
```

## Looper ##
**`android.os.Looper`**用于为一个线程开启一个消息循环，按分发机制将消息分发给目标处理者。

> 默认情况下，线程没有与之关联的消息循环；当一个线程运行到某处，准备创建一个消息循环，必须在将要运行循环的线程中先调用 `Looper.prepare()` 做一些准备工作(即：创建一个 Looper 对象，并把它设置进线程的本地存储区里)，然后继续调用 `Looper.loop()` 建立起消息处理循环，开始让它处理消息，直到循环停止。

### prepare() ###
**`prepare()`**方法用于线程创建消息循环前做一些准备工作，创建一个 `Looper` 对象，并把它设置进线程的本地存储区里。对于无参的情况，默认调用 `prepare(true)`，表示的是当前 `Looper` 允许退出，而对于 `false` 的情况则表示当前 `Looper` 不允许退出。`prepare()` 方法的源码如下：
```java
public static void prepare() {
    prepare(true);
}

private static void prepare(boolean quitAllowed) {
    // 每个线程只允许执行一次该方法，第二次执行时线程的TLS已有数据，则会抛出异常
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    // 创建Looper对象，并保存到当前线程的TLS区域
    sThreadLocal.set(new Looper(quitAllowed));
}
```
这里的 `sThreadLocal` 是 `ThreadLocal` 类型，`ThreadLocal` 表示`线程本地存储区`(Thread Local Storage，简称为 TLS)，每个线程都有自己的私有的本地存储区域，不同线程之间彼此不能访问对方的 TLS 区域。

`Looper.prepare()` 在每个线程只允许执行一次，该方法会创建 `Looper` 对象，`Looper` 的构造方法中会创建一个 `MessageQueue` 对象，再将 `Looper` 对象保存到当前线程本地存储区。`Looper` 的构造方法如下：
```java
private Looper(boolean quitAllowed) {
    // 创建MessageQueue对象
    mQueue = new MessageQueue(quitAllowed);
    // 记录当前线程
    mThread = Thread.currentThread();
}
```

另外，与 `prepare()` 方法功能相近的，还有一个 `prepareMainLooper()` 方法，该方法主要在 `android.app.ActivityThread` 类中使用，它是为主线程创建一个 `Looper`，在主线程创建 `Looper` 对象中，就设置了不允许退出消息循环。`prepareMainLooper()` 方法的源码如下：
```java
public static void prepareMainLooper() {
    // 设置不允许退出的Looper
    prepare(false);
    synchronized (Looper.class) {
        // 将当前的Looper保存为主Looper，每个线程只允许执行一次
        if (sMainLooper != null) {
            throw new IllegalStateException("The main Looper has already been prepared.");
        }
        sMainLooper = myLooper();
    }
}
```

### myLooper() ###
**`myLooper()`**方法用于获取 `TLS` 存储的 `Looper` 对象。如果调用线程未与 `Looper` 关联(没有调用过 `prepare()` 方法)则返回 `null`。`myLooper()` 方法的源码如下：
```java
public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}
```

另外，与 `myLooper()` 方法功能相近的，还有一个 `getMainLooper()` 方法，该方法用于获取主线程的 `Looper` 对象。`getMainLooper()` 方法的源码如下：
```java
public static Looper getMainLooper() {
    synchronized (Looper.class) {
        return sMainLooper;
    }
}
```

### loop() ###
**`loop()`**方法用于开启消息循环，让 `Looper` 开始工作，从消息队列里取出消息并处理。`loop()` 方法的源码如下：
```java
public static void loop() {
    // 获取TLS存储的Looper对象
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    // 获取Looper对象中的消息队列
    final MessageQueue queue = me.mQueue;

    // Make sure the identity of this thread is that of the local process,
    // and keep track of what that identity token actually is.
    Binder.clearCallingIdentity();
    // 确保在权限检查时基于本地进程，而不是调用进程
    final long ident = Binder.clearCallingIdentity();

    // 进入loop的主循环方法
    for (; ; ) {
        // 取出消息队列中的下一条消息(可能会阻塞)
        Message msg = queue.next(); // might block
        // 没有消息，则退出循环
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }

        // 默认为null，可通过setMessageLogging()方法来指定输出，用于debug功能
        final Printer logging = me.mLogging;
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }


        // 用于分发Message
        msg.target.dispatchMessage(msg);
        dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;

        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }

        // 恢复调用者信息
        final long newIdent = Binder.clearCallingIdentity();
        msg.recycleUnchecked();
    }
}
```

`loop()` 方法进入循环模式，不断重复下面的操作，直到没有消息时退出循环：
 - 读取 `MessageQueue` 的下一条 `Message`；
 - 把 `Message` 分发给相应的 `target`；
 - 再把分发后的 `Message` 回收到消息池，以便重复利用。

这是这个消息处理的核心部分。另外，上面代码中可以看到有 `logging` 方法，这是用于 `debug` 的，默认情况下 `logging == null`，通过设置 `setMessageLogging()` 用来开启 `debug` 工作。

> 注意：写在 `Looper.loop()` 之后的代码不会被执行，这个函数内部是一个循环，当调用 `mHandler.getLooper().quit()` 后，loop 才会中止，其后的代码才能得以运行。

### quit() ###
**`quit()`**方法用于退出消息循环，让 `Looper` 停止工作。`quit()` 方法的源码如下：
```java
public void quit() {
    // 移除消息队列中的消息
    mQueue.quit(false);
}

public void quitSafely() {
    // 安全地移除消息队列中的消息
    mQueue.quit(true);
}
```

`quit()` 方法的实现最终调用的是 `MessageQueue.quit()` 方法，传入 `true` 表示只移除尚未触发的所有消息，对于正在触发的消息并不移除；传入 `flase` 表示移除所有的消息。

### 总结 ###
`Looper` 是 `MessageQueue` 和 `Handler` 的通信媒介，一个线程中只能有一个 `Looper`，但是一个 `Looper` 可以绑定多个线程的 `Handler`，即多个线程可以向一个 `Looper` 所持有的 `MessageQueue` 中发送消息，提供了线程间通信的方式。

`Looper` 循环取出消息队列中的消息，并将取出的消息分发给创建该消息的处理者(Handler)，在消息循环过程中，若消息队列为空，则线程阻塞。

## Message ##
**`android.os.Message`**用于表示消息体，装载需要发送的信息。`Message` 主要包含以下内容：

| 数据类型 | 成员变量 | 解释         |
|----------|----------|--------------|
| int      | what     | 消息类别     |
| long     | when     | 消息触发时间 |
| int      | arg1     | 参数1        |
| int      | arg2     | 参数2        |
| Object   | obj      | 消息内容     |
| Handler  | target   | 消息响应方   |
| Runnable | callback | 回调方法     |

创建消息的过程，就是填充消息的上述内容的一项或多项。

### 消息池 ###
在代码中，可能经常看到 `recycle()` 方法，看似可能是在做虚拟机的 `gc()` 相关的工作，其实不然，这是用于把消息加入到消息池的作用。这样的好处是，当消息池不为空时，可以直接从消息池中获取 `Message` 对象，而不是直接创建，提高效率。

`Message` 类中的静态变量 `sPool` 的数据类型为 `Message`，通过 `next` 成员变量，维护一个消息池；静态变量 `MAX_POOL_SIZE` 代表消息池的可用大小；消息池的默认大小为 `50`。

消息池常用的操作方法是 `obtain()` 和 `recycle()`。

### obtain() ###
**`obtain()`**方法用于从消息池中取出 `Message`，该方法可以避免在许多情况下分配新对象，比创建和分配新实例效率更高。`obtain()` 方法的源码如下：
```java
public static Message obtain() {
    synchronized (sPoolSync) {
        // 判断消息池是否为空
        if (sPool != null) {
            // 从sPool中取出一个Message对象
            Message m = sPool;
            // 将sPool指向next
            sPool = m.next;
            // 将消息链表断开
            m.next = null;
            // 清除in-use flag
            m.flags = 0;
            // 消息池的可用大小进行减1操作
            sPoolSize--;
            return m;
        }
    }
    // 当消息池为空时，直接创建Message对象
    return new Message();
}
```

`obtain()` 方法有以下几种重载方法：
```java
public static Message obtain()
public static Message obtain(Message)
public static Message obtain(Handler)
public static Message obtain(Handler, Runnable)
public static Message obtain(Handler, int)
public static Message obtain(Handler, int, Object)
public static Message obtain(Handler, int, int, int)
public static Message obtain(Handler, int, int, int, Object)
```

`obtain()` 方法都是把消息池表头的 `Message` 取走，再把表头指向 `next`。

### recycle() ###
**`recycle()`**方法用于把不再使用的 `Message` 加入消息池。`recycle()` 方法的源码如下：
```java
public void recycle() {
    // 判断消息是否正在使用
    if (isInUse()) {
        // Android 5.0 之前的版本默认为false，之后的版本默认为true
        if (gCheckRecycle) {
            throw new IllegalStateException("This message cannot be recycled because it "
                    + "is still in use.");
        }
        return;
    }
    // 对于不再使用的消息，加入到消息池
    recycleUnchecked();
}

void recycleUnchecked() {
    // 将消息标示位置为IN_USE，并清空消息所有的参数
    flags = FLAG_IN_USE;
    what = 0;
    arg1 = 0;
    arg2 = 0;
    obj = null;
    replyTo = null;
    sendingUid = -1;
    when = 0;
    target = null;
    callback = null;
    data = null;

    synchronized (sPoolSync) {
        // 当消息池没有满时，将Message对象加入消息
        if (sPoolSize < MAX_POOL_SIZE) {
            // 将消息加到链表的表头
            next = sPool;
            sPool = this;
            // 消息池的可用大小进行加1操作
            sPoolSize++;
        }
    }
}
```

`recycle()` 方法将 `Message` 加入到消息池的过程，都是把 `Message` 加到链表的表头。

### 总结 ###
`Message` 是线程间通信的数据元，即 `Handler` 接受和处理的消息对象，被用来存储需操作的通信信息。

## MessageQueue ##
**`android.os.MessageQueue`**用于表示消息队列，它由对应的 `Looper` 创建并进行管理，每个线程中都只会有一个 `MessageQueue` 对象。

### 创建消息队列 ###
`MessageQueue` 会在 `Looper.prepare()` 方法被调用创建 `Looper` 对象时，在 `Looper` 的构造方法中被创建。`MessageQueue` 的构造方法如下：
```java
MessageQueue(boolean quitAllowed) {
    mQuitAllowed = quitAllowed;
    // 通过native方法初始化消息队列，其中mPtr是供native代码使用
    mPtr = nativeInit();
}
```

### next() ###
**`next()`**方法用于提取消息队列中的下一条消息，该方法在 `Looper.loop()` 方法的循环体中被调用。`next()` 方法的源码如下：
```java
Message next() {
    final long ptr = mPtr;
    // 当消息循环已经退出并被丢弃，则直接返回
    if (ptr == 0) {
        return null;
    }

    // 循环迭代的首次为-1
    int pendingIdleHandlerCount = -1;
    int nextPollTimeoutMillis = 0;
    for (; ; ) {
        if (nextPollTimeoutMillis != 0) {
            Binder.flushPendingCommands();
        }

        // 阻塞操作，当等待nextPollTimeoutMillis时长，或者消息队列被唤醒，都会返回
        nativePollOnce(ptr, nextPollTimeoutMillis);

        synchronized (this) {
            // 尝试检索下一条消息，如果找到则返回
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            if (msg != null && msg.target == null) {
                // 当消息Handler为空时，查询MessageQueue中的下一条异步消息msg，则退出循环
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            if (msg != null) {
                if (now < msg.when) {
                    // 当异步消息触发时间大于当前时间，则设置下一次轮询的超时时长
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                } else {
                    // 获取一条消息，并返回
                    mBlocked = false;
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    // 设置消息的使用状态，即flags |= FLAG_IN_USE
                    msg.markInUse();
                    return msg;
                }
            } else {
                // 没有消息
                nextPollTimeoutMillis = -1;
            }

            // 已处理所有待处理消息，处理退出消息，返回null
            if (mQuitting) {
                dispose();
                return null;
            }

            // 当消息队列为空，或者是消息队列的第一个消息时
            if (pendingIdleHandlerCount < 0
                    && (mMessages == null || now < mMessages.when)) {
                pendingIdleHandlerCount = mIdleHandlers.size();
            }
            if (pendingIdleHandlerCount <= 0) {
                // 没有idle handlers 需要运行，则循环并等待
                mBlocked = true;
                continue;
            }

            if (mPendingIdleHandlers == null) {
                mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
            }
            mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
        }

        //只有第一次循环时，会运行idle handlers，执行完成后，重置pendingIdleHandlerCount为0
        for (int i = 0; i < pendingIdleHandlerCount; i++) {
            final IdleHandler idler = mPendingIdleHandlers[i];
            // 释放handler的引用
            mPendingIdleHandlers[i] = null;

            boolean keep = false;
            try {
                // idler时执行的方法
                keep = idler.queueIdle();
            } catch (Throwable t) {
                Log.wtf(TAG, "IdleHandler threw exception", t);
            }

            if (!keep) {
                synchronized (this) {
                    mIdleHandlers.remove(idler);
                }
            }
        }

        // 重置idle handler个数为0，以保证不会再次重复运行
        pendingIdleHandlerCount = 0;

        // 当调用一个空闲handler时，一个新message能够被分发，因此无需等待可以直接查询pending message
        nextPollTimeoutMillis = 0;
    }
}
```

`nativePollOnce` 是阻塞操作，其中 `nextPollTimeoutMillis` 代表下一个消息到来前，还需要等待的时长；当 `nextPollTimeoutMillis = -1` 时，表示消息队列中无消息，会一直等待下去。

当处于空闲时，往往会执行 `IdleHandler` 中的方法。当 `nativePollOnce()` 返回后，`next()` 从 `mMessages` 中提取一个消息。`nativePollOnce()` 方法用于提取消息队列中的消息，提取消息的调用链。

### enqueueMessage() ###
**`enqueueMessage()`**方法用于向消息队列中添加一条消息，该方法在 `Handler.enqueueMessage()` 方法中被调用。`enqueueMessage()` 方法的源码如下：
```java
boolean enqueueMessage(Message msg, long when) {
    // 判断Message的target是否为null(每一个普通的Message必须有一个target)
    if (msg.target == null) {
        throw new IllegalArgumentException("Message must have a target.");
    }
    // 判断Message是否正在使用
    if (msg.isInUse()) {
        throw new IllegalStateException(msg + " This message is already in use.");
    }

    synchronized (this) {
        // 判断是否正在退出时
        if (mQuitting) {
            IllegalStateException e = new IllegalStateException(
                    msg.target + " sending message to a Handler on a dead thread");
            Log.w(TAG, e.getMessage(), e);
            // 回收Message，加入到消息池
            msg.recycle();
            return false;
        }

        // 设置消息的使用状态，即flags |= FLAG_IN_USE
        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        if (p == null || when == 0 || when < p.when) {
            // p为null(代表MessageQueue没有消息)或者msg的触发时间是队列中最早的，则进入该该分支
            msg.next = p;
            mMessages = msg;
            // 当阻塞时需要唤醒
            needWake = mBlocked;
        } else {
            // 将消息按时间顺序插入到MessageQueue。一般地，不需要唤醒事件队列，除非消息队头存在barrier，并且同时Message是队列中最早的异步消息。
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            for (; ; ) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            msg.next = p; // invariant: p == prev.next
            prev.next = msg;
        }

        // 消息没有退出，我们认为此时mPtr != 0
        if (needWake) {
            nativeWake(mPtr);
        }
    }
    return true;
}
```

`MessageQueue` 是按照 `Message` 触发时间的先后顺序排列的，队头的消息是将要最早触发的消息。当有消息需要加入消息队列时，会从队列头开始遍历，直到找到消息应该插入的合适位置，以保证所有消息的时间顺序。

### removeMessages() ###
**`removeMessages()`**方法用于移除消息队列中所有符合条件的消息，该方法在 `Handler.removeMessages()` 方法中被调用。`removeMessages()` 方法的源码如下：
```java
void removeMessages(Handler h, int what, Object object) {
    // 判断Handler是否为null，若为null则直接返回
    if (h == null) {
        return;
    }

    synchronized (this) {
        Message p = mMessages;

        // 从消息队列的头部开始，移除所有符合条件的消息
        while (p != null && p.target == h && p.what == what
                && (object == null || p.obj == object)) {
            Message n = p.next;
            mMessages = n;
            p.recycleUnchecked();
            p = n;
        }

        // 移除剩余的符合要求的消息
        while (p != null) {
            Message n = p.next;
            if (n != null) {
                if (n.target == h && n.what == what
                        && (object == null || n.obj == object)) {
                    Message nn = n.next;
                    n.recycleUnchecked();
                    p.next = nn;
                    continue;
                }
            }
            p = n;
        }
    }
}
```

`removeMessages()` 方法采用了两个 `while` 循环，第一个循环是从队头开始，移除所有符合条件的消息，第二个循环是从头部移除完连续的满足条件的消息之后，再从队列后面继续查询是否有满足条件的消息需要被移除。

另外，与 `removeMessages()` 方法功能相近的，还有一个重载方法方法，也是用于移除消息队列中所有符合条件的消息。此外，还有一个 `removeCallbacksAndMessages()` 方法，用于移除所有待处理的 `Message.obj` 为指定 `object` 或 `null` 的 `Message` 和 `Runnable`。

### postSyncBarrier() ###
**`postSyncBarrier()`**方法用于向 `Looper` 的消息队列发布同步障碍(创建了一个没有 `target` 的 `Message` 对象并加入到消息队列中)，该方法在 `ViewRootImpl.scheduleTraversals()` 方法中被调用。`postSyncBarrier()` 方法的源码如下：
```java
public int postSyncBarrier() {
    return postSyncBarrier(SystemClock.uptimeMillis());
}

private int postSyncBarrier(long when) {
    // 将新的同步屏障令牌加入队列(不需要唤醒队列，因为障碍的目的是阻止它)
    synchronized (this) {
        final int token = mNextBarrierToken++;
        final Message msg = Message.obtain();
        msg.markInUse();
        msg.when = when;
        msg.arg1 = token;

        Message prev = null;
        Message p = mMessages;
        if (when != 0) {
            while (p != null && p.when <= when) {
                prev = p;
                p = p.next;
            }
        }
        if (prev != null) { // invariant: p == prev.next
            msg.next = p;
            prev.next = msg;
        } else {
            msg.next = p;
            mMessages = msg;
        }
        return token;
    }
}
```

通常都是通过 `Handler` 发送消息的，`Handler` 在 `Message` 加入到消息队列时，都会为 `Message` 设置 `target`，所以每一个普通的 `Message` 都必定有一个 `target`。但是对于特殊的 `message` 是没有 `target`，即 `sync barrier token`，这个特殊消息的作用就是用于拦截同步消息，所以并不会唤醒 `Looper`。

同步消息被添加到消息队列后，消息处理照常进行，直到消息队列遇到已发布的同步屏障。遇到屏障时，队列中的后续同步消息将被挂起(暂停执行)，直到通过调用 `removeSyncBarrier()` 方法并指定标识同步屏障的令牌来移除同步屏障，该方法在 `ViewRootImpl.doTraversal()` 方法中被调用。`removeSyncBarrier()` 方法的源码如下：
```java
public void removeSyncBarrier(int token) {
    // 从队列中移除同步障碍标记(如果队列不再被障碍物阻挡，则将其唤醒)
    synchronized (this) {
        Message prev = null;
        Message p = mMessages;
        // 从消息队列找到target为null并且token相等的Message
        while (p != null && (p.target != null || p.arg1 != token)) {
            prev = p;
            p = p.next;
        }
        if (p == null) {
            throw new IllegalStateException("The specified message queue synchronization "
                    + " barrier token has not been posted or has already been removed.");
        }
        final boolean needWake;
        if (prev != null) {
            prev.next = p.next;
            needWake = false;
        } else {
            mMessages = p.next;
            needWake = mMessages == null || mMessages.target != null;
        }
        p.recycleUnchecked();

        // 如果循环正在退出，那么它已经处于唤醒状态；如果消息没有退出，我们认为此时mPtr != 0
        if (needWake && !mQuitting) {
            nativeWake(mPtr);
        }
    }
}
```

> `postSyncBarrier()` 方法只对同步消息产生影响，对于异步消息没有任何差别。这个机制的作用在于可以马上暂停执行“同步”消息，直到条件允许后通过移除同步屏障来恢复“同步”消息的处理。例如在 `View.invalidate()` 需要执行时，将会设置同步栏挂起所有“同步”消息，直到下一帧准备好显示后移除同步栏。

### native 方法 ###
`MessageQueue` 是消息机制的 Java 层和 C++ 层的连接纽带，大部分核心方法都交给 `native` 层来处理，其中 `MessageQueue` 类中涉及的 `native` 方法如下：
```java
private native static long nativeInit();
private native static void nativeDestroy(long ptr);
private native void nativePollOnce(long ptr, int timeoutMillis);
private native static void nativeWake(long ptr);
private native static boolean nativeIsPolling(long ptr);
private native static void nativeSetFileDescriptorEvents(long ptr, int fd, int events);
```

### 总结 ###
`MessageQueue` 是一种先进先出的数据结构，被用来存储 `Handler` 发送过来的消息。

## Handler ##
**`android.os.Handler`**用于作为发送和处理 `Message` 和 `Runnable` 的处理程序，每个 `Handler` 中都会有一个 `Looper` 和 `MessageQueue` 对象。

### 创建 Handler ###
`Handler` 的构造方法有以下几种：
```java
public Handler()
public Handler(Handler.Callback callback)
public Handler(boolean async)
public Handler(Handler.Callback callback, boolean async)
public Handler(Looper looper)
public Handler(Looper looper, Handler.Callback callback)
public Handler(Looper looper, Handler.Callback callback, boolean async)
```

`Handler` 的构造方法有两种情况：不指定 `Looper` 和指定 `Looper`。`Handler` 类在构造方法中，可指定 `Looper`、`Callback 回调方法`以及`消息的处理方式(同步或异步)`，对于无参的 `Handler`，默认是当前线程的 `Looper`。

#### 不指定 Looper ####
 ```java
public Handler() {
    this(null, false);
}

public Handler(Handler.Callback callback) {
    this(callback, false);
}

public Handler(boolean async) {
    this(null, async);
}

public Handler(Handler.Callback callback, boolean async) {
    if (FIND_POTENTIAL_LEAKS) {
        final Class<? extends Handler> klass = getClass();
        // 匿名类、内部类或本地类都必须申明为static，否则会警告可能出现内存泄露
        if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                (klass.getModifiers() & Modifier.STATIC) == 0) {
            Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
        }
    }

    // 从当前线程的TLS中获取Looper对象
    // 若在工作线程，必须先执行Looper.prepare()，才能获取Looper对象，否则为null
    mLooper = Looper.myLooper();
    if (mLooper == null) {
        throw new RuntimeException(
                "Can't create handler inside thread " + Thread.currentThread()
                        + " that has not called Looper.prepare()");
    }
    // 获取消息队列(来自Looper对象)
    mQueue = mLooper.mQueue;
    // 回调方法
    mCallback = callback;
    // 设置消息是否为异步处理方式
    mAsynchronous = async;
}
```

对于 `Handler` 的不指定 `Looper` 的构造方法，默认采用当前线程 `TLS` 中的 `Looper` 对象，只要执行的 `Looper.prepare()` 方法，那么便可以获取有效的 `Looper` 对象。

#### 指定 Looper ####
 ```java
public Handler(Looper looper) {
    this(looper, null, false);
}

public Handler(Looper looper, Handler.Callback callback) {
    this(looper, callback, false);
}

public Handler(Looper looper, Handler.Callback callback, boolean async) {
    mLooper = looper;
    mQueue = looper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```

### 消息分发 ###
在 `Looper.loop()` 方法中循环处理消息时，若消息队列中有消息时，就会通过 `Message` 的 `target`(即 Handler)，调用 `dispatchMessage()` 方法来分发消息。`dispatchMessage()` 方法的源码如下：
```java
public void dispatchMessage(Message msg) {
    // 判断Message的回调方法是否为null
    if (msg.callback != null) {
        // 当Message存在回调方法时，则执行message.callback.run()
        handleCallback(msg);
    } else {
        // 判断Handler是否被指定了Callback回调
        if (mCallback != null) {
            // 当Handler被指定了Callback回调时，回调handleMessage()方法
            if (mCallback.handleMessage(msg)) {
                // 若Callback的消息处理方法返回true，则直接返回
                return;
            }
        }
        // Handler自身的消息处理方法
        handleMessage(msg);
    }
}
```

消息分发流程如下：
1. 当 `Message` 的回调方法不为空时，则回调方法 `msg.callback.run()`，其中 `callback` 数据类型为 `Runnable`，否则进入步骤2；
2. 当 `Handler` 的 `mCallback` 成员变量不为空时，则回调方法 `mCallback.handleMessage(msg)`，否则进入步骤3；
3. 调用 `Handler` 自身的消息处理方法 `handleMessage()`，该方法默认为空，`Handler` 子类通过重写该方法来完成具体的逻辑。

> 对于很多情况下，消息分发后的处理方法是第3种情况，即 `Handler.handleMessage()`，一般地往往通过重写该方法从而实现自己的业务逻辑。

### 消息生成 ###
**`obtainMessage()`**方法用于从消息池中取出 `Message`，比创建和分配新实例效率更高。`obtainMessage()` 方法的源码如下：
```java
public final Message obtainMessage() {
    return Message.obtain(this);
}

public final Message obtainMessage(int what) {
    return Message.obtain(this, what);
}

public final Message obtainMessage(int what, Object obj) {
    return Message.obtain(this, what, obj);
}

public final Message obtainMessage(int what, int arg1, int arg2) {
    return Message.obtain(this, what, arg1, arg2);
}

public final Message obtainMessage(int what, int arg1, int arg2, Object obj) {
    return Message.obtain(this, what, arg1, arg2, obj);
}
```

从上面的源码可以看到 `Handler.obtainMessage()` 方法最终调用的都是 `Message.obtain()` 方法，都是从消息池中取出 `Message`，其中 `this` 为当前的 `Handler` 对象。

### 消息发送 ###
`Handler` 用于接受、传递和处理的消息分为 `Message` 和 `Runnable` 两种。

#### 发送 Message ####
发送 `Message` 的方式有以下几种：
 - sendMessage()
```java
public final boolean sendMessage(Message msg) {
    return sendMessageDelayed(msg, 0);
}
```

 - sendEmptyMessage()
```java
public final boolean sendEmptyMessage(int what) {
    return sendEmptyMessageDelayed(what, 0);
}
```

 - sendEmptyMessageDelayed()
```java
public final boolean sendEmptyMessageDelayed(int what, long delayMillis) {
    Message msg = Message.obtain();
    msg.what = what;
    return sendMessageDelayed(msg, delayMillis);
}
```

 - sendEmptyMessageAtTime()
```java
public final boolean sendEmptyMessageAtTime(int what, long uptimeMillis) {
    Message msg = Message.obtain();
    msg.what = what;
    return sendMessageAtTime(msg, uptimeMillis);
}
```

 - sendMessageDelayed()
```java
public final boolean sendMessageDelayed(Message msg, long delayMillis) {
    if (delayMillis < 0) {
        delayMillis = 0;
    }
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}
```

 - sendMessageAtTime()
```java
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}
```

 - sendMessageAtFrontOfQueue()
```java
public final boolean sendMessageAtFrontOfQueue(Message msg) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, 0);
}
```
> 该方法通过设置消息的触发时间为 `0`，从而使 `Message` 加入到消息队列的队头。

#### 发送 Runnable ####
发送 `Runnable` 的方式有以下几种：
 - post()
```java
public final boolean post(Runnable r) {
    return sendMessageDelayed(getPostMessage(r), 0);
}
```

 - postAtTime()
```java
public final boolean postAtTime(Runnable r, long uptimeMillis) {
    return sendMessageAtTime(getPostMessage(r), uptimeMillis);
}
```

 - postAtTime()
```java
public final boolean postAtTime(Runnable r, Object token, long uptimeMillis) {
    return sendMessageAtTime(getPostMessage(r, token), uptimeMillis);
}
```

 - postDelayed()
```java
public final boolean postDelayed(Runnable r, long delayMillis) {
    return sendMessageDelayed(getPostMessage(r), delayMillis);
}
```

 - postDelayed()
```java
public final boolean postDelayed(Runnable r, Object token, long delayMillis) {
    return sendMessageDelayed(getPostMessage(r, token), delayMillis);
}
```

 - postAtFrontOfQueue()
```java
public final boolean postAtFrontOfQueue(Runnable r) {
    return sendMessageAtFrontOfQueue(getPostMessage(r));
}
```

从上面的源码可以看到 `Handler` 发送 `Runnable` 的方法最终调用的都是发送 `Message` 的方法，把 `Runnable` 封装成 `Message` 对象的过程是在 `getPostMessage()` 方法中完成的。`getPostMessage()` 方法的源码如下：
```java
private static Message getPostMessage(Runnable r) {
    Message m = Message.obtain();
    m.callback = r;
    return m;
}

private static Message getPostMessage(Runnable r, Object token) {
    Message m = Message.obtain();
    m.obj = token;
    m.callback = r;
    return m;
}
```

#### 总结 ####
`Handler` 发送消息的一系列方法最终调用 `MessageQueue.enqueueMessage(msg, uptimeMillis)` 方法将消息添加到消息队列中，其中 `uptimeMillis` 为系统当前的运行时间，不包括休眠时间。

无论 `Handler` 将 `Message` 还是 `Runnable` 加入 `MessageQueue`，最终都只是将 `Message` 加入到 `MessageQueue`。分析源码可以看到 `Handler` 将 `Runnable` 加入到消息队列的方法内部都是调用 `getPostMessage()` 方法把 `Runnable` 封装成 `Message` 对象的 `callback` 属性，然后再将得到的 `Message` 加入到消息队列的。

### 消息移除 ###
`Handler` 移除消息的方法有以下几种：
```java
public final void removeMessages(int what) {
    mQueue.removeMessages(this, what, null);
}

public final void removeMessages(int what, Object object) {
    mQueue.removeMessages(this, what, object);
}

public final void removeCallbacksAndMessages(Object token) {
    mQueue.removeCallbacksAndMessages(this, token);
}
```

从上面的源码可以看到 `Handler` 移除消息的方法最终调用的都是 `MessageQueue` 的方法。

### 总结 ###
`Handler` 是消息机制中非常重要的辅助类，更多的实现都是 `Message` 和 `MessageQueue` 中的方法，`Handler` 的目的是为了更加方便的使用消息机制。

## 消息机制总结 ##
`Looper` 和 `MessageQueue` 消息循环和消息队列都是属于 `Thread`，而 `Handler` 本身并不具有 `Looper` 和 `MessageQueue`；但是消息系统的建立和交互，是 `Thread` 将 `Looper` 和 `MessageQueue` 交给某个 `Handler` 维护建立消息系统模型。所以消息系统模型的核心就是 `Looper`。消息循环和消息队列都是由 `Looper` 建立的，而建立 `Handler` 的关键就是这个 `Looper`。

### 消息机制图解 ###
最后用一张图，来表示整个消息机制：
![Handler消息处理机制](https://lyl873825813.github.io/medias/android/handler_summary.png)
从上图中可以看出：
 - `Handler` 通过 `sendMessage()` 方法发送 `Message` 到 `MessageQueue` 队列；
 - `Looper` 通过 `loop()` 方法不断提取出达到触发条件的 `Message`，并将 `Message` 交给它的 `target` 来处理；
 - 经过 `dispatchMessage()` 后，交回给 `Handler` 的 `handleMessage()` 来进行相应地处理。
 - 将 `Message` 加入 `MessageQueue` 时，处往管道写入字符，可以会唤醒 `loop` 线程；如果 `MessageQueue` 中没有 `Message`，并处于 `Idle` 状态，则会执行 `IdelHandler` 接口中的方法，往往用于做一些清理性地工作。

### 消息分发优先级 ###
`Handler` 通过 `dispatchMessage()` 方法分发消息：
1. `Message` 的回调方法：`message.callback.run()` 优先级最高；
2. `Handler` 的回调方法：`Handler.mCallback.handleMessage(msg)` 优先级仅次于1；
3. `Handler` 的默认方法：`Handler.handleMessage(msg)` 优先级最低。

### 消息缓存机制 ###
为了提高效率，`Message` 提供了一个大小为 `50` 的缓存池以链表的形式缓存 `Message` 对象，减少对象不断创建与销毁的过程。通过 `Message.obtain()` 方法或 `Handler.obtainMessage()` 方法可以直接从消息池中取出缓存的 `Message` 对象，从而提高效率。

