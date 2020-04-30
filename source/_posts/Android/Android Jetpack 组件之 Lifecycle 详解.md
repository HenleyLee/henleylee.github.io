---
title: Android Jetpack 组件之 Lifecycle 详解
categories: Android
tags:
  - Android
  - Jetpack
abbrlink: '322089e5'
date: 2019-11-07 18:36:32
---

`Lifecycle` 组件是 Android Jetpack 的一部分，该组件可以感知 `Activity` 和 `Fragment` 的生命周期状态的改变，有利于生成更易组织、更轻量化、更易于维护的代码，常用的开发方式就是在组件的对应的生命周期方法中处理相关业务逻辑，这种方式会导致不良代码的产生以及 bug 的增加，如果使用生命周期组件则可将依赖组件的代码移除生命周期并移入组件本身。

## 简介 ##
`Lifecycle` 是一个持有组件生命周期状态信息的抽象类，其直接子类是 `LifecycleRegistry`，`Activity` 和 `Fragment` 都间接实现了 `LifecycleOwner` 接口，且提供了对应的 `getLifecycle()` 方法获取 `Lifecycle` 对象，获取到的也就是 `LifecycleRegistry` 对象。

`Lifecycle` 使用两个主要枚举来跟踪其关联组件的生命周期状态：
 - **`Event：`**`Lifecycle` 的生命周期事件对应的 `Activity` 和 `Fragment` 的生命周期方法。
 - **`State：`**组件在 `Lifecycle` 中生命周期所处的状态。

下面是 `Lifecycle` 的生命周期对应状态图：
![Lifecycle的生命周期对应状态图](https://henleylee.github.io/medias/android/lifecycle-states.png)

## 使用 ##
### 添加依赖 ###
要将 `Lifecycle` 库引入到 Android 项目中，请将以下依赖项添加到应用或模块的 `build.gradle` 文件中：
```gradle
dependencies {
    def lifecycle_version = "2.2.0"

    // ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel:$lifecycle_version"
    // LiveData
    implementation "androidx.lifecycle:lifecycle-livedata:$lifecycle_version"
    // Lifecycles only (without ViewModel or LiveData)
    implementation "androidx.lifecycle:lifecycle-runtime:$lifecycle_version"

    // Saved state module for ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel-savedstate:$lifecycle_version"

    // Annotation processor
    annotationProcessor "androidx.lifecycle:lifecycle-compiler:$lifecycle_version"
    // alternately - if using Java8, use the following instead of lifecycle-compiler
    implementation "androidx.lifecycle:lifecycle-common-java8:$lifecycle_version"

    // optional - ReactiveStreams support for LiveData
    implementation "androidx.lifecycle:lifecycle-reactivestreams:$lifecycle_version"

    // optional - Test helpers for LiveData
    testImplementation "androidx.arch.core:core-testing:$lifecycle_version"
}
```

### JDK 1.7 ###
如果使用的是 JDK1.7，可以实现 `LifecycleObserver` 接口并使用注解的方式来监听生命周期变化。

首先在对应 module 的 `build.gradle` 文件文件中引入：
```gradle
annotationProcessor "androidx.lifecycle:lifecycle-compiler:$lifecycle_version"
```

然后，创建生命周期监听器，示例代码如下：
```java
public class SimpleLifecycleObserver implements LifecycleObserver {

    private static final String TAG = "LifecycleObserver";

    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    public void onCreate(@NonNull LifecycleOwner owner) {
        Log.i(TAG, "--onCreate-->");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    public void onStart(@NonNull LifecycleOwner owner) {
        Log.i(TAG, "--onStart-->");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void onResume(@NonNull LifecycleOwner owner) {
        Log.i(TAG, "--onResume-->");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void onPause(@NonNull LifecycleOwner owner) {
        Log.i(TAG, "--onPause-->");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    public void onStop(@NonNull LifecycleOwner owner) {
        Log.i(TAG, "--onStop-->");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    public void onDestroy(@NonNull LifecycleOwner owner) {
        Log.i(TAG, "--onDestroy-->");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_ANY)
    public void onAny(LifecycleOwner owner, Lifecycle.Event event) {
        Log.i(TAG, "--onAny-->" + event.name());
    }

}
```

### JDK 1.8 ###
如果使用的是 JDK1.8，可以实现 `DefaultLifecycleObserver` 接口的方式来监听生命周期变化，示例如下：

首先在对应 module 的 `build.gradle` 文件文件中引入：
```gradle
implementation "androidx.lifecycle:lifecycle-common-java8:$lifecycle_version"
```

然后，创建生命周期监听器，示例代码如下：
```java
public class SimpleLifecycleObserver implements DefaultLifecycleObserver {

    private static final String TAG = "LifecycleObserver";

    @Override
    public void onCreate(@NonNull LifecycleOwner owner) {
        Log.i(TAG, "--onCreate-->");
    }

    @Override
    public void onStart(@NonNull LifecycleOwner owner) {
        Log.i(TAG, "--onStart-->");
    }

    @Override
    public void onResume(@NonNull LifecycleOwner owner) {
        Log.i(TAG, "--onResume-->");
    }

    @Override
    public void onPause(@NonNull LifecycleOwner owner) {
        Log.i(TAG, "--onPause-->");
    }

    @Override
    public void onStop(@NonNull LifecycleOwner owner) {
        Log.i(TAG, "--onStop-->");
    }

    @Override
    public void onDestroy(@NonNull LifecycleOwner owner) {
        Log.i(TAG, "--onDestroy-->");
    }

}
```

## 源码分析 ##
### Lifecycle ###
`Lifecycle` 是一个具有 Android 生命周期的对象，`Fragment` 和 `ComponentActivity` 类实现了 `LifecycleOwner` 接口，该接口 `getLifecycle()` 方法返回 `Lifecycle` 对象。`Lifecycle` 的源码如下：
```java
public abstract class Lifecycle {

    /**
     * 添加一个LifecycleObserver，当LifecycleOwner更改状态时，将通知该观察者
     */
    @MainThread
    public abstract void addObserver(@NonNull LifecycleObserver observer);

    /**
     * 从观察者列表中移除LifecycleObserver
     */
    @MainThread
    public abstract void removeObserver(@NonNull LifecycleObserver observer);

    /**
     * 返回Lifecycle的当前状态
     */
    @MainThread
    @NonNull
    public abstract State getCurrentState();

    /** 生命周期事件 */
    public enum Event {
        ON_CREATE,
        ON_START,
        ON_RESUME,
        ON_PAUSE,
        ON_STOP,
        ON_DESTROY,
        ON_ANY
    }

    /** 生命周期状态 */
    public enum State {
        DESTROYED,
        INITIALIZED,
        CREATED,
        STARTED,
        RESUMED;

        /**
         * 比较此状态是否大于或等于给定的state
         */
        public boolean isAtLeast(@NonNull State state) {
            return compareTo(state) >= 0;
        }
    }

}
```

### LifecycleOwner ###
**`LifecycleOwner`** 是一个接口，表示实现该接口的类具有生命周期，如所有继承自 `AppCompatActivity` 的 `Activity` 都间接实现了 `LifecycleOwner` 接口，源码中是 `ComponentActivity` 实现了该接口，实际开发中可实现 `LifecycleOwner` 接口将原先 `Activity` 或 `Fragment` 生命周期中执行的业务移到组件本身。`LifecycleOwner` 的源码如下：
```java
public interface LifecycleOwner {
    /**
     * 返回提供者的Lifecycle对象
     */
    @NonNull
    Lifecycle getLifecycle();
}
```

`getLifecycle()` 方法实际返回的是 `Lifecycle` 的实现类 `LifecycleRegistry`，然后通过使用 `addObserver()` 添加 `LifecycleObserver` 对象，也就是添加要通知的观察者，之后才能够在 `Activity` 或 `Fragment` 生命周期发生变化时通知观察者。

### ReportFragment ###
在 `ComponentActivity` 中添加了一个无界面的 `Fragment` 用来感知 `Activity` 的生命周期，`ComponentActivity` 的源码如下：
```java
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mSavedStateRegistryController.performRestore(savedInstanceState);
    // 在Activity中添加一个ReportFragment用于感知Activity的生命周期
    ReportFragment.injectIfNeededIn(this);
}
```

`injectIfNeededIn()` 方法会创建一个 `ReportFragment`，当每创建一个 `Activity` 的时候，该 `Fragment` 就会被添加到对应的 `Activity` 中。`ReportFragment` 源码如下：
```java
public class ReportFragment extends Fragment {

    private static final String REPORT_FRAGMENT_TAG = "androidx.lifecycle"
            + ".LifecycleDispatcher.report_fragment_tag";

    /**
     * 创建ReportFragment，并添加到Activity中
     */
    public static void injectIfNeededIn(Activity activity) {
        if (Build.VERSION.SDK_INT >= 29) {
            // API 29+，可以直接使用ActivityLifecycleCallbacks处理生命周期回调
            activity.registerActivityLifecycleCallbacks(new LifecycleCallbacks());
        }
        // 避免重复添加
        android.app.FragmentManager manager = activity.getFragmentManager();
        if (manager.findFragmentByTag(REPORT_FRAGMENT_TAG) == null) {
            manager.beginTransaction().add(new ReportFragment(), REPORT_FRAGMENT_TAG).commit();
            // Hopefully, we are the first to make a transaction.
            manager.executePendingTransactions();
        }
    }

    // Activity 初始化监听器，监听Activity的生命周期状态
    private ActivityInitializationListener mProcessListener;

    //<editor-fold desc="Activity生命周期状态分发">
    private void dispatchCreate(ActivityInitializationListener listener) {
        if (listener != null) {
            listener.onCreate();
        }
    }

    private void dispatchStart(ActivityInitializationListener listener) {
        if (listener != null) {
            listener.onStart();
        }
    }

    private void dispatchResume(ActivityInitializationListener listener) {
        if (listener != null) {
            listener.onResume();
        }
    }

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        dispatchCreate(mProcessListener);
        dispatch(Lifecycle.Event.ON_CREATE);
    }

    @Override
    public void onStart() {
        super.onStart();
        dispatchStart(mProcessListener);
        dispatch(Lifecycle.Event.ON_START);
    }

    @Override
    public void onResume() {
        super.onResume();
        dispatchResume(mProcessListener);
        dispatch(Lifecycle.Event.ON_RESUME);
    }

    @Override
    public void onPause() {
        super.onPause();
        dispatch(Lifecycle.Event.ON_PAUSE);
    }

    @Override
    public void onStop() {
        super.onStop();
        dispatch(Lifecycle.Event.ON_STOP);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        dispatch(Lifecycle.Event.ON_DESTROY);
        // just want to be sure that we won't leak reference to an activity
        mProcessListener = null;
    }

    /**
     * 分发生命周期状态
     */
    private void dispatch(@NonNull Lifecycle.Event event) {
        // API 29以下，使用dispatch()方法分发生命周期状态
        if (Build.VERSION.SDK_INT < 29) {
            dispatch(getActivity(), event);
        }
    }

    @SuppressWarnings("deprecation")
    static void dispatch(@NonNull Activity activity, @NonNull Lifecycle.Event event) {
        // LifecycleRegistryOwner已被废弃，留着应该是为了兼容
        if (activity instanceof LifecycleRegistryOwner) {
            ((LifecycleRegistryOwner) activity).getLifecycle().handleLifecycleEvent(event);
            return;
        }
        // 获取Activity中的Lifecycle
        if (activity instanceof LifecycleOwner) {
            Lifecycle lifecycle = ((LifecycleOwner) activity).getLifecycle();
            if (lifecycle instanceof LifecycleRegistry) {
                // 分发Activity生命周期状态
                ((LifecycleRegistry) lifecycle).handleLifecycleEvent(event);
            }
        }
    }

    //</editor-fold>

    void setProcessListener(ActivityInitializationListener processListener) {
        mProcessListener = processListener;
    }

    interface ActivityInitializationListener {
        void onCreate();

        void onStart();

        void onResume();
    }

    // this class isn't inlined only because we need to add a proguard rule for it. (b/142778206)
    static class LifecycleCallbacks implements Application.ActivityLifecycleCallbacks {
        @Override
        public void onActivityCreated(@NonNull Activity activity, @Nullable Bundle bundle) {
        }

        @Override
        public void onActivityPostCreated(@NonNull Activity activity, @Nullable Bundle savedInstanceState) {
            dispatch(activity, Lifecycle.Event.ON_CREATE);
        }

        @Override
        public void onActivityStarted(@NonNull Activity activity) {
        }

        @Override
        public void onActivityPostStarted(@NonNull Activity activity) {
            dispatch(activity, Lifecycle.Event.ON_START);
        }

        @Override
        public void onActivityResumed(@NonNull Activity activity) {
        }

        @Override
        public void onActivityPostResumed(@NonNull Activity activity) {
            dispatch(activity, Lifecycle.Event.ON_RESUME);
        }

        @Override
        public void onActivityPrePaused(@NonNull Activity activity) {
            dispatch(activity, Lifecycle.Event.ON_PAUSE);
        }

        @Override
        public void onActivityPaused(@NonNull Activity activity) {
        }

        @Override
        public void onActivityPreStopped(@NonNull Activity activity) {
            dispatch(activity, Lifecycle.Event.ON_STOP);
        }

        @Override
        public void onActivityStopped(@NonNull Activity activity) {
        }

        @Override
        public void onActivitySaveInstanceState(@NonNull Activity activity, @NonNull Bundle bundle) {
        }

        @Override
        public void onActivityPreDestroyed(@NonNull Activity activity) {
            dispatch(activity, Lifecycle.Event.ON_DESTROY);
        }

        @Override
        public void onActivityDestroyed(@NonNull Activity activity) {
        }
    }
}
```

分析源码可以发现，在 `ReportFragment` 对应的生命周期方法里面进行对应生命周期状态的分发，分发过程对应 `dispatch()` 方法，里面调用了 `LifecycleRegistry` 的 `handleLifecycleEvent()` 方法。

### ProcessLifecycleOwner ###
在 API 29 之前对 `Activity` 对应的生命周期监听是在 `ActivityInitializationListener` 中完成的，该接口的实现类是一个位于 `ProcessLifecycleOwner` 中的匿名内部类，`ProcessLifecycleOwner` 的源码如下：
```java
public class ProcessLifecycleOwner implements LifecycleOwner {

    // ...

    private Handler mHandler;
    private final LifecycleRegistry mRegistry = new LifecycleRegistry(this);

    private Runnable mDelayedPauseRunnable = new Runnable() {
        @Override
        public void run() {
            dispatchPauseIfNeeded();
            dispatchStopIfNeeded();
        }
    };

    /** ActivityInitializationListener匿名实现类 */
    ActivityInitializationListener mInitializationListener = new ActivityInitializationListener() {
                @Override
                public void onCreate() {
                }

                @Override
                public void onStart() {
                    activityStarted();
                }

                @Override
                public void onResume() {
                    activityResumed();
                }
            };

    private static final ProcessLifecycleOwner sInstance = new ProcessLifecycleOwner();

    @NonNull
    public static LifecycleOwner get() {
        return sInstance;
    }

    /** 初始化 */
    static void init(Context context) {
        sInstance.attach(context);
    }

    /** ON_START事件回调LifecycleRegistry的handleLifecycleEvent方法 */
    void activityStarted() {
        mStartedCounter++;
        if (mStartedCounter == 1 && mStopSent) {
            mRegistry.handleLifecycleEvent(Lifecycle.Event.ON_START);
            mStopSent = false;
        }
    }

    /** ON_RESUME事件回调LifecycleRegistry的handleLifecycleEvent方法 */
    void activityResumed() {
        mResumedCounter++;
        if (mResumedCounter == 1) {
            if (mPauseSent) {
                mRegistry.handleLifecycleEvent(Lifecycle.Event.ON_RESUME);
                mPauseSent = false;
            } else {
                mHandler.removeCallbacks(mDelayedPauseRunnable);
            }
        }
    }

    void activityPaused() {
        mResumedCounter--;
        if (mResumedCounter == 0) {
            mHandler.postDelayed(mDelayedPauseRunnable, TIMEOUT_MS);
        }
    }

    void activityStopped() {
        mStartedCounter--;
        dispatchStopIfNeeded();
    }

    void dispatchPauseIfNeeded() {
        if (mResumedCounter == 0) {
            mPauseSent = true;
            mRegistry.handleLifecycleEvent(Lifecycle.Event.ON_PAUSE);
        }
    }

    void dispatchStopIfNeeded() {
        if (mStartedCounter == 0 && mPauseSent) {
            mRegistry.handleLifecycleEvent(Lifecycle.Event.ON_STOP);
            mStopSent = true;
        }
    }

    private ProcessLifecycleOwner() {
    }

    void attach(Context context) {
        mHandler = new Handler();
        mRegistry.handleLifecycleEvent(Lifecycle.Event.ON_CREATE);
        Application app = (Application) context.getApplicationContext();
        // 在没有Lifecycle组件之前，使用ActivityLifecycleCallbacks监听所有Activity的生命周期
        app.registerActivityLifecycleCallbacks(new EmptyActivityLifecycleCallbacks() {
            @Override
            public void onActivityPreCreated(@NonNull Activity activity,
                                             @Nullable Bundle savedInstanceState) {
                // We need the ProcessLifecycleOwner to get ON_START and ON_RESUME precisely
                // before the first activity gets its LifecycleOwner started/resumed.
                // The activity's LifecycleOwner gets started/resumed via an activity registered
                // callback added in onCreate(). By adding our own activity registered callback in
                // onActivityPreCreated(), we get our callbacks first while still having the
                // right relative order compared to the Activity's onStart()/onResume() callbacks.
                activity.registerActivityLifecycleCallbacks(new EmptyActivityLifecycleCallbacks() {
                    @Override
                    public void onActivityPostStarted(@NonNull Activity activity) {
                        activityStarted();
                    }

                    @Override
                    public void onActivityPostResumed(@NonNull Activity activity) {
                        activityResumed();
                    }
                });
            }

            @Override
            public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
                // Only use ReportFragment pre API 29 - after that, we can use the
                // onActivityPostStarted and onActivityPostResumed callbacks registered in
                // onActivityPreCreated()
                if (Build.VERSION.SDK_INT < 29) {
                    ReportFragment.get(activity).setProcessListener(mInitializationListener);
                }
            }

            @Override
            public void onActivityPaused(Activity activity) {
                activityPaused();
            }

            @Override
            public void onActivityStopped(Activity activity) {
                activityStopped();
            }
        });
    }

    @NonNull
    @Override
    public Lifecycle getLifecycle() {
        return mRegistry;
    }
}
```

### LifecycleRegistry ###
通过上面的源码分析可以发现，最终的生命周期状态分发在 `LifecycleRegistry` 的 `handleLifecycleEvent()` 方法中，最终调用了 `sync()` 方法，`LifecycleRegistry` 的源码如下：
```java
public class LifecycleRegistry extends Lifecycle {

    // ...

    /**
     * 将生命周期移至给定状态，并将必要的事件分派给观察者
     */
    @MainThread
    public void setCurrentState(@NonNull State state) {
        moveToState(state);
    }

    /**
     * 设置当前状态并通知观察者
     */
    public void handleLifecycleEvent(@NonNull Lifecycle.Event event) {
        State next = getStateAfter(event);
        moveToState(next);
    }

    private void moveToState(State next) {
        // 如果next与上次对此方法的调用处于相同状态，则直接返回
        if (mState == next) {
            return;
        }
        mState = next;
        if (mHandlingEvent || mAddingObserverCounter != 0) {
            mNewEventOccurred = true;
            // we will figure out what to do on upper level.
            return;
        }
        mHandlingEvent = true;
        sync();
        mHandlingEvent = false;
    }

    private boolean isSynced() {
        if (mObserverMap.size() == 0) {
            return true;
        }
        State eldestObserverState = mObserverMap.eldest().getValue().mState;
        State newestObserverState = mObserverMap.newest().getValue().mState;
        return eldestObserverState == newestObserverState && mState == newestObserverState;
    }

    static State getStateAfter(Event event) {
        switch (event) {
            case ON_CREATE:
            case ON_STOP:
                return CREATED;
            case ON_START:
            case ON_PAUSE:
                return STARTED;
            case ON_RESUME:
                return RESUMED;
            case ON_DESTROY:
                return DESTROYED;
            case ON_ANY:
                break;
        }
        throw new IllegalArgumentException("Unexpected event value " + event);
    }

    private static Event downEvent(State state) {
        switch (state) {
            case INITIALIZED:
                throw new IllegalArgumentException();
            case CREATED:
                return ON_DESTROY;
            case STARTED:
                return ON_STOP;
            case RESUMED:
                return ON_PAUSE;
            case DESTROYED:
                throw new IllegalArgumentException();
        }
        throw new IllegalArgumentException("Unexpected state value " + state);
    }

    private static Event upEvent(State state) {
        switch (state) {
            case INITIALIZED:
            case DESTROYED:
                return ON_CREATE;
            case CREATED:
                return ON_START;
            case STARTED:
                return ON_RESUME;
            case RESUMED:
                throw new IllegalArgumentException();
        }
        throw new IllegalArgumentException("Unexpected state value " + state);
    }

    private void forwardPass(LifecycleOwner lifecycleOwner) {
        Iterator<Entry<LifecycleObserver, ObserverWithState>> ascendingIterator =
                mObserverMap.iteratorWithAdditions();
        while (ascendingIterator.hasNext() && !mNewEventOccurred) {
            Entry<LifecycleObserver, ObserverWithState> entry = ascendingIterator.next();
            ObserverWithState observer = entry.getValue();
            while ((observer.mState.compareTo(mState) < 0 && !mNewEventOccurred
                    && mObserverMap.contains(entry.getKey()))) {
                pushParentState(observer.mState);
                // 生命周期状态分发
                observer.dispatchEvent(lifecycleOwner, upEvent(observer.mState));
                popParentState();
            }
        }
    }

    private void backwardPass(LifecycleOwner lifecycleOwner) {
        Iterator<Entry<LifecycleObserver, ObserverWithState>> descendingIterator =
                mObserverMap.descendingIterator();
        while (descendingIterator.hasNext() && !mNewEventOccurred) {
            Entry<LifecycleObserver, ObserverWithState> entry = descendingIterator.next();
            ObserverWithState observer = entry.getValue();
            while ((observer.mState.compareTo(mState) > 0 && !mNewEventOccurred
                    && mObserverMap.contains(entry.getKey()))) {
                Event event = downEvent(observer.mState);
                pushParentState(getStateAfter(event));
                // 生命周期状态分发
                observer.dispatchEvent(lifecycleOwner, event);
                popParentState();
            }
        }
    }

    // happens only on the top of stack (never in reentrance),
    // so it doesn't have to take in account parents
    private void sync() {
        LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
        if (lifecycleOwner == null) {
            throw new IllegalStateException("LifecycleOwner of this LifecycleRegistry is already"
                    + "garbage collected. It is too late to change lifecycle state.");
        }
        while (!isSynced()) {
            mNewEventOccurred = false;
            // no need to check eldest for nullability, because isSynced does it for us.
            if (mState.compareTo(mObserverMap.eldest().getValue().mState) < 0) {
                backwardPass(lifecycleOwner);
            }
            Entry<LifecycleObserver, ObserverWithState> newest = mObserverMap.newest();
            if (!mNewEventOccurred && newest != null
                    && mState.compareTo(newest.getValue().mState) > 0) {
                forwardPass(lifecycleOwner);
            }
        }
        mNewEventOccurred = false;
    }

    static class ObserverWithState {
        State mState;
        LifecycleEventObserver mLifecycleObserver;

        ObserverWithState(LifecycleObserver observer, State initialState) {
            // 根据observer生成对应类型的LifecycleEventObserver
            mLifecycleObserver = Lifecycling.lifecycleEventObserver(observer);
            mState = initialState;
        }

        void dispatchEvent(LifecycleOwner owner, Event event) {
            State newState = getStateAfter(event);
            mState = min(mState, newState);
            // 回调生命周期状态
            mLifecycleObserver.onStateChanged(owner, event);
            mState = newState;
        }
    }
}
```

分析源码可以发现，最终通过 `ObserverWithState` 的 `dispatchEvent()` 方法进行生命周期状态处理。

### Lifecycling ###
通过上面的源码分析可以发现，最终用到 `LifecycleEventObserver` 的哪个实现类取决于 `Lifecycling` 的 `lifecycleEventObserver()` 方法传入的 `observer`。`Lifecycling` 的源码如下：
```java
public class Lifecycling {

    @NonNull
    static LifecycleEventObserver lifecycleEventObserver(Object object) {
        boolean isLifecycleEventObserver = object instanceof LifecycleEventObserver;
        boolean isFullLifecycleObserver = object instanceof FullLifecycleObserver;
        // 直接实现FullLifecycleObserver和LifecycleEventObserver接口的方式
        if (isLifecycleEventObserver && isFullLifecycleObserver) {
            return new FullLifecycleObserverAdapter((FullLifecycleObserver) object,
                    (LifecycleEventObserver) object);
        }
        // 直接实现FullLifecycleObserver接口的方式
        if (isFullLifecycleObserver) {
            return new FullLifecycleObserverAdapter((FullLifecycleObserver) object, null);
        }
        // 直接实现LifecycleEventObserver接口的方式
        if (isLifecycleEventObserver) {
            return (LifecycleEventObserver) object;
        }

        // 使用了注解处理器(Annotation Processor)
        final Class<?> klass = object.getClass();
        int type = getObserverConstructorType(klass);
        if (type == GENERATED_CALLBACK) {
            List<Constructor<? extends GeneratedAdapter>> constructors =
                    sClassToAdapters.get(klass);
            if (constructors.size() == 1) {
                GeneratedAdapter generatedAdapter = createGeneratedAdapter(
                        constructors.get(0), object);
                return new SingleGeneratedAdapterObserver(generatedAdapter);
            }
            GeneratedAdapter[] adapters = new GeneratedAdapter[constructors.size()];
            for (int i = 0; i < constructors.size(); i++) {
                adapters[i] = createGeneratedAdapter(constructors.get(i), object);
            }
            return new CompositeGeneratedAdaptersObserver(adapters);
        }
        return new ReflectiveGenericLifecycleObserver(object);
    }

}
```

######
`DefaultLifecycleObserver` 接口继承了 `FullLifecycleObserver` 接口，如果是直接实现 `DefaultLifecycleObserver` 接口的方式，则会生成 `FullLifecycleObserverAdapter`，在其实现的 `onStatChanged()` 回调方法中会将最终的生命周期状态分发出去。`FullLifecycleObserverAdapter` 的源码如下：
```java
class FullLifecycleObserverAdapter implements LifecycleEventObserver {

    private final FullLifecycleObserver mFullLifecycleObserver;
    private final LifecycleEventObserver mLifecycleEventObserver;

    FullLifecycleObserverAdapter(FullLifecycleObserver fullLifecycleObserver,
            LifecycleEventObserver lifecycleEventObserver) {
        mFullLifecycleObserver = fullLifecycleObserver;
        mLifecycleEventObserver = lifecycleEventObserver;
    }

    @Override
    public void onStateChanged(@NonNull LifecycleOwner source, @NonNull Lifecycle.Event event) {
        // 根据具体的event回调到对应的生命周期方法中
	switch (event) {
            case ON_CREATE:
                mFullLifecycleObserver.onCreate(source);
                break;
            case ON_START:
                mFullLifecycleObserver.onStart(source);
                break;
            case ON_RESUME:
                mFullLifecycleObserver.onResume(source);
                break;
            case ON_PAUSE:
                mFullLifecycleObserver.onPause(source);
                break;
            case ON_STOP:
                mFullLifecycleObserver.onStop(source);
                break;
            case ON_DESTROY:
                mFullLifecycleObserver.onDestroy(source);
                break;
            case ON_ANY:
                throw new IllegalArgumentException("ON_ANY must not been send by anybody");
        }
        if (mLifecycleEventObserver != null) {
            mLifecycleEventObserver.onStateChanged(source, event);
        }
    }
}
```

## 参考 ##
[Handling Lifecycles](https://developer.android.com/topic/libraries/architecture/lifecycle)

