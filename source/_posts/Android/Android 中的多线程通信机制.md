---
title: Android 中的多线程通信机制
categories: Android
tags:
  - Android
abbrlink: e116562b
date: 2019-02-24 11:22:36
---

## 线程简介 ##
在 `Java` 中，线程会有那么几种状态：创建，就绪，运行，阻塞，死亡。当应用程序有组件在运行时，`UI 线程`是处于运行状态的。默认情况下，应用的所有组件的操作都是在 `UI线程`里完成的，包括响应用户的操作(触摸、点击等)、组件生命周期方法的调用、UI 的更新等。因此如果 `UI线程`处理阻塞状态时(在线程里做一些耗时的操作，如网络连接等)，就会不能响应各种操作，如果阻塞时间达到5秒，就会让程序处于 `ANR(application not response)` 状态，这时用户就可能退出甚至卸载你的应用，这是我们不能接受的。这时，有人就会说，我们在其他线程中更新 UI 不就可以了吗，就不会导致 UI 线程处于阻塞状态了。但答案是否定的。

因为 Android 的 `UI线程`是非线程安全的，应用更新 UI，是调用 `invalidate()` 方法来实现界面的重绘，而 `invalidate()` 方法是非线程安全的，也就是说当我们在非 UI 线程来更新 UI 时，可能会有其他的线程或 UI 线程也在更新 UI，这就会导致界面更新的不同步。因此我们不能在非 UI 主线程中做更新 UI 的操作。也就是说在使用 Android 中的线程时，要保证：
 - 不能阻塞 UI 主线程，也就是不能在 UI 主线程中做耗时的操作(如网络连接、文件的 I/O 操作)；
 - 只能在 UI 主线程中做更新 UI 的操作。

在 Android 中，把除主线程外的，其他所有的线程都叫做工作线程，也就是说 Android 只会存在两种线程：**`主线程(UI thread)`**和**`工作线程(work thread)`**。
> 主线程主要负责处理和界面有关的事情，而工作线程则往往用于执行耗时操作。

## Android 中的线程 ##
在 Android 中除了 `Thread` 本身以外，可以扮演线程的角色还有很多，比如：`AnsyncTask` 和 `IntentService`，同时 `HandlerThread` 也是一种的特殊的线程，`AnsyncTask` 底层用到了线程池，对于 `IntentService` 和 `HandlerThread` 来说，他们的底层则直接使用了线程。

### AsyncTask ###
**`AsyncTask`**是 Android 提供的一种轻量级的异步任务类，它可以在线程池中执行后台任务，然后把执行进度和最终结果传递给主线程并在主线程中更新 UI，其内部封装了 `Thread` 和 `Handler`，通过 `AsyncTask` 可以更加方便地执行后台任务，以及在主线程中访问 UI，但是不能执行可别耗时的后台任务，对于特别耗时的后台任务，建议使用线程池。

`AsyncTask` 是一个抽象的泛型类，它提供了 `Params`、`Progress`、`Result` 三个参数，`Params` 表示参数类型，`Progerss` 表示后台任务的执行进度的类型，而 `Result` 则表示后台任务的返回结果的类型，如果不需要传递具体的参数，那么这三个参数可以用 `Void` 来代替。

`AsyncTask` 还提供了四个核心方法：
 - `onPreExecute()：`在主线程中执行，在异步任务执行之前，此方法被调用，一般可以用于做一些初始化工作。
 - `doInBackground(Params... params)：`在线程池中执行，此方法用于执行异步任务，params 参数表示异步任务的输入参数，在此方法中可以通过 `publishProgress()` 方法来更新任务进度，`publishProgress()` 方法会调用 `onProgressUpdate()` 方法，另外此方法需要返回执行结果给 `onPostExecute()` 方法。
 - `onProgressUpdate(Progress... values)：`在主线程中执行，当后台任务的执行进度发生改变时此方法会被调用。
 - `onPostExecute(Result result)：`在主线程执行，在异步任务执行后此方法会被调用，其中 result 参数是后台任务的返回值，即 `doInBackground()` 方法的返回值。

除了上述四个方法之外，`AsyncTask` 还提供了 `onCancelled()` 方法，它同样在主线程中执行，当异步任务被取消时，`onCancelled()` 方法会被调用，这个时候 `onPostExecute()` 方法则不会被调用。

使用 `AsyncTask` 需要注意以下几点：
 - `AsyncTask` 类必须在主线程中加载，这就意味着第一次访问 `AsyncTask` 必须发生在主线程；
 - `AsyncTask` 的对象必须在 UI 线程中创建；
 - `execute()` 方法必须在 UI 线程调用；
 - 不要在程序中直接调用 `onPreExecute()`、`onPostExecute()`、`doInBackground()` 和 `onProgressUpdate()` 方法；
 - 一个 `AsyncTask` 对象只能执行一次，即只能调用一次 `execute()` 方法，否则会报运行时异常。

> `AsyncTask` 为主线程与工作线程之间进行快速的切换提供一种简单便捷的机制。适用于当下立即需要启动，但是异步执行的生命周期短暂的使用场景。

### HandlerThread ###
**`HandlerThread`**继承了 `Thread`，它是一种可以使用 `Handler` 的 `Thread`，它的实现很简单，就是在 `run()` 方法中通过 `Looper.prepare()` 来创建消息队列，并通过 `Looper.loop()` 来开启消息循环，这样在实际的使用中就允许在 `HandlerThread` 中创建 `Handler` 了。

`HandlerThread` 的 `run()` 方法实现如下所示：
```java
@Override
public void run() {
    mTid = Process.myTid();
    Looper.prepare();
    synchronized (this) {
        mLooper = Looper.myLooper();
        notifyAll();
    }
    Process.setThreadPriority(mPriority);
    onLooperPrepared();
    Looper.loop();
    mTid = -1;
}
```

从 `HandlerThread` 的实现来看，它和普通 `Thread` 有显著的不同之处，普通 `Thread` 主要用于在 `run()` 方法中执行一个耗时的任务，而 `HandlerThread` 在内部创建了消息队列，外界需要通过 `Handler` 的消息方式来通知 `HandlerThread` 执行一个具体的任务。

创建一个 `HandlerThread` 对象并调用 `HandlerThraed#start()` 方法启动线程，然后调用 `HandlerThread#getLooper()` 方法获取 `Looper` 对象作为参数创建 `Handler` 对象。

`HandlerThread` 是一个很有用的类，它在 Android 中的一个具体的使用场景是 `IntentService`，由于 `HandlerThread` 的 `run()` 方法是一个无限循环，因此当明确不需要再使用 `HandlerThread` 时，可以通过它的，`quit()` 或者 `quitSafely()` 方法来终止线程的执行，这就是一个良好的编程习惯。

> `HandlerThread` 为某些回调方法或者等待某些任务的执行设置一个专属的线程，并提供线程任务的调度机制。

### IntentService ###
**`IntentService`**是一个特殊的 `Service`，它集成了 `Service` 并且它是一个抽象类，因此必须创建它的子类才能使用 `IntentService`。`IntentService` 可用于执行后台耗时的任务，当任务执行后它会自动停止，同时由于 `IntentService` 是服务的原因，这导致了它的优先级比单纯的线程要高的多，所以 `IntentService` 比较适合执行一些高优先级的的后台任务，因为它优先级高不容易被系统杀死。

在实现上，`IntentService` 封装了 `HandlerThread` 和 `Handler`，这一点可以从它的 `onCreate()` 方法中看出来：
```java
@Override
public void onCreate() {
    super.onCreate();
    HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
    thread.start();

    mServiceLooper = thread.getLooper();
    mServiceHandler = new ServiceHandler(mServiceLooper);
}
```

第一次启动 `IntentService` 的时候，它的 `onCreate()` 方法会被调用，会创建一个 `HandlerThread`，然后使用它的 `Looper` 来构造一个 `Handler` 对象，这样通过 `mServiceHandler` 发送的消息最终都会在 `HandlerThread` 中执行，从这个角度来看，`IntentService` 也可以在后台执行任务，每次启动 `IntentService`，它的 `onStartCommand()` 都会调用一次，在这个方法中处理每个后台任务的`Intent`。下面来看一下 `onStartCommand()` 方法是如何处理外界的 `Intent`：
```java
@Override
public int onStartCommand(@Nullable Intent intent, int flags, int startId) {
    onStart(intent, startId);
    return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
}
```

可以看到 `onStartCommand()` 方法调用了 `onStart()` 方法，下面来看一下 `onStart()` 方法的具体实现：
```java
@Override
public void onStart(@Nullable Intent intent, int startId) {
    Message msg = mServiceHandler.obtainMessage();
    msg.arg1 = startId;
    msg.obj = intent;
    mServiceHandler.sendMessage(msg);
}
```

可以看到 `onStart()` 方法仅仅是通过 `mServiceHandler` 发送了一消息，这个消息会在 `HandlerThread` 中处理。下面来看一下 `ServiceHandler` 具体实现：
```java
private final class ServiceHandler extends Handler {
    public ServiceHandler(Looper looper) {
        super(looper);
    }

    @Override
    public void handleMessage(Message msg) {
        onHandleIntent((Intent) msg.obj);
        stopSelf(msg.arg1);
    }
}
```

通过上面的代码可以看到 `mServiceHandler` 收到消息以后，会将 `Intent` 对象传递给 `onHandleIntent()` 方法处理，注意这个 `Intent` 对象的内容和外界的 `startService(intent)` 中的 `intent` 的内容是完全一致的，`IntentService` 也是按顺序执行后台任务的。

> `IntentService` 适合于执行由 `UI` 触发的后台 `Service` 任务，并可以把后台任务执行的情况通过一定的机制反馈给 `UI`。

## Android 线程间通信方式 ##
Android 线程间通信的可以归纳为以下两种情况：
 - 将任务从主线程抛到工作线程
 - 将任务从工作线程抛到主线程

将任务从主线程抛到工作线程可以使用前面提到的线程或线程池的方式实现。下面主要讲解一下将任务从工作线程抛到主线程的方式。

### Activity.runOnUiThread() ###
**`Activity.runOnUiThread()`**是一种常用的将任务从工作线程抛到主线程的方式。下面看一下 `Activity.runOnUiThread()` 方法的具体实现：
```java
public final void runOnUiThread(Runnable action) {
    if (Thread.currentThread() != mUiThread) {
        mHandler.post(action);
    } else {
        action.run();
    }
}
```

通过上面的代码可以看到如果当前线程是 UI 线程，直接运行该 `Runnable` 对象的 `run()` 方法，如果不在 UI 线程，则通过 `Activity` 持有的 `Handler` 对象调用 `post()` 方法。

### View.post() ###
**`View.post()`**也是一种常用的将任务从工作线程抛到主线程的方式。下面看一下 `View.post()` 方法的具体实现：
```java
public boolean post(Runnable action) {
    final AttachInfo attachInfo = mAttachInfo;
    if (attachInfo != null) {
        return attachInfo.mHandler.post(action);
    }

    // Postpone the runnable until we know on which thread it needs to run.
    // Assume that the runnable will be successfully placed after attach.
    getRunQueue().post(action);
    return true;
}
```

通过上面的代码可以看到如果当前 `View` 已经关联到父窗口，直接通过 `AttachInfo` 持有的 `Handler` 对象调用 `post()` 方法，如果该视图还没有关联到父窗口，则将任务添加到待处理任务队列中等待该视图被关联到父窗口时通过 `AttachInfo` 持有的 `Handler` 对象执行。

> `View.postDelayed()` 方法的执行过程和 `View.post()` 方法相同，`View.postDelayed()` 方法除了可以传递 `Runnable` 对象外，还可以传递一个 `delayMillis` 参数表示执行 `Runnable` 之前的延迟(以毫秒为单位)。

### AsyncTask ###
**`AsyncTask`**是 Android 给我们提供的一个处理异步任务的类。通过此类，可以实现 UI 线程和工作线程进行通讯，工作线程执行异步任务并把结果返回给 UI 线程，其内部封装了 `Thread` 和 `Handler`。

> 注意：`AsyncTask` 作为匿名内部类使用时，会隐式地持有外部类的引用，可能会导致内存泄露。比如：在 `Activity` 里声明且实例化一个匿名的 `AsyncTask` 对象，则可能会发生内存泄漏，如果这个线程在 `Activity` 销毁后还一直在后台执行，那这个线程会继续持有这个 `Activity` 的引用从而不会被 `GC` 回收，直到线程执行完成。

### Handler ###
**`Handler`**是 Android 中相当经典的异步消息机制，在 Android 发展的历史长河中扮演着很重要的角色，无论是我们直接面对的应用层还是 FrameWork 层，使用的场景还是相当的多。`Activity.runOnUiThread()`、`View.post()` 以及 `AsyncTask` 内部都是使用的 `Handler` 将任务从工作线程抛到主线程，所以 `Handler` 是一种最常用的将任务从工作线程抛到主线程的方式。

`Handler` 有以下两个主要用途：
 - 将未来某个时间点将要执行的 `Message` 或 `Runnable` 加入到消息队列；
 - 在子线程把需要在另一个线程执行的操作加入到消息队列中去。

`Handler` 常用的构造方法、将 `Message` 或 `Runnable` 加入到消息队列的方法如下所示：
 - `Handler` 的构造方法有以下几种：
```java
public Handler()
public Handler(Handler.Callback callback)
public Handler(Looper looper)
public Handler(Looper looper, Handler.Callback callback)
public Handler(boolean async)
public Handler(Handler.Callback callback, boolean async)
public Handler(Looper looper, Handler.Callback callback, boolean async)
```

 - `Handler` 将 `Message` 加入到消息队列的方法有以下几种：
```java
public final boolean sendMessage(Message msg)
public final boolean sendEmptyMessage(int what)
public final boolean sendEmptyMessageDelayed(int what, long delayMillis)
public final boolean sendEmptyMessageAtTime(int what, long uptimeMillis)
public final boolean sendMessageDelayed(Message msg, long delayMillis)
public final boolean sendMessageAtTime(Message msg, long uptimeMillis)
public final boolean sendMessageAtFrontOfQueue(Message msg)
```

 - `Handler` 将 `Runnable` 加入到消息队列的方法有以下几种：
```java
public final boolean post(Runnable r)
public final boolean postAtTime(Runnable r, long uptimeMillis)
public final boolean postAtTime(Runnable r, Object token, long uptimeMillis)
public final boolean postDelayed(Runnable r, long delayMillis)
public final boolean postDelayed(Runnable r, Object token, long delayMillis)
public final boolean postAtFrontOfQueue(Runnable r)
```

> 无论 `Handler` 将 `Message` 还是 `Runnable` 加入 `MessageQueue`，最终都只是将 `Message` 加入到 `MessageQueue`。分析源码可以看到 `Handler` 将 `Runnable` 加入到消息队列的方法内部都是调用 `getPostMessage()` 方法把 `Runnable` 封装成 `Message` 对象的 `callback` 属性，然后再将得到的 `Message` 加入到消息队列的。

