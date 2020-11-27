---
title: Android View.post() 原理详解
categories: Android
tags:
  - Android
abbrlink: d7d7c7c2
date: 2020-03-02 18:25:36
---

在 Android 开发中，子线程是不能进行 UI 操作的，或者很多场景下，一些操作需要延迟执行，这些都可以通过 `Handler` 来解决。有时候为了避免重新定义一个 `Handler` 对象，经常会用到 `View.post()` 方法或 `View.postDelay()` 方法代替 `Handler` 使用，将一个 `Runnable` 发送到主线程去执行。`View.post()` 内部也是使用的 `Handler`，那么它是如何实现的呢？

## View.post() ##

在 Android 7.0(Api level 24) 上，`View.post()` 方法的内部实现发生了改变。

### API 24 以下 ###
```java
public boolean post(Runnable action) {
    final AttachInfo attachInfo = mAttachInfo;
    // 首先判断AttachInfo是否为null
    if (attachInfo != null) {
        // 如果不为null，直接调用其内部Handler的post()方法
        return attachInfo.mHandler.post(action);
    }
    // 否则加入当前ViewRootImpl的等待队列
    ViewRootImpl.getRunQueue().post(action);
    return true;
}

public boolean postDelayed(Runnable action, long delayMillis) {
    final AttachInfo attachInfo = mAttachInfo;
    // 首先判断AttachInfo是否为null
    if (attachInfo != null) {
        // 如果不为null，直接调用其内部Handler的post()方法
        return attachInfo.mHandler.postDelayed(action, delayMillis);
    }
    // 否则加入当前ViewRootImpl的等待队列
    ViewRootImpl.getRunQueue().postDelayed(action, delayMillis);
    return true;
}
```

如果 `mAttachInfo` 不为空，那就调用 `mAttachInfo.mHanlder.post()` 方法，如果为空，则调用 `ViewRootImpl.getRunQueue().post()` 方法。

`ViewRootImpl` 可以理解是一个 `Activity` 的 `ViewTree` 的根节点的实例，每个 `ViewRootImpl` 就是用来管理 `DecorView` 和 `ViewTree`。`getRunQueue()` 方法的源码如下：
```java
static RunQueue getRunQueue() {
    RunQueue rq = sRunQueues.get();
    if (rq != null) {
        return rq;
    }
    rq = new RunQueue();
    sRunQueues.set(rq);
    return rq;
}
```

`getRunQueue()` 方法返回的是 `RunQueue` 对象，也就是调用了 `RunQueue` 的 `post()` 方法：
```java
static final class RunQueue {

    // 保存被封装的HandlerAction的集合
    private final ArrayList<HandlerAction> mActions = new ArrayList<HandlerAction>();

    void post(Runnable action) {
        postDelayed(action, 0);
    }

    void postDelayed(Runnable action, long delayMillis) {
        // 将Runnable和delayMillis包装成一个HandlerAction对象
        HandlerAction handlerAction = new HandlerAction();
        handlerAction.action = action;
        handlerAction.delay = delayMillis;

        // 将要执行的任务HandlerAction保存在mActions集合中
        synchronized (mActions) {
            mActions.add(handlerAction);
        }
    }

    void executeActions(Handler handler) {
        synchronized (mActions) {
            final ArrayList<HandlerAction> actions = mActions;
            final int count = actions.size();

            for (int i = 0; i < count; i++) {
                final HandlerAction handlerAction = actions.get(i);
                // 调用handler.postDelayed()方法执行Runnable
                handler.postDelayed(handlerAction.action, handlerAction.delay);
            }

            actions.clear();
        }
    }

}
```

`HandlerAction` 表示一个待执行的任务，内部持有要执行的 `Runnable` 和延迟时间，`HandlerAction` 的类声明如下：
```java
private static class HandlerAction {
    Runnable action;   // 要执行的任务
    long delay;        // 延迟时间

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        HandlerAction that = (HandlerAction) o;
        return !(action != null ? !action.equals(that.action) : that.action != null);

    }

    @Override
    public int hashCode() {
        int result = action != null ? action.hashCode() : 0;
        result = 31 * result + (int) (delay ^ (delay >>> 32));
        return result;
    }
}
```

从上面的源码可以看到，`post()` 方法只是单纯的将 `Runnable` 包装成一个 `HandlerAction` 对象，然后放入 `mActions` 这个 `ArrayList` 中，而 `mActions` 中添加的 `HandlerAction` 是在 `executeActions()` 方法中被处理的，最终调用的还是 `handler.postDelayed()`。

### API 24 以上 ###
```java
public boolean post(Runnable action) {
    final AttachInfo attachInfo = mAttachInfo;
    // 首先判断AttachInfo是否为null
    if (attachInfo != null) {
        // 如果不为null，直接调用其内部Handler的post()方法
        return attachInfo.mHandler.post(action);
    }

    // 否则加入当前View的等待队列
    getRunQueue().post(action);
    return true;
}
    
public boolean postDelayed(Runnable action, long delayMillis) {
    final AttachInfo attachInfo = mAttachInfo;
    // 首先判断AttachInfo是否为null
    if (attachInfo != null) {
        // 如果不为null，直接调用其内部Handler的post()方法
        return attachInfo.mHandler.postDelayed(action, delayMillis);
    }

    // 否则加入当前View的等待队列
    getRunQueue().postDelayed(action, delayMillis);
    return true;
}
```

如果 `mAttachInfo` 不为空，那就调用 `mAttachInfo.mHanlder.post()` 方法，如果为空，则调用 `getRunQueue().post()` 方法。`getRunQueue()` 方法的源码如下：
```java
private HandlerActionQueue getRunQueue() {
    if (mRunQueue == null) {
        mRunQueue = new HandlerActionQueue();
    }
    return mRunQueue;
}
```

`getRunQueue()` 方法返回的是 `HandlerActionQueue`，实际就是调用了 `HandlerActionQueue` 的 `post()` 方法：
```java
public class HandlerActionQueue {
    private HandlerAction[] mActions;
    private int mCount;

    public void post(Runnable action) {
        postDelayed(action, 0);
    }

    public void executeActions(Handler handler) {
        synchronized (this) {
            // 任务队列
            final HandlerAction[] actions = mActions;
            // 遍历所有任务
            for (int i = 0, count = mCount; i < count; i++) {
                final HandlerAction handlerAction = actions[i];
                // 调用handler.postDelayed()方法将要执行的Runnable发送到Handler中，等待执行
                handler.postDelayed(handlerAction.action, handlerAction.delay);
            }

            // 将保存任务的mActions置为null（后续View.post()直接添加到AttachInfo内部的Handler）
            mActions = null;
            mCount = 0;
        }
    }

    public int size() {
        return mCount;
    }

}
```

`HandlerAction` 表示一个待执行的任务，内部持有要执行的 `Runnable` 和延迟时间，`HandlerAction` 的类声明如下：
```java
private static class HandlerAction {
    final Runnable action;       // 要执行的任务
    final long delay;            // 延迟时间

    public HandlerAction(Runnable action, long delay) {
        this.action = action;
        this.delay = delay;
    }

    /**
     * 比较是否是同一个任务
     */
    public boolean matches(Runnable otherAction) {
        return otherAction == null && action == null
                || action != null && action.equals(otherAction);
    }
    
}
```


> 很明显的可以看出来，在 API 24 以下和 API 24 以上的源码中，只有在 `mAttachInfo` 为 `null` 的时候，执行的逻辑才会有差异。在 API 24 中，会调用 `getRunQueue().post(action)`，而 API 23 会调用 `ViewRootImpl.getRunQueue().post(action)` 方法，他们的差异就在这里。

## AttachInfo ##
`AttachInfo` 是 `View` 的静态内部类，每个 `View` 都会持有一个 `AttachInfo`，它默认为 null。那么 `AttachInfo` 是在什么时候被赋值的呢？

### AttachInfo构造方法 ###
首先看下 `AttachInfo` 的创建过程，它的构造方法如下：
```java
AttachInfo(IWindowSession session, IWindow window, Display display,
        ViewRootImpl viewRootImpl, Handler handler, Callbacks effectPlayer,
        Context context) {
    mSession = session;
    mWindow = window;
    mWindowToken = window.asBinder();
    mDisplay = display;
    mViewRootImpl = viewRootImpl;
    mHandler = handler;
    mRootCallbacks = effectPlayer;
    mTreeObserver = new ViewTreeObserver(context);
}
```

> `AttachInfo` 中持有当前线程的 `Handler`。

### View.dispatchAttachedToWindow() ###
查看 `View` 的源码，可以发现仅有两处对 `mAttachInfor` 赋值操作，一处是在 `dispatchAttachedToWindow()` 方法中为其赋值，另一处是在 `dispatchDetachedFromWindow()` 方法中将其置为 `null`。`View` 的 `dispatchAttachedToWindow()` 方法的源码如下：
```java
void dispatchAttachedToWindow(AttachInfo info, int visibility) {
    // 给当前View赋值AttachInfo，此时所有的View共用同一个AttachInfo（同一个ViewRootImpl内）
    mAttachInfo = info;
    // View浮层，是在Android 4.3添加的
    if (mOverlay != null) {
        // 任何一个View都有一个ViewOverlay，ViewGroup的是ViewGroupOverlay
        // 它区别于直接在类似RelativeLaout/FrameLayout添加View，通过ViewOverlay添加的元素没有任何事件
        // 此时主要分发给这些View浮层
        mOverlay.getOverlayView().dispatchAttachedToWindow(info, visibility);
    }
    mWindowAttachCount++;

    // ...

    if ((mPrivateFlags & PFLAG_SCROLL_CONTAINER) != 0) {
        mAttachInfo.mScrollContainers.add(this);
        mPrivateFlags |= PFLAG_SCROLL_CONTAINER_ADDED;
    }
    // mRunQueue的实际类型是HandlerActionQueue，内部保存了当前View.post()的任务
    if (mRunQueue != null) {
        // 调用executeActions()方法执行所有待执行的任务（post到渲染线程的Handler中）
        mRunQueue.executeActions(info.mHandler);
        // 保存延迟任务的队列被置为null
        mRunQueue = null;
    }
    performCollectViewAttributes(mAttachInfo, visibility);
    // 回调View的onAttachedToWindow方法（在Activity的onResume方法中调用，但是在View绘制流程之前）
    onAttachedToWindow();

    ListenerInfo li = mListenerInfo;
    final CopyOnWriteArrayList<OnAttachStateChangeListener> listeners =
            li != null ? li.mOnAttachStateChangeListeners : null;
    if (listeners != null && listeners.size() > 0) {
        // 通知所有监听View已经onAttachToWindow的客户端，即view.addOnAttachStateChangeListener();
        // 但此时View还没有开始绘制，不能正确获取测量大小或View实际大小
        for (OnAttachStateChangeListener listener : listeners) {
            listener.onViewAttachedToWindow(this);
        }
    }

    // ...

    // 回调View的onVisibilityChanged（注意这时候View绘制流程还未真正开始）
    onVisibilityChanged(this, visibility);

    // ...
}
```

`dispatchAttachedToWindow()` 方法最开始为当前 `View` 赋值 `AttachInfo`。注意 `mRunQueue` 就是保存了 `View.post()` 任务的 `HandlerActionQueue`，此时调用 `HandlerActionQueue` 的 `executeActions()` 方法将所有待执行的任务发送到当前线程的 `Handler` 中等待执行。

### View.dispatchDetachedFromWindow() ###
`dispatchDetachedFromWindow()` 首先回调 `View` 的 `onDetachedFromWindow()` 方法，然后通知所有监听者 `onViewDetachedFromWindow()`，最后将 `mAttachInfo` 置为 `null`。`View` 的 `dispatchDetachedFromWindow()` 方法的源码如下：
```java
void dispatchDetachedFromWindow() {
    AttachInfo info = mAttachInfo;
    if (info != null) {
        int vis = info.mWindowVisibility;
        if (vis != GONE) {
            // 通知 Window显示状态发生变化
            onWindowVisibilityChanged(GONE);
            if (isShown()) {
                onVisibilityAggregated(false);
            }
        }
    }

    // 回调View的onDetachedFromWindow
    onDetachedFromWindow();
    onDetachedFromWindowInternal();

    // ...

    ListenerInfo li = mListenerInfo;
    final CopyOnWriteArrayList<OnAttachStateChangeListener> listeners =
            li != null ? li.mOnAttachStateChangeListeners : null;
    if (listeners != null && listeners.size() > 0) {
        // 通知所有监听View已经onAttachToWindow()的客户端，即view.addOnAttachStateChangeListener()
        for (OnAttachStateChangeListener listener : listeners) {
            listener.onViewDetachedFromWindow(this);
        }
    }

    // ...

    // 将AttachInfo置为null
    mAttachInfo = null;
    // 通知浮层View
    if (mOverlay != null) {
        mOverlay.getOverlayView().dispatchDetachedFromWindow();
    }

    notifyEnterOrExitForAutoFillIfNeeded(false);
}
```

## ViewRootImpl ##
同一个 `View Hierachy` 树结构中所有 `View` 共用一个 `AttachInfo`，`AttachInfo` 的创建是在 `ViewRootImpl` 的构造方法中：
```java
mAttachInfo = new View.AttachInfo(mWindowSession, mWindow, display, this, mHandler, this,
        context);
```

> 一般 `Activity` 包含多个 `View` 形成 `View Hierachy` 的树形结构，只有最顶层的 `DecorView` 才是对 `WindowManagerService` “可见的”。

### ViewRootImpl.performTraversals() ###
`dispatchAttachedToWindow()` 的调用时机是在 `View` 绘制流程的开始阶段。在 `ViewRootImpl` 的 `performTraversals()` 方法中，将会依次完成 `View` 绘制流程的三大阶段：测量、布局和绘制。`performTraversals()` 方法的源码如下：
```java
// View 绘制流程开始在 ViewRootImpl
private void performTraversals() {
    // mView是DecorView
    final View host = mView;
    if (mFirst) {
        // ...
        // host为DecorView
        // 调用DecorView的dispatchAttachedToWindow()方法，并把mAttachInfo给子View
        host.dispatchAttachedToWindow(mAttachInfo, 0);
        mAttachInfo.mTreeObserver.dispatchOnWindowAttachedChange(true);
        dispatchApplyInsets(host);
        // ...
    }
    mFirst = false;
    // ...
    // 执行ViewRootImpl的任务队列中待执行的任务(API 24之前添加的任务在这里执行)
    getRunQueue().executeActions(mAttachInfo.mHandler);
    // View 绘制流程的测量阶段
    performMeasure();
    // View 绘制流程的布局阶段
    performLayout();
    // View 绘制流程的绘制阶段
    performDraw();
    // ...
}
```

`performTraversals()` 方法中的 `host` 的实际类型是 `DecorView`，`DecorView` 继承自 `FrameLayout`。

> 每个 `Activity` 都有一个关联的 `Window` 对象，用来描述应用程序窗口，每个窗口内部又包含一个 `DecorView` 对象，`DecorView` 对象用来描述窗口的视图 — xml 布局。通过 `setContentView()` 方法设置的 `View` 布局最终添加到 `DecorView` 的 `content` 容器中。

### ViewGroup.dispatchAttachedToWindow() ###
`DecorView` 并没有重写 `dispatchAttachedToWindow()` 方法，具体实现是在其父类 `ViewGroup` 中。`ViewGroup` 的 `dispatchAttachedToWindow()` 方法的源码如下：
```java
void dispatchAttachedToWindow(AttachInfo info, int visibility) {
    mGroupFlags |= FLAG_PREVENT_DISPATCH_ATTACHED_TO_WINDOW;
    super.dispatchAttachedToWindow(info, visibility);
    mGroupFlags &= ~FLAG_PREVENT_DISPATCH_ATTACHED_TO_WINDOW;

    // 子View的数量
    final int count = mChildrenCount;
    // 子View的数组
    final View[] children = mChildren;
    // 遍历所有子View
    for (int i = 0; i < count; i++) {
        final View child = children[i];
        // 遍历调用所有子View的dispatchAttachedToWindow()为每个子View关联AttachInfo
        child.dispatchAttachedToWindow(info,
                combineVisibility(visibility, child.getVisibility()));
    }
    // ...
}
```
分析源码可以发现，`dispatchAttachedToWindow()` 方法中通过循环遍历当前 `ViewGroup` 的所有 `childView`，为其关联 `AttachInfo`。子 `View` 的 `dispatchAttachedToWindow()` 方法在前面我们已经分析过了：首先为当前 `View` 关联 `AttachInfo`，然后将之前 `View.post()` 保存的任务添加到 `AttachInfo` 内部的 `Handler`。

因为这一过程是在 `View` 的绘制任务中，所以通过 `View.post()` 添加的任务，是在 `View` 绘制流程的开始阶段，将所有任务重新发送到消息队列的尾部，此时相关任务的执行已经在 `View` 绘制任务之后，即 `View` 绘制流程已经结束，此时便可以正确获取到 `View` 的宽高了。`View.post()` 添加的任务能够保证在所有 `View`（同一个 `View Hierachy` 内） 绘制流程结束之后才被执行。

> 在 API 24 之前，通过 `View.post()` 任务被直接添加到 `ViewRootImpl` 中，在 API 24 及以后，每个 `View` 自行维护待执行的 `post()` 任务，它们要依赖于 `dispatchAttachedToWindow()` 方法，如果 `View` 未添加到窗口视图，`post()` 方法添加的任务将永远得不到执行。

### ViewGroup.dispatchDetachedFromWindow() ###
`DecorView` 也没有重写 `dispatchDetachedFromWindow()` 方法，具体实现是在其父类 `ViewGroup` 中。`ViewGroup` 的 `dispatchDetachedFromWindow()` 方法的源码如下：
```java
void dispatchDetachedFromWindow() {

    // ...

    // 子View的数量
    final int count = mChildrenCount;
    // 子View的数组
    final View[] children = mChildren;
    // 遍历所有childView
    for (int i = 0; i < count; i++) {
        // 遍历调用所有子View的dispatchDetachedFromWindow()将每个子View的AttachInfo置为null
        children[i].dispatchDetachedFromWindow();
    }
    // ...

    super.dispatchDetachedFromWindow();
}
```

`ViewGroup` 的 `dispatchDetachedFromWindow()` 方法会遍历所有 `childView`，将其字 `View` 的 `AttachInfo` 置为 `null`。子 `View` 的 `dispatchDetachedFromWindow()` 方法在前面我们已经分析过了：首先回调 `View` 的 `onDetachedFromWindow()` 方法，然后通知所有监听者 `onViewDetachedFromWindow()`，最后将 `mAttachInfo` 置为 `null`。

### ViewRootImpl.doDie() ###
由于 `dispatchAttachedToWindow()` 方法是在 `ViewRootImpl` 中完成，此时很容易想到它的释放过程肯定也在 `ViewRootImpl`，跟踪发现 `dispatchAttachedToWindow()` 方法在 `ViewRootImpl` 的 `doDie()` 方法中被调用，调用过程如下：
```java
void doDie() {
    // 检查执行线程
    checkThread();
    synchronized (this) {
        if (mRemoved) {
            return;
        }
        mRemoved = true;
        if (mAdded) {
            // 回调View的dispatchDetachedFromWindow()方法
            dispatchDetachedFromWindow();
        }

        if (mAdded && !mFirst) {
            destroyHardwareRenderer();
            // mView是DecorView
            if (mView != null) {
                int viewVisibility = mView.getVisibility();
                // 窗口状态是否发生变化
                boolean viewVisibilityChanged = mViewVisibility != viewVisibility;
                if (mWindowAttributesChanged || viewVisibilityChanged) {
                    try {
                        if ((relayoutWindow(mWindowAttributes, viewVisibility, false)
                                & WindowManagerGlobal.RELAYOUT_RES_FIRST_TIME) != 0) {
                            mWindowSession.finishDrawing(mWindow);
                        }
                    } catch (RemoteException e) {
                    }
                }
                // 释放画布
                destroySurface();
            }
        }

        mAdded = false;
    }
    // 将DecorView从WindowManagerGlobal中移除，并将ViewRootImpl从WindowManager中移除
    WindowManagerGlobal.getInstance().doRemoveView(this);
}
```

### ActivityThread.handleResumeActivity() ###
经过分析已经知道，`AttachInfo` 的赋值操作是在 `View` 绘制任务的开始阶段，而它的调用者是 `ActivityThread` 的 `handleResumeActivity()` 方法，即 `Activity` 生命周期 `onResume()` 方法之后。
```java
public void handleResumeActivity(IBinder token, boolean finalStateRequest, boolean isForward,
        String reason) {

    // 回调Activity的onResume()方法
    final ActivityClientRecord r = performResumeActivity(token, finalStateRequest, reason);

    // ...

    final Activity a = r.activity;

    boolean willBeVisible = !a.mStartedActivity;

    // 当Window为空时创建窗体
    if (r.window == null && !a.mFinished && willBeVisible) {
        r.window = r.activity.getWindow();
        View decor = r.window.getDecorView();
        decor.setVisibility(View.INVISIBLE);
        ViewManager wm = a.getWindowManager();
        WindowManager.LayoutParams l = r.window.getAttributes();
        a.mDecor = decor;
        l.type = WindowManager.LayoutParams.TYPE_BASE_APPLICATION;
        l.softInputMode |= forwardBit;
        if (r.mPreserveWindow) {
            a.mWindowAdded = true;
            r.mPreserveWindow = false;
            ViewRootImpl impl = decor.getViewRootImpl();
            if (impl != null) {
                impl.notifyChildRebuilt();
            }
        }
        if (a.mVisibleFromClient) {
            if (!a.mWindowAdded) {
                // 将DecorView添加到窗体，它是用的WindowManager来添加的
                a.mWindowAdded = true;
                wm.addView(decor, l);
            } else {
                a.onWindowAttributesChanged(l);
            }
        }
    }

    // ...

    if (!r.activity.mFinished && willBeVisible && r.activity.mDecor != null && !r.hideForNow) {
        // ...
        WindowManager.LayoutParams l = r.window.getAttributes();
        if ((l.softInputMode
                & WindowManager.LayoutParams.SOFT_INPUT_IS_FORWARD_NAVIGATION)
                != forwardBit) {
            l.softInputMode = (l.softInputMode
                    & (~WindowManager.LayoutParams.SOFT_INPUT_IS_FORWARD_NAVIGATION))
                    | forwardBit;
            if (r.activity.mVisibleFromClient) {
                ViewManager wm = a.getWindowManager();
                View decor = r.window.getDecorView();
                wm.updateViewLayout(decor, l);
            }
        }

        r.activity.mVisibleFromServer = true;
        mNumVisibleActivities++;
        if (r.activity.mVisibleFromClient) {
            r.activity.makeVisible();
        }
    }

    // ...

    Looper.myQueue().addIdleHandler(new Idler());
}
```

### ActivityThread.handleDestroyActivity() ###
移除 `Window` 窗口任务是通过 `ActivityThread` 完成的，具体调用在 `handleDestoryActivity()` 方法完成：
```java
public void handleDestroyActivity(IBinder token, boolean finishing, int configChanges,
        boolean getNonConfigInstance, String reason) {
    // 回调Activity的onDestory()方法
    ActivityClientRecord r = performDestroyActivity(token, finishing,
            configChanges, getNonConfigInstance, reason);
    if (r != null) {
        cleanUpPendingRemoveWindows(r, finishing);
        // 获取当前Window的WindowManager， 实际是WindowManagerImpl
        WindowManager wm = r.activity.getWindowManager();
        // 当前Window的DecorView
        View v = r.activity.mDecor;
        if (v != null) {
            if (r.activity.mVisibleFromServer) {
                mNumVisibleActivities--;
            }
            IBinder wtoken = v.getWindowToken();
            // 判断Window是否添加到WindowManager
            if (r.activity.mWindowAdded) {
                if (r.mPreserveWindow) {
                    r.mPendingRemoveWindow = r.window;
                    r.mPendingRemoveWindowManager = wm;
                    r.window.clearContentView();
                } else {
                    // 通知WindowManager移除当前Window窗口
                    wm.removeViewImmediate(v);
                }
            }
            // ...

            // 将当前Window的DecorView置为null
            r.activity.mDecor = null;
        }

        // ...
    }

}
```

`AttachInfo` 的释放操作是在 `Activity` 生命周期 `onDestory()` 方法之后，在整个 `Activity` 的生命周期内都可以正常使用 `View.post()` 任务。











`executeAction()` 方法是被 `TraversalRunnable` 调用 `doTraversa()` 方法，在 `doTraversa()` 方法中进行调用的。
而 `TraversalRunnable` 又是通过 `Choreographer.postCallBack()` 去循环调用的。
这个 `Choreographer` 通过 `doScheduleCallback()` 发送一个 `MSG_DO_SCHEDULE_CALLBACK` 类型的消息循环调用，间隔就是一个 `VSync` 的间隔。


https://www.cnblogs.com/dasusu/p/8047172.html
https://www.cnblogs.com/plokmju/p/7481727.html
https://mp.weixin.qq.com/s?__biz=MzAxMTI4MTkwNQ==&mid=2650831953&idx=1&sn=476ae9f31dfe10676cb4dfbb8c49d00c&chksm=80b7a8cfb7c021d99cac398ac9d03d7901e80030f44e3b21653061dcc4f7d06602e7c764f043&scene=21#wechat_redirect




