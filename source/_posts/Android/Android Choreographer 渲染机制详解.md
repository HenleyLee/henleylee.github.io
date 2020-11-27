---
title: Android Choreographer 渲染机制详解
categories: Android
tags:
  - Android
abbrlink: f831dca5
date: 2020-03-24 18:35:26
---

Android 的 UI 渲染性能是 Google 长期以来非常重视的，基本每次 Google I/O 都会花很多篇幅讲这一块。不过随着 Android 系统的不断演进和完善，时至今日，关于 Android UI 卡顿的话题也越来越少。

`VSYNC` 是 `Systrace` 中一个非常关键的机制，虽然我们在操作手机的时候看不见，摸不着，但是在 `Systrace` 中我们可以看到，Android 系统在 `VSYNC` 信号的指引下，有条不紊地进行者每一帧的渲染、合成操作，使我们可以享受稳定帧率的画面。引入 `VSYNC` 之前的 Android 版本，渲染一帧相关的 `Message` ，中间是没有间隔的，上一帧绘制完，下一帧的 `Message` 紧接着就开始被处理。这样的问题就是，帧率不稳定，可能高也可能低，不稳定。

`Choreographer` 是 Android 4.1 新增的机制，主要是配合 `VSYNC` ，给上层 App 的渲染提供一个稳定的 `Message` 处理的时机，也就是 `VSYNC` 到来的时候 ，系统通过对 `VSYNC` 信号周期的调整，来控制每一帧绘制操作的时机。目前大部分手机都是 60Hz 的刷新率，也就是 16.6ms 刷新一次，系统为了配合屏幕的刷新频率，将 `VSYNC` 的周期也设置为 16.6 ms，每个 16.6 ms，`VSYNC` 信号唤醒 `Choreographer` 来做 App 的绘制操作 ，这就是引入 `Choreographer` 的主要作用。

## Choreographer 简介 ##
`Choreographer` 扮演 Android 渲染链路中承上启下的角色：
 - **承上：**负责接收和处理 App 的各种更新消息和回调，等到 `VSYNC` 到来的时候统一处理。比如集中处理 Input(主要是 Input 事件的处理) 、Animation(动画相关)、Traversal(包括 measure、layout、draw 等操作) ，判断卡顿掉帧情况，记录 `CallBack` 耗时等。
 - **启下：**负责请求和接收 `VSYNC` 信号。接收 `VSYNC` 事件回调(通过 `FrameDisplayEventReceiver.onVsync()`)；请求 `VSYNC`(`FrameDisplayEventReceiver.scheduleVsync()`)。

从上面可以看出来， `Choreographer` 担任的是一个工具人的角色，他之所以重要，是因为通过 `Choreographer + SurfaceFlinger + Vsync + TripleBuffer` 这一套从上到下的机制，保证了 Android App 可以以一个稳定的帧率运行(目前大部分是 60fps)，减少帧率波动带来的不适感。

了解 `Choreographer` 还可以帮助 App 开发者知道程序每一帧运行的基本原理，也可以加深对 `Message`、`Handler`、`Looper`、`MessageQueue`、`Measure`、`Layout`、`Draw` 的理解 , 很多 APM 工具也用到了 `Choreographer`(利用 FrameCallback + FrameInfo) + `MessageQueue`(利用 IdleHandler) + `Looper`( 设置自定义 MessageLogging) 这些组合拳，深入了解了这些之后，再去做优化，脑子里的思路会更清晰。

## Choreographer 工作流程 ##
Choreographer 的工作流程如下：
1. Choreographer 初始化
   初始化 FrameHandler ，绑定 Looper
   初始化 FrameDisplayEventReceiver ，与 SurfaceFlinger 建立通信用于接收和请求 Vsync
   初始化 CallBackQueues
2. SurfaceFlinger 的 appEventThread 唤醒发送 Vsync ，Choreographer 回调 FrameDisplayEventReceiver.onVsync , 进入 Choreographer 的主处理函数 doFrame
3. Choreographer.doFrame() 计算掉帧逻辑
4. Choreographer.doFrame() 处理 Choreographer 的第一个 callback ： input
5. Choreographer.doFrame() 处理 Choreographer 的第二个 callback ： animation
6. Choreographer.doFrame() 处理 Choreographer 的第三个 callback ： insets animation
7. Choreographer.doFrame() 处理 Choreographer 的第四个 callback ： traversal
   traversal-draw 中 UIThread 与 RenderThread 同步数据
8. Choreographer.doFrame() 处理 Choreographer 的第五个 callback ： commit ?
9. RenderThread 处理绘制数据，真正进行渲染
10. 将渲染好的 Buffer swap 给 SurfaceFlinger 进行合成

## Choreographer 源码分析 ##
下面从源码的角度来简单看一下，源码只摘抄了部分重要的逻辑，其他的逻辑则被剔除，另外 Native 部分与 SurfaceFlinger 交互的部分也没有列入。

### Choreographer 的初始化 ###

#### Choreographer 的构造函数 ####
`Choreographer` 的构造方法源码如下：
```java
private Choreographer(Looper looper, int vsyncSource) {
    // 当前线程的Looper
    mLooper = looper;
    // 初始化FrameHandler
    mHandler = new FrameHandler(looper);
    // 初始化 DisplayEventReceiver(开启VSYNC后将通过FrameDisplayEventReceiver接收VSYNC脉冲信号)
    mDisplayEventReceiver = USE_VSYNC
            ? new FrameDisplayEventReceiver(looper, vsyncSource)
            : null;
    mLastFrameTimeNanos = Long.MIN_VALUE;

    // 计算一帧的时间，Android手机屏幕采用60Hz的刷新频率(这里是纳秒 ≈16000000ns 还是16ms)
    mFrameIntervalNanos = (long) (1000000000 / getRefreshRate());

    // 初始化CallbackQueue(CallbackQueue中存放要执行的输入、动画、遍历绘制等任务)
    mCallbackQueues = new CallbackQueue[CALLBACK_LAST + 1];
    for (int i = 0; i <= CALLBACK_LAST; i++) {
        mCallbackQueues[i] = new CallbackQueue();
    }
    // b/68769804: For low FPS experiments.
    setFPSDivisor(SystemProperties.getInt(ThreadedRenderer.DEBUG_FPS_DIVISOR, 1));
}
```

`Choreographer` 的构造方法被设计成私有，并且是线程单例的。只能通过其内部的 `getInstance()` 方法获取当前线程的 `Choreographer` 实例：
```java
public static Choreographer getInstance() {
    // Choreographer线程单例的实现方式
    return sThreadInstance.get();
}
```

注意 `USE_VSYNC`，用于判断当前是否启用 `VSYNC` 机制，Android 在 4.1 之后默认开启该机制。
```java
private static final boolean USE_VSYNC = SystemProperties.getBoolean(
        "debug.choreographer.vsync", true);
```

#### Choreographer 的单例初始化 ####
`Choreographer` 通过 `ThreadLocal` 实现 `Choreographer` 的线程单例，`sThreadInstance` 源码如下：
```java
// Thread local storage for the choreographer.
private static final ThreadLocal<Choreographer> sThreadInstance =
        new ThreadLocal<Choreographer>() {
            @Override
            protected Choreographer initialValue() {
                // 获取当前线程的 Looper
                Looper looper = Looper.myLooper();
                if (looper == null) {
                    throw new IllegalStateException("The current thread must have a looper!");
                }
                // 构造 Choreographer 对象
                Choreographer choreographer = new Choreographer(looper, VSYNC_SOURCE_APP);
                if (looper == Looper.getMainLooper()) {
                    mMainInstance = choreographer;
                }
                return choreographer;
            }
        };
```

#### FrameHandler ####
`Choreographer` 的构造必须传递一个 `Looper` 对象，其内部会根据该 `Looper` 创建一个 `FrameHandler`。
```java
private final class FrameHandler extends Handler {
    public FrameHandler(Looper looper) {
        super(looper);
    }

    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
            case MSG_DO_FRAME:
                // 如果启用VSYNC机制，当VSYNC信号到来时触发
                // 执行doFrame()，开始渲染下一帧的操作
                doFrame(System.nanoTime(), 0);
                break;
            case MSG_DO_SCHEDULE_VSYNC:
                // 请求VSYNC信号，例如当前需要绘制任务时
                doScheduleVsync();
                break;
            case MSG_DO_SCHEDULE_CALLBACK:
                // 处理 Callback(需要延迟的任务，最终还是执行上述两个事件)
                doScheduleCallback(msg.arg1);
                break;
        }
    }
}
```

`Choreographer` 的所有任务最终都会发送到该 `Looper` 所在的线程。

#### Choreographer 初始化链 ####
在 Activity 启动过程，执行完 onResume() 后，会调用 `Activity.makeVisible()`，然后再调用到 `addView()`， 层层调用会进入如下方法：
```
ActivityThread.handleResumeActivity(IBinder, boolean, boolean, String) (android.app) 
-->WindowManagerImpl.addView(View, LayoutParams) (android.view) 
    -->WindowManagerGlobal.addView(View, LayoutParams, Display, Window) (android.view) 
        -->ViewRootImpl.ViewRootImpl(Context, Display) (android.view) 
            public ViewRootImpl(Context context, Display display) {
                ......
                mChoreographer = Choreographer.getInstance();
                ......
            }
```

### FrameDisplayEventReceiver ###
`VSYNC` 的注册、申请、接收都是通过 `FrameDisplayEventReceiver` 这个类，`FrameDisplayEventReceiver` 是 `DisplayEventReceiver` 的子类，， 有三个比较重要的方法：
 - **`onVsync()：`**Vsync 信号回调
 - **`scheduleVsync()：`**请求 Vsync 信号
 - **`run()：`**执行 doFrame

`DisplayEventReceiver` 是一个 `abstract class`，在 `DisplayEventReceiver` 的构造方法会通过 `JNI` 创建一个 `IDisplayEventConnection` 的 `VSYNC` 的监听者，`DisplayEventReceiver` 的构造方法源码如下：
```java
public DisplayEventReceiver(Looper looper) {
    this(looper, VSYNC_SOURCE_APP);
}

public DisplayEventReceiver(Looper looper, int vsyncSource) {
    if (looper == null) {
        throw new IllegalArgumentException("looper must not be null");
    }

    mMessageQueue = looper.getQueue();
    // 注册VSYNC信号监听者
    mReceiverPtr = nativeInit(new WeakReference<DisplayEventReceiver>(this), mMessageQueue,
            vsyncSource);

    mCloseGuard.open("dispose");
}
```

另外 `DisplayEventReceiver` 内还包括用于申请 `VSYNC` 信号的 `scheduledVsync()` 方法，源码如下：
```java
public void scheduleVsync() {
    if (mReceiverPtr == 0) {
        Log.w(TAG, "Attempted to schedule a vertical sync pulse but the display event "
                + "receiver has already been disposed.");
    } else {
        // 申请VSYNC中断信号，会回调onVsync()方法
        nativeScheduleVsync(mReceiverPtr);
    }
}
```

 `DisplayEventReceiver` 内还包括用于接收 `VSYNC` 信号的 `onVsync()` 方法，源码如下：
```java
public void onVsync(long timestampNanos, long physicalDisplayId, int frame) {
    // 该方法在其子类FrameDisplayEventReceiver中被重写，目的是通知Choreographer
}
```

这样，当应用需要绘制时，通过 `scheduledVsync()` 方法申请 `VSYNC` 中断，来自 `EventThread` 的 `VSYNC` 信号就可以传递到 `Choreographer`。

`FrameDisplayEventReceiver` 重写了 `onVsync()` 方法，源码如下：
```java
private final class FrameDisplayEventReceiver extends DisplayEventReceiver
        implements Runnable {
    private boolean mHavePendingVsync;
    private long mTimestampNanos;
    private int mFrame;

    //looper 此时也是主线程
    public FrameDisplayEventReceiver(Looper looper, int vsyncSource) {
        super(looper, vsyncSource);
    }

    // 系统native方法会调用此方法
    public void onVsync(long timestampNanos, int builtInDisplayId, int frame) {
        // 判断是否是主显示屏幕(忽略来自非主屏的VSYNC信号)
        if (builtInDisplayId != SurfaceControl.BUILT_IN_DISPLAY_ID_MAIN) {
            Log.d(TAG, "Received vsync from secondary display, but we don't support "
                    + "this case yet.  Choreographer needs a way to explicitly request "
                    + "vsync for a specific display to ensure it doesn't lose track "
                    + "of its scheduled vsync.");
            scheduleVsync();
            return;
        }
        // 修正时间确保时间顺序正确
        long now = System.nanoTime();
        if (timestampNanos > now) {
            Log.w(TAG, "Frame time is " + ((timestampNanos - now) * 0.000001f)
                    + " ms in the future!  Check that graphics HAL is generating vsync "
                    + "timestamps using the correct timebase.");
            timestampNanos = now;
        }

        if (mHavePendingVsync) {
            Log.w(TAG, "Already have a pending vsync event.  There should only be "
                    + "one at a time.");
        } else {
            mHavePendingVsync = true;
        }

        mTimestampNanos = timestampNanos;
        mFrame = frame;
        // 发送消息执行doFrame()处理下一帧 即调用run()方法(注意this，表示当前Runnable)
        Message msg = Message.obtain(mHandler, this);
        msg.setAsynchronous(true);
        mHandler.sendMessageAtTime(msg, timestampNanos / TimeUtils.NANOS_PER_MS);
    }
    
    public void run() {
        mHavePendingVsync = false;
        doFrame(mTimestampNanos, mFrame);
    }
    
}
```

`FrameDisplayEventReceiver` 实现了 `Runnable`，将其作为 `callback` 发送到 `FrameHandler`，此时 `run()` 方法便得到执行并且执行 `doFrame()` 方法。

### Choreographer 处理一帧的逻辑 ###
`Choreographer` 处理绘制的逻辑核心在 `Choreographer.doFrame()` 函数中，从上面的源码可以看到，`FrameDisplayEventReceiver.onVsync()` 方法 post 了自己，其 `run()` 方法直接调用了 `doFrame()` 方法开始一帧的逻辑处理。`doFrame()` 方法的源码如下：
```java
void doFrame(long frameTimeNanos, int frame) {
    final long startNanos;
    synchronized (mLock) {
        if (!mFrameScheduled) {
            // 不是在执行Frame任务直接return
            return;
        }

        // ...

        // 预期执行时间
        long intendedFrameTimeNanos = frameTimeNanos;
        // 当前时间
        startNanos = System.nanoTime();
        final long jitterNanos = startNanos - frameTimeNanos;
        // 超时时间是否超过一帧的时间
        if (jitterNanos >= mFrameIntervalNanos) {
            // 计算掉帧数
            final long skippedFrames = jitterNanos / mFrameIntervalNanos;
            // 掉帧超过30帧打印Log提示
            if (skippedFrames >= SKIPPED_FRAME_WARNING_LIMIT) {
                // 著名的掉帧Log
                // 该Log用于提示开发人员当前存在耗时的任务导致UI绘制掉帧超过30帧（≈16ms*30 >= 480ms）。
                Log.i(TAG, "Skipped " + skippedFrames + " frames!  "
                        + "The application may be doing too much work on its main thread.");
            }
            final long lastFrameOffset = jitterNanos % mFrameIntervalNanos;

            frameTimeNanos = startNanos - lastFrameOffset;
        }

        if (frameTimeNanos < mLastFrameTimeNanos) {
            // 未知原因，居然小于最后一帧的时间
            // 重新申请VSYNC信号
            scheduleVsyncLocked();
            return;
        }

        if (mFPSDivisor > 1) {
            long timeSinceVsync = frameTimeNanos - mLastFrameTimeNanos;
            if (timeSinceVsync < (mFrameIntervalNanos * mFPSDivisor) && timeSinceVsync > 0) {
                scheduleVsyncLocked();
                return;
            }
        }

        mFrameInfo.setVsync(intendedFrameTimeNanos, frameTimeNanos);
        // Frame标志位恢复
        mFrameScheduled = false;
        // 记录最后一帧时间
        mLastFrameTimeNanos = frameTimeNanos;
    }

    try {
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, "Choreographer#doFrame");
        AnimationUtils.lockAnimationClock(frameTimeNanos / TimeUtils.NANOS_PER_MS);

        mFrameInfo.markInputHandlingStart();
        // 先执行CALLBACK_INPUT任务
        doCallbacks(Choreographer.CALLBACK_INPUT, frameTimeNanos);

        mFrameInfo.markAnimationsStart();
        // 再执行CALLBACK_ANIMATION
        doCallbacks(Choreographer.CALLBACK_ANIMATION, frameTimeNanos);
	doCallbacks(Choreographer.CALLBACK_INSETS_ANIMATION, frameTimeNanos);

        mFrameInfo.markPerformTraversalsStart();
        // 其次执行CALLBACK_TRAVERSAL
        doCallbacks(Choreographer.CALLBACK_TRAVERSAL, frameTimeNanos);
        // API Level 23 之后加入，
        doCallbacks(Choreographer.CALLBACK_COMMIT, frameTimeNanos);
    } finally {
        AnimationUtils.unlockAnimationClock();
        Trace.traceEnd(Trace.TRACE_TAG_VIEW);
    }
}
```

注意查看该方法的最后，按照类型顺序触发 `doCallbacks()` 回调相关任务：
```java
doCallbacks(Choreographer.CALLBACK_INPUT, frameTimeNanos);
doCallbacks(Choreographer.CALLBACK_ANIMATION, frameTimeNanos);
doCallbacks(Choreographer.CALLBACK_INSETS_ANIMATION, frameTimeNanos);
doCallbacks(Choreographer.CALLBACK_TRAVERSAL, frameTimeNanos);
doCallbacks(Choreographer.CALLBACK_COMMIT, frameTimeNanos);
```

`doCallbacks()` 方法将根据不同的任务类型依次执行其 `run()` 方法：
```java
void doCallbacks(int callbackType, long frameTimeNanos) {
    CallbackRecord callbacks;
    synchronized (mLock) {
        final long now = System.nanoTime();
        // 根据指定的类型CallbackkQueue中查找到达执行时间的CallbackRecord
        callbacks = mCallbackQueues[callbackType].extractDueCallbacksLocked(
                now / TimeUtils.NANOS_PER_MS);
        if (callbacks == null) {
            return;
        }
        mCallbacksRunning = true;

        if (callbackType == Choreographer.CALLBACK_COMMIT) {
            final long jitterNanos = now - frameTimeNanos;
            Trace.traceCounter(Trace.TRACE_TAG_VIEW, "jitterNanos", (int) jitterNanos);
            if (jitterNanos >= 2 * mFrameIntervalNanos) {
                final long lastFrameOffset = jitterNanos % mFrameIntervalNanos
                        + mFrameIntervalNanos;
                if (DEBUG_JANK) {
                    Log.d(TAG, "Commit callback delayed by " + (jitterNanos * 0.000001f)
                            + " ms which is more than twice the frame interval of "
                            + (mFrameIntervalNanos * 0.000001f) + " ms!  "
                            + "Setting frame time to " + (lastFrameOffset * 0.000001f)
                            + " ms in the past.");
                    mDebugPrintNextFrameTimeDelta = true;
                }
                frameTimeNanos = now - lastFrameOffset;
                mLastFrameTimeNanos = frameTimeNanos;
            }
        }
    }
    try {
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, CALLBACK_TRACE_TITLES[callbackType]);
        // 迭代执行所有任务
        for (CallbackRecord c = callbacks; c != null; c = c.next) {
            if (DEBUG_FRAMES) {
                Log.d(TAG, "RunCallback: type=" + callbackType
                        + ", action=" + c.action + ", token=" + c.token
                        + ", latencyMillis=" + (SystemClock.uptimeMillis() - c.dueTime));
            }
            // 回调CallbackRecord的run()方法，其内部回调Callback的run()方法
            c.run(frameTimeNanos);
        }
    } finally {
        synchronized (mLock) {
            mCallbacksRunning = false;
            do {
                final CallbackRecord next = callbacks.next;
                recycleCallbackLocked(callbacks);
                callbacks = next;
            } while (callbacks != null);
        }
        Trace.traceEnd(Trace.TRACE_TAG_VIEW);
    }
}
```

### Choreographer 执行 Callbacks ###
#### CallbackQueue ####
`CallbackQueue` 用于保存通过 `postCallback()` 添加的任务。目前一共定义了五种任务类型，它们分别是：
 - **`CALLBACK_INPUT：`**优先级最高，和输入事件处理有关。
 - **`CALLBACK_ANIMATION：`**优先级其次，和 Animation 的处理有关
 - **`CALLBACK_INSETS_ANIMATION：`**优先级其次，和 Insets Animation 的处理有关
 - **`CALLBACK_TRAVERSAL：`**优先级最低，和 UI 绘制任务有关
 - **`CALLBACK_COMMIT：`**最后执行，和提交任务有关(在 API Level 23 添加)

优先级的高低和处理顺序有关，每当收到 `VSYNC` 信号时，`Choreographer` 将首先处理 `INPUT` 类型的任务，然后是 `ANIMATION` 类型，最后才是 `TRAVERSAL` 类型。`CallbackQueue` 是一个容量为 4 的数组，分别对应不同的任务类型。

#### CallbackRecord ####
通过 `Choreographer` 添加的任务最后都被封装成 `CallbackRecord`，同种任务之间按照时间顺序以链表的形式保存在 `CallbackQueue` 内。`CallbackRecord` 的源码如下：
```java
private static final class CallbackRecord {

    public CallbackRecord next; // 链表，指向下一个
    public long dueTime;        // 到期时间
    public Object action;       // Runnable or FrameCallback
    public Object token;

    @UnsupportedAppUsage
    public void run(long frameTimeNanos) {
        if (token == FRAME_CALLBACK_TOKEN) {
            // 通过postFrameCallback()或postFrameCallbackDelayed()添加的任务会执行这里
            ((Choreographer.FrameCallback) action).doFrame(frameTimeNanos);
        } else {
            ((Runnable) action).run();
        }
    }
}
```

注意 `token == FRAME_CALLBACK_TOKEN` 表示通过 `postFrameCallback()` 方法添加的任务。这里就是按照 `Callback` 类型回调其 `run()` 方法。

#### Input 回调调用栈 ####
`Input Callback` 一般是执行 `ViewRootImpl.ConsumeBatchedInputRunnable`，源码如下：
```java
final class ConsumeBatchedInputRunnable implements Runnable {
    @Override
    public void run() {
        doConsumeBatchedInput(mChoreographer.getFrameTimeNanos());
    }
}

void doConsumeBatchedInput(long frameTimeNanos) {
    if (mConsumeBatchedInputScheduled) {
        mConsumeBatchedInputScheduled = false;
        if (mInputEventReceiver != null) {
            if (mInputEventReceiver.consumeBatchedInputEvents(frameTimeNanos)
                    && frameTimeNanos != -1) {
                scheduleConsumeBatchedInput();
            }
        }
        doProcessInputEvents();
    }
}
```

`Input` 事件经过处理，最终会传给 `DecorView` 的 `dispatchTouchEvent()`，这就到了常见的的 `Input` 事件分发机制。

#### Animation 回调调用栈 ####
一般接触的多的是调用 `View.postOnAnimation()` 的时候，会使用到 `CALLBACK_ANIMATION`，`View` 的 `postOnAnimation()` 方法源码如下：
```java
public void postOnAnimation(Runnable action) {
    final AttachInfo attachInfo = mAttachInfo;
    // 判断AttachInfo是否为空
    if (attachInfo != null) {
        // 如果不为null，直接调用Choreographer.postCallback()方法
        attachInfo.mViewRootImpl.mChoreographer.postCallback(
                Choreographer.CALLBACK_ANIMATION, action, null);
    } else {
        // 否则加入当前View的等待队列
        getRunQueue().post(action);
    }
}
```

另外 `Choreographer` 的 `FrameCallback` 也是用的 `CALLBACK_ANIMATION`，`Choreographer` 的 `postFrameCallbackDelayed()` 方法源码如下：
```java
public void postFrameCallbackDelayed(Choreographer.FrameCallback callback, long delayMillis) {
    if (callback == null) {
        throw new IllegalArgumentException("callback must not be null");
    }

    postCallbackDelayedInternal(CALLBACK_ANIMATION,
            callback, FRAME_CALLBACK_TOKEN, delayMillis);
}
```

#### ViewRootImpl 调用栈 ####
`View` 的绘制流程是由 `ViewRootImpl` 发起的，源码如下：
```java
void scheduleTraversals() {
    if (!mTraversalScheduled) {
        mTraversalScheduled = true;
        // 为了提高优先级，先postSyncBarrier()
        mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
        mChoreographer.postCallback(
                Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
        if (!mUnbufferedInputDispatch) {
            scheduleConsumeBatchedInput();
        }
        notifyRendererOfFramePending();
        pokeDrawLockIfNeeded();
    }
}

final class TraversalRunnable implements Runnable {
    @Override
    public void run() {
        // View的绘制任务开始(真正开始执行 measure、layout、draw)
        doTraversal();
    }
}

final TraversalRunnable mTraversalRunnable = new TraversalRunnable();

void doTraversal() {
    if (mTraversalScheduled) {
        mTraversalScheduled = false;
        // 这里把 SyncBarrier remove
        mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);

        // 真正开始
        performTraversals();
    }
}

private void performTraversals() {
    // measure 操作
    if (focusChangedDueToTouchMode || mWidth != host.getMeasuredWidth()
            || mHeight != host.getMeasuredHeight() || contentInsetsChanged ||
            updatedConfiguration) {
        performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
    }

    // layout 操作
    if (didLayout) {
        performLayout(lp, mWidth, mHeight);
    }

    // draw 操作
    if (!cancelDraw) {
        performDraw();
    }
}
```

## 总结 ##
 - `Choreographer 是线程单例的，而且必须要和一个 `Looper` 绑定，因为其内部有一个 `Handler` 需要和 `Looper` 绑定，一般是 App 主线程的 `Looper` 绑定。
 - `DisplayEventReceiver` 是一个 `abstract class`，其 JNI 的代码部分会创建一个IDisplayEventConnection 的 `VSYNC` 监听者对象。这样，来自 `AppEventThread` 的 `VSYNC` 中断信号就可以传递给 `Choreographer` 对象了。当 `VSYNC` 信号到来时，`DisplayEventReceiver` 的 `onVsync()` 方法将被调用。
 - `DisplayEventReceiver` 还有一个 `scheduleVsync`` 函数。当应用需要绘制UI时，将首先申请一次 Vsync 中断，然后再在中断处理的 onVsync 函数去进行绘制。
 - `Choreographer` 定义了一个 `FrameCallback` 接口，每当 `VSYNC` 到来时，其 `doFrame()` 函数将被调用。这个接口对 Android Animation 的实现起了很大的帮助作用。
 - `Choreographer` 的主要功能是，当收到 `VSYNC` 信号时，去调用使用者通过 `postCallback()` 设置的回调函数。目前一共定义了五种类型的回调，它们分别是：
  - `CALLBACK_INPUT：`处理输入事件处理有关
  - `CALLBACK_ANIMATION：`处理 Animation 的处理有关
  - `CALLBACK_INSETS_ANIMATION：`处理 Insets Animation 的相关回调
  - `CALLBACK_TRAVERSAL：`处理和 UI 等控件绘制有关
  - `CALLBACK_COMMIT：`处理 Commit 相关回调
 - `ListView` 的 `Item` 初始化(obtain\setup) 会在 `Input` 里面也会在 `Animation` 里面，这取决于 `CALLBACK_INPUT`、`CALLBACK_ANIMATION` 会修改 `View` 的属性，所以要先与 `CALLBACK_TRAVERSAL` 执行。

