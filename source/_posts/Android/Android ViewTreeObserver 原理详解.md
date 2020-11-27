---
title: Android ViewTreeObserver 原理详解
categories: Android
tags:
  - Android
abbrlink: 57966f8d
date: 2020-03-12 18:45:56
---

`View` 和 `ViewGroup` 是 Android UI 的基本组件， 而 `ViewGroup` 作为容器，可以包含一组 `View`， 并且 `ViewGroup` 其本身就是 `View` 的扩展。

`ViewTree`：视图树。在 Android 中，所有视图由 `View` 和 `View` 的子类组成。`ViewGroup` 也是 `View` 的子类，它是 `View` 的容器，它可以装载 `View` 和 `ViewGroup`。这样 `ViewGroup` 和 `View` 以树形结构一层一层的嵌套组合，就形成了视图树。

`Observer`：观察者。使用了观察者的设计模式，在这里，即 `ViewTree` 时被观察者，`ViewTreeObserver` 是观察者，通过 `ViewTreeObserver` 注册监听来观察 `ViewTree` 的变化，当 `ViewTree` 发生变化，就会调用 `ViewTreeObserver` 的相关方法来通知其这一改变。可以在 `ViewTreeObserver` 中 `add` 自己的监听器，从而得到 `ViewTree` 的某一变化的通知做出自己的逻辑处理。

## ViewTreeObserver ##
[`ViewTreeObserver`](https://developer.android.com/reference/android/view/ViewTreeObserver) 是用于注册在 `ViewTree` 状态发生变化时可被**全局通知**的 `Listener`。这些 `Listeners` 包括视图树的布局、视图树的焦点、视图树将要绘制、视图树滚动等发生变化等场景。

`ViewTreeObserver` 不能被外部实例化，因为它是由视图提供，只能通过 `android.view.View#getViewTreeObserver()` 方式获取。

## 公共接口 ##
`ViewTreeObserver` 为应用提供了全局监听的 `View` 状态变化，那具体可以帮助我们监听哪些状态呢？

| 序号 | Interface                   | 作用                                                                   |
| :--: | :-------------------------- | :--------------------------------------------------------------------- |
| 1    | OnWindowAttachListener      | 当视图层次结构关联到窗口或与之分离时回调                               |
| 2    | OnWindowFocusChangeListener | 当视图层次结构的窗口焦点状态发生改变时回调                             |
| 3    | OnGlobalFocusChangeListener | 当视图树中的焦点状态发生改变时回调                                     |
| 4    | OnGlobalLayoutListener      | 当视图树全局布局状态发生改变或视图树中某个视图的可视状态发生改变时回调 |
| 5    | OnPreDrawListener           | 当视图树即将开始绘制时回调                                             |
| 6    | OnDrawListener              | 当视图树即将开始绘制时回调                                             |
| 7    | OnTouchModeChangeListener   | 当视图树的触摸模式发生改变时回调                                       |
| 8    | OnScrollChangedListener     | 当视图树中的一些组件发生滚动时回调                                     |
| 9    | ...                         | 被 @hide 声明                                                          |

上述接口在 `ViewTreeObserver` 中的管理：
```java
    private CopyOnWriteArrayList<OnWindowFocusChangeListener> mOnWindowFocusListeners;
    private CopyOnWriteArrayList<OnWindowAttachListener> mOnWindowAttachListeners;
    private CopyOnWriteArrayList<OnGlobalFocusChangeListener> mOnGlobalFocusListeners;
    private CopyOnWriteArrayList<OnTouchModeChangeListener> mOnTouchModeChangeListeners;
    private CopyOnWriteArray<OnGlobalLayoutListener> mOnGlobalLayoutListeners;
    private CopyOnWriteArray<OnScrollChangedListener> mOnScrollChangedListeners;
    private CopyOnWriteArray<OnPreDrawListener> mOnPreDrawListeners;
    private ArrayList<OnDrawListener> mOnDrawListeners;
```

`ViewTreeObserver` 内部使用 `CopyOnWriteArrayList` 保存相关 `Listener`，这主要考虑可能出现的多线程场景，不过熟悉它的朋友知道，之所以叫做 `CopyOnWrite` 就是当添加元素的时候，不直接向当前容器添加，而是先将当前容器进行 Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。如果频发的申请与释放其实对内存还是有一定影响的。

`ViewTreeObserver` 不能被外部实例化，只能通过 `view.getViewTreeObserver()` 方式获取：
```java
public ViewTreeObserver getViewTreeObserver() {
    if (mAttachInfo != null) {
        // 在绘制流程开始时，才会被赋值，
        // 此时所有的View共用ViewRootImpl中提供的ViewTreeObserver
        return mAttachInfo.mTreeObserver;
    }
    if (mFloatingTreeObserver == null) {
        // 懒加载，为当前View创建ViewTreeObserver
        mFloatingTreeObserver = new ViewTreeObserver(mContext);
    }
    return mFloatingTreeObserver;
}
```

注意 `mAttachInfo`，它的类型是 `AttachInfo`，内部持有一个 `ViewTreeObserver`，`mAttachInfo` 在 `View` 绘制任务的开始阶段被赋值，但是在此之前，每个 `View` 会临时创建一个 `ViewTreeObserver` 用于保存当前添加的 `Listener`。

`View` 一旦被关联 `AttachInfo`，它会合并该 `View` 的 `ViewTreeObserver`，后续的添加的 `Listener` 都被保存在该 `AttachInfo` 内部的 `ViewTreeObserver`，即同一个 `View Hierarchy` 内所有 `View` 共用同一个 `ViewTreeObserver`。该过程在后面会分析到。

获取到 `ViewTreeObserver` 后就可以添加 `Listener` 了，`ViewTreeObserver` 内部对所有保存 `Listener` 的容器均采用懒加载方式管理，这里仅以 `OnWindowAttachListener` 为例：
```java
public void addOnWindowAttachListener(OnWindowAttachListener listener) {
    // 检查当前ViewTreeObserver是否可用
    // 当绘制流程开始时，所有View内部的ViewTreeObserver替换为
    // ViewRootImpl内部的，故在同一个View Hierarchy内共用同一个。
    checkIsAlive();

    if (mOnWindowAttachListeners == null) {
        // 当使用时在创建
        mOnWindowAttachListeners
                = new CopyOnWriteArrayList<OnWindowAttachListener>();
    }

    mOnWindowAttachListeners.add(listener);
}
```

具体的 `Listener` 是在什么时机被回调的呢？既然所有 `View`（同一个 `View Hierarchy` 内） 的 `ViewTreeObserver` 最终要被合并到 `AttachInfo` 内的 `ViewTreeObserver`，在合并之前肯定不会被调用，所以，此时有必要去跟踪下 `AttachInfo` 被关联的时机：
```java
/**执行View绘制任务*/
private void performTraversals() {
    // ...

    // host是DecorView
    // 每个ViewRootImpl包含一个AttachInfo
    // 为所有的childView关联AttachInfo，即同一个View Hierarchy内共用一个AttachInfo
    // AttachInfo 中包含ViewTreeObserver
    host.dispatchAttachedToWindow(mAttachInfo, 0);
    // 回调 OnWindowAttachedListener
    mAttachInfo.mTreeObserver.dispatchOnWindowAttachedChange(true);

    // ...

    // 开始测量
    performMeasure();
    ...

    // 开始布局
    performLayout();

    // ...

    // 开始绘制
    performDraw();

    ...
}
```

在 `Activity` 中，`View` 绘制任务的开始时机是在 `ActivityThread` 的 `handleResumeActivity()` 方法，在该方法首先完成 `onResume()` 生命周期方法回调。然后发起 `View` 绘制任务，最终会执行到 `ViewRootImpl` 的 `performTraversals` 方法，在该方法真正开始执行 `View` 的三大绘制流程。

> 注意上面的 `host`，它的实际类型是 `DecorView`。

每个 `Activity` 都有一个关联的 `Window` 对象(实际类型是 `PhoneWindow`)，用来描述应用程序窗口，每个窗口内部又包含一个 `DecorView` 对象(继承自 `FrameLayout`)，`DecorView` 用来描述窗口的根视图 — `xml` 布局。

这里实际调用其父类 `ViewGroup` 的 `dispatchAttachedToWindow()` 方法，参数 `mAttachInfo` 很关键，看下它在 `ViewRootImpl` 中的声明：
```java
mAttachInfo = new View.AttachInfo(mWindowSession, mWindow, display, this, mHandler, this,
	context);
```

`ViewGroup` 的 `dispatchAttachedToWindow()` 方法如下：
```java
/**
 * ViewGroup重写View的dispatchAttachedToWindow
 */
void dispatchAttachedToWindow(AttachInfo info, int visibility) {
    mGroupFlags |= FLAG_PREVENT_DISPATCH_ATTACHED_TO_WINDOW;
    super.dispatchAttachedToWindow(info, visibility);
    mGroupFlags &= ~FLAG_PREVENT_DISPATCH_ATTACHED_TO_WINDOW;

    final int count = mChildrenCount;
    final View[] children = mChildren;
    // 遍历所有childView
    for (int i = 0; i < count; i++) {
        final View child = children[i];
        // 调用子View的dispatchAttachedToWindow
        child.dispatchAttachedToWindow(info,
                combineVisibility(visibility, child.getVisibility()));
    }
    final int transientCount = mTransientIndices == null ? 0 : mTransientIndices.size();
    for (int i = 0; i < transientCount; ++i) {
        View view = mTransientViews.get(i);
        view.dispatchAttachedToWindow(info,
                combineVisibility(visibility, view.getVisibility()));
    }
}
```

可以看到遍历所有 `childView`，将 `ViewRootImpl` 的 `AttachInfo` 作为参数调用子 `View` 的 `dispatchAttachedToWindow()` 方法：
```java
/**
 * View的dispatchAttachedToWindow
 */
void dispatchAttachedToWindow(AttachInfo info, int visibility) {
    // 给当前View赋值AttachInfo，此时所有的View共用同一个AttachInfo（同一个ViewRootImpl内）
    mAttachInfo = info;
    // View浮层，是在Android 4.3添加的
    if (mOverlay != null) {
        // 任何一个View都有一个ViewOverlay
        // ViewGroup的是ViewGroupOverlay
        // 它区别于直接在类似RelativeLaout/FrameLayout添加View，通过ViewOverlay添加的元素没有任何事件
        // 此时主要分发给这些View浮层
        mOverlay.getOverlayView().dispatchAttachedToWindow(info, visibility);
    }
    mWindowAttachCount++;
    // We will need to evaluate the drawable state at least once.
    mPrivateFlags |= PFLAG_DRAWABLE_STATE_DIRTY;
    if (mFloatingTreeObserver != null) {
        // 合并当前View的ViewTreeObserver到ViewRootImpl的ViewTreeObserver
        info.mTreeObserver.merge(mFloatingTreeObserver);
        mFloatingTreeObserver = null;
    }

    registerPendingFrameMetricsObservers();

    if ((mPrivateFlags & PFLAG_SCROLL_CONTAINER) != 0) {
        mAttachInfo.mScrollContainers.add(this);
        mPrivateFlags |= PFLAG_SCROLL_CONTAINER_ADDED;
    }
    // Transfer all pending runnables.
    if (mRunQueue != null) {
        // 执行使用View.post的任务，info是AttachInfo
        mRunQueue.executeActions(info.mHandler);
        // 保存延迟任务的队列被置为null，因为此时所有的View共用AttachInfo
        mRunQueue = null;
    }
    performCollectViewAttributes(mAttachInfo, visibility);
    // 回调View的onAttachedToWindow()方法
    // 在Activity的onResume()方法中调用，但是在View绘制流程之前
    onAttachedToWindow();

    ListenerInfo li = mListenerInfo;
    final CopyOnWriteArrayList<OnAttachStateChangeListener> listeners =
            li != null ? li.mOnAttachStateChangeListeners : null;
    if (listeners != null && listeners.size() > 0) {
        // NOTE: because of the use of CopyOnWriteArrayList, we *must* use an iterator to
        // perform the dispatching. The iterator is a safe guard against listeners that
        // could mutate the list by calling the various add/remove methods. This prevents
        // the array from being modified while we iterate it.
        for (OnAttachStateChangeListener listener : listeners) {
            // 通知所有监听View已经onAttachToWindow的客户端，即view.addOnAttachStateChangeListener();
            // 但此时View还没有开始绘制，不能正确获取测量大小或View实际大小
            listener.onViewAttachedToWindow(this);
        }
    }

    int vis = info.mWindowVisibility;
    if (vis != GONE) {
        onWindowVisibilityChanged(vis);
        if (isShown()) {
            // Calling onVisibilityAggregated directly here since the subtree will also
            // receive dispatchAttachedToWindow and this same call
            onVisibilityAggregated(vis == VISIBLE);
        }
    }

    // 回调View的onVisibilityChanged
    // 注意这时候View绘制流程还未真正开始
    onVisibilityChanged(this, visibility);

    if ((mPrivateFlags & PFLAG_DRAWABLE_STATE_DIRTY) != 0) {
        // If nobody has evaluated the drawable state yet, then do it now.
        refreshDrawableState();
    }
    needGlobalAttributesUpdate(false);

    notifyEnterOrExitForAutoFillIfNeeded(true);
}
```
在该方法中，首先将 `AttachInfo` 赋值给当前 `childView`，记得前面分析到的 `addXxxListener()`，此后所有添加的 `Listener` 都被保存在该 `AttachInfo` 的 `ViewTreeObserver` 内。

这里还要着重看下 `info.mTreeObserver.merge()`，前面有提到，`View` 在被关联 `AttachInfo` 之前，每个 `View` 拥有自己临时的 `ViewTreeObserver`，在绘制任务的开始阶段，会为每个 `View` 关联一个 `AttachInfo`，此时便将当前 `View` 的 `ViewTreeObserver` 合并到 `AttachInfo` 的 `ViewTreeObserver`。看下这一合并过程 `ViewTreeObserver` 的 `merge()` 方法：
```java
/**
 * 在 ViewRootImpl 的 dispatchAttachedToWindow() 方法被调用
 *
 * @param observer 子View的ViewTreeObserver
 */
void merge(ViewTreeObserver observer) {
    // 合并OnWindowAttachListener
    if (observer.mOnWindowAttachListeners != null) {
        if (mOnWindowAttachListeners != null) {
            // 添加到AttachInfo的ViewTreeObserver
            mOnWindowAttachListeners.addAll(observer.mOnWindowAttachListeners);
        } else {
            // 首次直接赋值
            mOnWindowAttachListeners = observer.mOnWindowAttachListeners;
        }
    }
    // 合并OnWindowFocusListener
    if (observer.mOnWindowFocusListeners != null) {
        if (mOnWindowFocusListeners != null) {
            mOnWindowFocusListeners.addAll(observer.mOnWindowFocusListeners);
        } else {
            mOnWindowFocusListeners = observer.mOnWindowFocusListeners;
        }
    }
    // 合并OnGlobalFocusListener
    if (observer.mOnGlobalFocusListeners != null) {
        if (mOnGlobalFocusListeners != null) {
            mOnGlobalFocusListeners.addAll(observer.mOnGlobalFocusListeners);
        } else {
            mOnGlobalFocusListeners = observer.mOnGlobalFocusListeners;
        }
    }

    // ...

    observer.kill();
}
```

可以看到，依次将 `View` 内部的 `ViewTreeObserver` 合并到 `AttachInfo` 的 `ViewTreeObserver`。

### OnWindowAttachListener ###
注意 `ViewRootImpl` 的 `performTraversals()` 方法，该方法将依次完成 `View` 的三大绘制流程，不过在此之前先回调 `AttachInfo` 内 `ViewTreeObserver` 的 `dispatchOnWindowAttachChange()` 方法：
```java
/**
 * 通知已注册的监听器窗口已被关联/分离
 */
final void dispatchOnWindowAttachedChange(boolean attached) {
    // 注意:由于使用了CopyOnWriteArrayList，必须使用迭代器来执行调度。
    // 迭代器是对侦听器的安全防护，可以通过调用各种添加/删除方法来改变列表。
    // 这样可以防止当我们迭代数组时，数组不会被修改。
    final CopyOnWriteArrayList<OnWindowAttachListener> listeners
            = mOnWindowAttachListeners;
    if (listeners != null && listeners.size() > 0) {
        for (OnWindowAttachListener listener : listeners) {
            if (attached) {
                // View 被附加到窗口
                listener.onWindowAttached();
            } else {
                // View 从窗口中分离
                listener.onWindowDetached();
            }
        }
    }
}
```
即 `onWindowAttached()` 方法的调用在 `Activity` 生命周期 `onResume()` 方法之后，在绘制流程之前。此时 `View` 的绘制流程还未开始，在该方法不能直接获取到 `View` 的实际宽高。

那 `onWindowDetached()` 方法是在什么时候被回调呢？又在 `Activity` 的哪个生命周期阶段呢？

它的调用时机是在 `ActivityThread` 的 `handleDestoryActivity()` 方法，在该方法首先回调 `Activity` 的 `onDestory()` 生命周期方法，然后将当前窗口从 `WindowManager` 移除，最终调用到 `ViewRootImpl` 的 `doDie()`，在该方法将完成 `dispatchDetachedFromWindow()` 通知：
```java
void dispatchDetachedFromWindow() {
    // 回调 OnWindowAttachListener 的 onWindowDetached 方法
    if (mView != null && mView.mAttachInfo != null) {
        mAttachInfo.mTreeObserver.dispatchOnWindowAttachedChange(false);
        mView.dispatchDetachedFromWindow();
    }
    ...
}
```

> `onWindowAttached()` 在 `Activity` 生命周期 `onResume()` 方法之后，但是在 `View` 绘制流程之前；`onWindowDetached()` 在 `Activity` 生命周期 `onDestory()` 方法之后，此时 `View` 已经从窗口中移除。

### OnWindowFocusChangeListener ###
`OnWindowFocusChangeListener` 表示当窗口焦点发生变化时回调，它在 `ViewTreeObserver` 内的分发过程如下：
```java
/**
 * 通知已注册的侦听器窗口焦点已更改
 */
final void dispatchOnWindowFocusChange(boolean hasFocus) {
    final CopyOnWriteArrayList<OnWindowFocusChangeListener> listeners
            = mOnWindowFocusListeners;
    if (listeners != null && listeners.size() > 0) {
        for (OnWindowFocusChangeListener listener : listeners) {
            // 回调OnWindowFocusChangeListener的onWindowFocusChanged()方法
            listener.onWindowFocusChanged(hasFocus);
        }
    }
}
```

那么，它是在哪里被回调的呢？

猜测肯定还是离不开 `ViewRootImpl`。查找发现在 `ViewRootImpl` 的静态内部类 `W` 中，简单看下这一过程：
```java
/**
 * mWindow是ViewRootImpl的静态内部类，内部持有当前ViewRootImpl
 */
static class W extends IWindow.Stub {
    // 弱引用持有当前ViewRootImpl
    // 因为它将被保存在WindowManagerService
    private final WeakReference<ViewRootImpl> mViewAncestor;
    private final IWindowSession mWindowSession;

    W(ViewRootImpl viewAncestor) {
        mViewAncestor = new WeakReference<ViewRootImpl>(viewAncestor);
        mWindowSession = viewAncestor.mWindowSession;
    }


    /**
     * 由远程 WMS 通知窗口焦点发生变化
     */
    @Override
    public void windowFocusChanged(boolean hasFocus, boolean inTouchMode) {
        final ViewRootImpl viewAncestor = mViewAncestor.get();
        if (viewAncestor != null) {
            viewAncestor.windowFocusChanged(hasFocus, inTouchMode);
        }
    }

    ...

}
```
`W` 本质是一个 `Binder` 对象，内部通过弱引用持有当前的 `ViewRootImpl`，它将作为客户端保存在 `WMS`(`WindowManagerService`)，用来和 `WindowSession` 交互，实际间接在与 `WMS` 交互。

`windowFocusChanged()` 方法最终会发送消息到当前绘制线程，关键代码如下：
```java
case MSG_WINDOW_FOCUS_CHANGED: {
    if(mView != null){
        // 通知 View 窗口焦点发生变化
        mView.dispatchWindowFocusChanged(hasWindowFocus);
        // 通知 ViewTreeObserver 窗口焦点发生变化
        mAttachInfo.mTreeObserver.dispatchOnWindowFocusChange(hasWindowFocus);
    }
}
```

### OnGlobalFocusChangeListener ###
`OnGlobalFocusChangeListener` 表示当视图树中的焦点状态更改时回调，它在 `ViewTreeObserver` 内的分发过程如下：
```java
/**
* 通知已注册的侦听器焦点已更改
 */
final void dispatchOnGlobalFocusChange(View oldFocus, View newFocus) {
    final CopyOnWriteArrayList<OnGlobalFocusChangeListener> listeners = mOnGlobalFocusListeners;
    if (listeners != null && listeners.size() > 0) {
        for (OnGlobalFocusChangeListener listener : listeners) {
            // 回调OnGlobalFocusChangeListener的onGlobalFocusChanged()方法
            listener.onGlobalFocusChanged(oldFocus, newFocus);
        }
    }
}
```

`onGlobalFocusChanged()` 方法的参数如下：
 - `oldFocus：`之前获取到焦点的视图
 - `newFocus：`当前获取到焦点的视图

> 注意：`oldFocus` 和 `newFocus` 都有可能为 `null`。

如果要监听某个 `View` 是否获取到焦点，我们可以使用如下方式：
```java
    mView.setOnFocusChangeListener(new View.OnFocusChangeListener() {
        @Override
        public void onFocusChange(View v, boolean hasFocus) {
            // 监听当前 View 是否获取到焦点
        }
    });
```

不过，当要监听的 `View` 数量较多时，这种方式好像就不太友好了，此时我们便可以使用如下方式，监听整个视图树内 `View` 焦点的变化：
```java
    mView.getViewTreeObserver().addOnGlobalFocusChangeListener(new ViewTreeObserver.OnGlobalFocusChangeListener() {
        @Override
        public void onGlobalFocusChanged(View oldFocus, View newFocus) {
            // 监听整个视图树内的View焦点变化
        }
    });
```

### OnGlobalLayoutListener ###
`OnGlobalLayoutListener` 表示视图树中的的布局状态或视图的可见性更改时回调，它在 `ViewTreeObserver` 内的分发过程如下：
```java
/**
 * 通知已注册的侦听器发生了布局状态或视图的可见性改变。
 * 如果正在强制视图或视图层次结构上的布局不附加到窗口或处于消失状态，则可以手动调用此方法。
 */
public final void dispatchOnGlobalLayout() {
    final CopyOnWriteArray<OnGlobalLayoutListener> listeners = mOnGlobalLayoutListeners;
    if (listeners != null && listeners.size() > 0) {
        // 调用了 start，此时如果有其他线程要 add 内容就会执行copyOnWrite
        CopyOnWriteArray.Access<OnGlobalLayoutListener> access = listeners.start();
        try {
            int count = access.size();
            for (int i = 0; i < count; i++) {
                // 回调OnGlobalLayoutListener的onGlobalLayout()方法
                access.get(i).onGlobalLayout();
            }
        } finally {
            listeners.end();
        }
    }
}
```

注意保存 `OnGlobalLayoutListener` 的容器是 `CopyOnWriteArray`，而非 `CopyOnWriteArrayList`，前者是 `ViewTreeObserver` 的静态内部类，后者是 `Java` 内置的并发工具类，两者有什么区别吗？`CopyOnWriteArray` 在没有遍历任务时(未调用 `start()` 方法)，此时执行 `add` 操作直接添加到原容器，否则才会执行 `CopyOnWrite`，这有助于减少内存抖动场景的发生。

如果仅从名字来看，它应该和 `View` 的 `layout` 阶段有关，下面就再看下 `View` 的绘制流程：
```java
private void performTraversals() {
    // ...

    // View的测量阶段
    performMeasure();
    // View 的布局阶段
    performLayout();

    // ...
    // 关键方法在这里
    if (triggerGlobalLayoutListener) {
        mAttachInfo.mRecomputeGlobalAttributes = false;
        // 布局阶段完成之后回调 OnGlobalLayoutListener，
        // 在该方法中可以正确获取 View 的实际宽高
        mAttachInfo.mTreeObserver.dispatchOnGlobalLayout();
    }

    // ...

    // View 的绘制阶段
    performDraw();

}
```

`dispatchOnGlobalLayout()` 的回调是在 `performLayout()` 方法之后，此时视图树的布局阶段已经完成，在该方法中可以正确获取到 `View` 的实际宽高：
```java
    view.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
        @Override
        public void onGlobalLayout() {
            // 正确获取到View的宽高
            int width = view.getWidth();
            int height = view.getHeight();
        }
    });
```

> 注意：此时 `View` 的绘制任务还未开始！

### OnPreDrawListener ###
`OnPreDrawListener` 表示当视图即将绘制时回调，它在 `ViewTreeObserver` 内的分发过程如下：
```java
/**
 * 通知已注册的侦听器视图即将开始绘制。
 * 如果要强制在未附加到Window或处于GONE状态的View或View的层次结构上进行绘图，则可以手动调用此方法。
 */
public final boolean dispatchOnPreDraw() {
    boolean cancelDraw = false;
    final CopyOnWriteArray<OnPreDrawListener> listeners = mOnPreDrawListeners;
    if (listeners != null && listeners.size() > 0) {
        // 调用了 start，此时如果有其他线程要 add 内容就会执行copyOnWrite
        CopyOnWriteArray.Access<OnPreDrawListener> access = listeners.start();
        try {
            int count = access.size();
            for (int i = 0; i < count; i++) {
                // 回调OnPreDrawListener的onPreDraw()方法
                cancelDraw |= !(access.get(i).onPreDraw());
            }
        } finally {
            listeners.end();
        }
    }
    return cancelDraw;
}
```

`OnPreDrawListener` 和 `OnDrawListener` 两者有什么区别呢？`Pre` 表达的应该是 `Previous`，即在 `Draw` 之前，直接看下 `View` 的绘制任务：
```java
private void performTraversals() {
    // ...

    // 调用 AttachInfo 内 ViewTreeObserver 的 dispatchOnPreDraw方法
    boolean cancelDraw = mAttachInfo.mTreeObserver.dispatchOnPreDraw() || !isViewVisible;

    // 只要有一个 Listener 返回true，将重新发起绘制流程
    if (!cancelDraw) {
        if (mPendingTransitions != null && mPendingTransitions.size() > 0) {
            for (int i = 0; i < mPendingTransitions.size(); ++i) {
                mPendingTransitions.get(i).startChangingAnimations();
            }
            mPendingTransitions.clear();
        }

        //执行绘制操作
        performDraw();
    } else {
        if (isViewVisible) {
            // 重新发起绘制流程，measure、layout
            scheduleTraversals();
        } else if (mPendingTransitions != null && mPendingTransitions.size() > 0) {
            for (int i = 0; i < mPendingTransitions.size(); ++i) {
                mPendingTransitions.get(i).endChangingAnimations();
            }
            mPendingTransitions.clear();
        }
    }
}
```

如果 `dispatchOnPreDraw()` 方法返回 `true`，此时将拦截 `performDraw()` 的绘制任务，重新发起绘制流程 `scheduleTraversals()`。其实它的主要作用是在绘制任务之前给一次拦截机会，例如有新的 `View` 需要添加到当前视图树中，此时可以拦截该次绘制任务，重新发起 `measure`、`layout` 流程。

### OnDrawListener ###
`OnDrawListener` 与 `OnPreDrawListener` 类似，后者可以拦截绘制任务，重新发起绘制流程，它在 `ViewTreeObserver` 内的分发过程如下：
/**
 * 通知已注册的侦听器绘图即将开始
 */
public final void dispatchOnDraw() {
    if (mOnDrawListeners != null) {
        mInDispatchOnDraw = true;
        final ArrayList<OnDrawListener> listeners = mOnDrawListeners;
        int numListeners = listeners.size();
        for (int i = 0; i < numListeners; ++i) {
            // 回调OnDrawListener的onDraw()方法，通知View开始绘制
            listeners.get(i).onDraw();
        }
        mInDispatchOnDraw = false;
    }
}

`onDraw` 肯定和 `View` 的绘制任务有关！还是要从 `ViewRootImpl` 入手，`View` 的绘制任务 `performDraw()` 方法最终会调用到 `draw()` 方法：
```java
private boolean draw(boolean fullRedrawNeeded) {
    // 窗口都关联有一个 Surface
    // 在 Android 中，所有的元素都在 Surface 这张画纸上进行绘制和渲染，
    // 普通 View（例如非 SurfaceView 或 TextureView） 是没有 Surface 的，
    // 一般 Activity 包含多个 View 形成 View Hierachy 的树形结构，只有最顶层的 DecorView 才是对 WindowManagerService “可见的”。
    // 而为普通 View 提供 Surface 的正是 ViewRootImpl。
    Surface surface = mSurface;
    // 判断Surface 是否有效
    if (!surface.isValid()) {
        return false;
    }

    // 跟踪FPS
    if (DEBUG_FPS) {
        trackFPS();
    }

    // ...

    scrollToRectOrFocus(null, false);

    if (mAttachInfo.mViewScrollChanged) {
        mAttachInfo.mViewScrollChanged = false;
        // 注意 OnScrollChangedListener 在这里回调
        mAttachInfo.mTreeObserver.dispatchOnScrollChanged();
    }

    // ...

    // 回调 ViewTreeObserver内的dispatchOnDraw()方法
    mAttachInfo.mTreeObserver.dispatchOnDraw();

    // ...

    boolean useAsyncReport = false;
    if (!dirty.isEmpty() || mIsAnimating || accessibilityFocusDirty) {
        if (mAttachInfo.mThreadedRenderer != null && mAttachInfo.mThreadedRenderer.isEnabled()) {
            // 是否开启了硬件加速
            boolean invalidateRoot = accessibilityFocusDirty || mInvalidateRootRequested;
            mInvalidateRootRequested = false;

            // ...

            useAsyncReport = true;

            // 开启硬件加速绘制执行这里，最终还是执行View的draw()开始
            mAttachInfo.mThreadedRenderer.draw(mView, mAttachInfo, this);
        } else {
            // ...

            // 最终调用到drawSoftware
            // surface，每个 View 都由某一个窗口管理，而每一个窗口都关联有一个 Surface
            // mDirty.set(0, 0, mWidth, mHeight); dirty 表示画纸尺寸，对于DecorView，left = 0,
            if (!drawSoftware(surface, mAttachInfo, xOffset, yOffset,
                    scalingRequired, dirty, surfaceInsets)) {
                return false;
            }
        }
    }

    // ...

    return useAsyncReport;
}
```

需要注意，此时 `View` 绘制流程的前两个阶段：`measure` 和 `layout` 都已经完成。通过该 `Listener` 可以监听开始执行绘制任务。

### OnTouchModeChangeListener ###
`OnTouchModeChangeListener` 表示当触摸模式改变时回调，它在 `ViewTreeObserver` 内的分发过程如下：
```java
/**
 * 通知已注册的侦听器触摸模式已更改
 */
final void dispatchOnTouchModeChanged(boolean inTouchMode) {
    final CopyOnWriteArrayList<OnTouchModeChangeListener> listeners =
            mOnTouchModeChangeListeners;
    if (listeners != null && listeners.size() > 0) {
        for (OnTouchModeChangeListener listener : listeners) {
            // 回调OnTouchModeChangeListener的onTouchModeChanged()方法
            listener.onTouchModeChanged(inTouchMode);
        }
    }
}
```

前面分析 `OnWindowFocusChangeListener` 的回调是通过 `ViewRootImpl` 静态内部类 `W` 完成的，在该类中 `windowFocusChanged()` 方法接收远程 `WMS` 的回调通知，发送消息到当前绘制线程：
```java
case MSG_WINDOW_FOCUS_CHANGED: {
    if(hasWindowFocus){
        boolean inTouchMode = msg.arg2 != 0
        // 这里将回调 OnTouchModeChangeListener
        ensureTouchModeLocally(inTouchMode)
    }
    // ... 下面回调 dispatchWindowFocusChanged() 方法
}
```

`ensureTouchModeLocally()` 方法将会完成该过程：
```java
private boolean ensureTouchModeLocally(boolean inTouchMode) {
    if (mAttachInfo.mInTouchMode == inTouchMode) return false;

    mAttachInfo.mInTouchMode = inTouchMode;
    // 回调AttachInfo内ViewTreeObserver的dispatchOnTouchModeChanged()方法
    mAttachInfo.mTreeObserver.dispatchOnTouchModeChanged(inTouchMode);

    return (inTouchMode) ? enterTouchMode() : leaveTouchMode();
}
```

### OnScrollChangedListener ###
`OnScrollChangeListener` 表示当视图树中的某些内容被滚动时回调，它在 `ViewTreeObserver` 内的分发过程如下：
```java
/**
 * 通知已注册的侦听器某些内容发生滚动滚动
 */
final void dispatchOnScrollChanged() {
    final CopyOnWriteArray<OnScrollChangedListener> listeners = mOnScrollChangedListeners;
    if (listeners != null && listeners.size() > 0) {
        CopyOnWriteArray.Access<OnScrollChangedListener> access = listeners.start();
        try {
            int count = access.size();
            for (int i = 0; i < count; i++) {
                // 回调OnScrollChangedListener的onScrollChanged()方法
                access.get(i).onScrollChanged();
            }
        } finally {
            listeners.end();
        }
    }
}
```

是否有注意到前面分析 `OnDrawListener` 时的绘制方法 `draw()`，在调用 `ViewTreeObserver` 的 `dispatchOnDraw()` 方法之前，会根据情况率先执行 `ViewTreeObserver` 的 `dispachOnScrollChanged()` 方法。

`OnScrollChangedListener` 回调在 `OnDrawListener` 之前，用于通知外部当前视图树中有 `View` 发生滚动。

## 公共方法 ##

| Method                              | 作用                                                                                       |
| :---------------------------------- | :----------------------------------------------------------------------------------------- |
| addOnWindowAttachListener()         | 注册视图层次结构关联到窗口或与之分离的监听                                                 |
| removeOnWindowAttachListener()      | 移除视图层次结构关联到窗口或与之分离的监听                                                 |
| addOnWindowFocusChangeListener()    | 注册视图层次结构的窗口焦点状态发生改变的监听                                               |
| removeOnWindowFocusChangeListener() | 移除视图层次结构的窗口焦点状态发生改变的监听                                               |
| addOnGlobalFocusChangeListener()    | 注册视图树中的焦点状态发生改变的监听                                                       |
| removeOnGlobalFocusChangeListener() | 移除视图树中的焦点状态发生改变的监听                                                       |
| addOnGlobalLayoutListener()         | 注册视图树全局布局状态发生改变或视图树中某个视图的可视状态发生改变的监听                   |
| removeOnGlobalLayoutListener()      | 移除视图树全局布局状态发生改变或视图树中某个视图的可视状态发生改变的监听                   |
| addOnPreDrawListener()              | 注册视图树即将开始绘制的监听                                                               |
| removeOnPreDrawListener()           | 移除视图树即将开始绘制的监听                                                               |
| addOnDrawListener()                 | 注册视图树即将开始绘制的监听                                                               |
| removeOnDrawListener()              | 移除视图树即将开始绘制的监听                                                               |
| addOnScrollChangedListener()        | 注册视图树中的一些组件发生滚动的监听                                                       |
| removeOnScrollChangedListener()     | 移除视图树中的一些组件发生滚动的监听                                                       |
| addOnTouchModeChangeListener()      | 注册视图树的触摸模式发生改变的监听                                                         |
| removeOnTouchModeChangeListener()   | 移除视图树的触摸模式发生改变的监听                                                         |
| addOnWindowShownListener()          | 注册视图树显示的监听                                                                       |
| removeOnWindowShownListener()       | 移除视图树显示的监听                                                                       |
| isAlive()                           | 判断当前的 ViewTreeObserver 是否可用，当不可用时，对方法的任何调用(除此方法外)都将引发异常 |
| dispatchOnGlobalLayout()            | 通知已注册的侦听器发生了布局状态或视图的可见性改变                                         |
| dispatchOnPreDraw()                 | 通知已注册的侦听器视图即将开始绘制                                                         |
| dispatchOnDraw()                    | 通知已注册的侦听器视图即将开始绘制                                                         |



