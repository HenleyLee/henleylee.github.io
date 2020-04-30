---
title: Android Jetpack 组件之 LiveData 详解
categories: Android
tags:
  - Android
  - Jetpack
abbrlink: fd298b9f
date: 2019-11-17 18:36:32
---

**`LiveData`** 是一个可观察的数据持有者类，与常规 `Observable` 不同，`LiveData` 是生命周期感知的，`LiveData` 也是 Android Jetpack 组件的一部分。

## 简介 ##
`LiveData` 是一个可观察的数据持有者类，与常规的 `Observable` 不同，`LiveData` 可感知 `Activity`、`Fragment`、`Service` 的生命周期，确保 `LiveData` 仅更新处于活动生命周期状态的组件观察者。

如果应用程序组件观察者所处的状态是 `STARTED` 或 `RESUMED`，则 `LiveData` 认为该组件处于活跃状态，该组件会收到 `LiveData` 的数据更新，而其他注册的组件观察者将不会收到任何数据更新。如果组件观察者所处的状态改变为 `DESTROYED` 时，则移除该观察者。

`LiveData` 的创建基本会在 `ViewModel` 中，从而使数据在界面销毁时继续保持。

## 优点 ##
`LiveData` 具有以下优点：
 - **保持 UI 与数据的一致性：**LiveData 遵循观察者设计模式，生命周期发生变化时，LiveData 会通知对应的应用程序组件(Observer)，数据发生变化时也会通知更新 UI。
 - **避免内存泄漏：**这个 Observer 绑定了 Lifecycle 对象，当 Lifecycle 对象生命周期 destory 之后，这些 Observer 也会被自动清理。
 - **避免 Activity 处于不活跃状态的时候产生崩溃：**如果观察者(Observer)处于不活跃状态，则 Observer 不会接收任何 LiveData 事件。
 - **不在手动处理生命周期：**UI 组件只是观察相关数据，而不会停止或恢复观察，LiveData 会根据具体生命周期的变化而自动管理。
 - **始终保持最新数据：**如果生命周期为非活跃状态，则会在由非活跃状态转为活跃状态时接收最新数据，如从后台切换到前台自动接收最新数据。
 - **正确处理配置更改：**如果 Activity 或 Fragment 因为设备配置发生变化而重新创建，比如屏幕旋转等，也将会立即重新接收最新数据。
 - **共享服务：**可以借助 LiveData 数据的观察能力，根据 Livecycle 的生命周期状态随时连接或断开服务。

## 使用 ##
在某个具体的 `ViewModel` 类中定义 `LiveData` 数据，然后在对应的 `Activity` 或 `Fragment` 中观察 `LiveData` 数据的变化，`LiveData` 的使用使得我们不在将数据保存在 `Activity` 或 `Fragment` 中，而是将 `LiveData` 对象中存储的数据保存在 `ViewModel` 中，减轻了 `Activity` 和 `Fragment` 的工作量，使得 `Activity` 和 `Fragment` 只负责界面管理和显示，而不在保存数据，且在配置更改时数据不受影响。

`LiveData` 的使用步骤如下：
 - 在 ViewModel 创建具体的 LiveData 实例来存储数据；
 - 用 LiveData 对象的 observe() 方法将对应的 Activity 或 Fragment 等添加为该 LiveData 对象的观察者；
 - 使用 LiveData 的 setValue() 或 postValue() 方法更新数据，然后在观察者(Activity/Fragment)中获取更新数据。

> 如果需要创建没有 `LifecycleOwner` 的观察者，可以使用 `LiveData` 对象的 `observeForever()` 方法来将一个没有 `LifecycleOwner` 的类添加到观察者列表中。不过需要注意的是，使用 `observeForever()` 方法添加的观察者对象会一直处于活跃状态，此时就需要手动调用 `removeObserver()` 方法移除该观察者。

## 转换 ##
`LiveData` 提供了工具类 `Transformations` 来对 `LiveData` 的数据类型进行转换，可以在 `LiveData` 的数据返回给观察者之前修改 `LiveData` 中数据的具体类型，比如 int 型数字 1、2 等转化为中文大写壹、贰等，那么如何使用呢，创建 MapViewModel 如下：

### map() ###
`LiveData<Y> map(LiveData<X> source, Function<X, Y> mapFunction)` 提供一个方法去在 `LiveData.setValue()` 之前对 `value` 进行相应的转换和处理。代码示例如下：
```java
private LiveData<User> userLiveData = ...;
private LiveData<String> userFullNameLiveData = Transformations.map(userLiveData, new Function<User, String>() {
    @Override
    public String apply(User user) {
        return user.firstName + user.lastName;
    }
});
```

### switchMap() ###
LiveData<Y> switchMap(LiveData<X> source, Function<X, LiveData<Y>> switchMapFunction)：与 `map()`类似，但是是对 `LiveData` 进行转换，而不是对应的 `value`。代码示例如下：
```java
class UserViewModel extends AndroidViewModel {

    MutableLiveData<String> nameQueryLiveData = new MutableLiveData<>();

    LiveData<List<String>> getUsersWithNameLiveData() {
        return Transformations.switchMap(nameQueryLiveData, new Function<String, LiveData<List<String>>>() {
            @Override
            public LiveData<List<String>> apply(String name) {
                return mDataSource.getUsersWithNameLiveData(name);
            }
        });
    }

    void setNameQuery(String name) {
        this.nameQueryLiveData.setValue(name);
    }

}
```

### 源码解析 ###
`Transformations` 的源码如下：
```java
public class Transformations {

    @MainThread
    @NonNull
    public static <X, Y> LiveData<Y> map(
            @NonNull LiveData<X> source,
            @NonNull final Function<X, Y> mapFunction) {
        // 创建一个MediatorLiveData对象
        final MediatorLiveData<Y> result = new MediatorLiveData<>();
        result.addSource(source, new Observer<X>() {
            @Override
            public void onChanged(@Nullable X x) {
                // 在setValue()前执行了mapFunction并且将新的值赋值给LiveData
                result.setValue(mapFunction.apply(x));
            }
        });
        return result;
    }

    @MainThread
    @NonNull
    public static <X, Y> LiveData<Y> switchMap(
            @NonNull LiveData<X> source,
            @NonNull final Function<X, LiveData<Y>> switchMapFunction) {
        // 创建一个MediatorLiveData对象
        final MediatorLiveData<Y> result = new MediatorLiveData<>();
        result.addSource(source, new Observer<X>() {
            LiveData<Y> mSource;

            @Override
            public void onChanged(@Nullable X x) {
                // 得到转换后的LiveData
                LiveData<Y> newLiveData = switchMapFunction.apply(x);
                // 如果新获得的LiveData没有发生变化，则直接返回
                if (mSource == newLiveData) {
                    return;
                }
                // 停止监听源LiveData
                if (mSource != null) {
                    result.removeSource(mSource);
                }
                // 将新得到的LiveData赋值给源LiveData
                mSource = newLiveData;
                // 重新重新绑定观察者
                if (mSource != null) {
                    result.addSource(mSource, new Observer<Y>() {
                        @Override
                        public void onChanged(@Nullable Y y) {
                            result.setValue(y);
                        }
                    });
                }
            }
        });
        return result;
    }

}
```

> `map()` 方法和 `switchMap()` 方法内部都是由 `MediatorLiveData` 参与转换。`MediatorLiveData` 是 `LiveData` 的子类，它可以用来正确的处理其他多个 `LiveData` 的事件变化并处理这些事件。`MediatorLiveData` 会将自身的 `active/inactive` 状态变化正确的传递给它所处理的 `LiveData`。可以自己使用 `MediatorLiveData` 来实现更多类似 `map()` 、 `switchMap()` 这样的数据类型转换。

## 扩展 ##
`LocationLiveData` 是一个继承 `LiveData` 的自定义 `LiveData`：
```java
public class LocationLiveData extends LiveData<Location> {
    private LocationManager locationManager;

    private SimpleLocationListener listener = new SimpleLocationListener() {
        @Override
        public void onLocationChanged(Location location) {
            setValue(location);
        }
    };

    public LocationLiveData(Context context) {
        locationManager = (LocationManager) context.getSystemService(
                Context.LOCATION_SERVICE);
    }

    @Override
    protected void onActive() {
        locationManager.requestLocationUpdates(LocationManager.GPS_PROVIDER, 0, 0, listener);
    }

    @Override
    protected void onInactive() {
        locationManager.removeUpdates(listener);
    }
}
```

上面有三个值得注意的地方：
 - **onActive()：**当该方法被调用时，表示 LiveData 的观察者数量从0变为了1，这时就应该注册位置监听了。
 - **onInactive()：**当该方法被调用时，表示 LiveData 的观察者数量变为了0，既然没有了观察者，也就没有理由再做监听，此时就应该将位置监听移除。
 - **setValue()：**通过调用该方法来更新 LiveData 的数据，并通知处于活动状态的观察者。

## 源码分析 ##
### LiveData ###
`LiveData` 的源码如下：
```java
public abstract class LiveData<T> {
    
    private final Runnable mPostValueRunnable = new Runnable() {
        @SuppressWarnings("unchecked")
        @Override
        public void run() {
            Object newValue;
            synchronized (mDataLock) {
                newValue = mPendingData;
                mPendingData = NOT_SET;
            }
            setValue((T) newValue);
        }
    };

    /**
     * 创建一个使用给定值初始化的LiveData
     */
    public LiveData(T value) {
        mData = value;
        mVersion = START_VERSION + 1;
    }

    /**
     * 创建一个未分配任何值的LiveData
     */
    public LiveData() {
        mData = NOT_SET;
        mVersion = START_VERSION;
    }

    @SuppressWarnings("unchecked")
    private void considerNotify(ObserverWrapper observer) {
        if (!observer.mActive) {
            return;
        }
        // Check latest state b4 dispatch. Maybe it changed state but we didn't get the event yet.
        //
        // we still first check observer.active to keep it as the entrance for events. So even if
        // the observer moved to an active state, if we've not received that event, we better not
        // notify for a more predictable notification order.
        if (!observer.shouldBeActive()) {
            observer.activeStateChanged(false);
            return;
        }
        if (observer.mLastVersion >= mVersion) {
            return;
        }
        observer.mLastVersion = mVersion;
        // 调用mObserver的onChange()方法，通知观察者数据改变
        observer.mObserver.onChanged((T) mData);
    }

    @SuppressWarnings("WeakerAccess") /* synthetic access */
    void dispatchingValue(@Nullable ObserverWrapper initiator) {
        if (mDispatchingValue) {
            mDispatchInvalidated = true;
            return;
        }
        mDispatchingValue = true;
        do {
            mDispatchInvalidated = false;
            if (initiator != null) {
                // 调用considerNotify()方法更新数据
                considerNotify(initiator);
                initiator = null;
            } else {
                // 遍历mObservers中所有的Observer，调用considerNotify()方法更新数据
                for (Iterator<Map.Entry<Observer<? super T>, ObserverWrapper>> iterator =
                     mObservers.iteratorWithAdditions(); iterator.hasNext(); ) {
                    considerNotify(iterator.next().getValue());
                    if (mDispatchInvalidated) {
                        break;
                    }
                }
            }
        } while (mDispatchInvalidated);
        mDispatchingValue = false;
    }

    /**
     * 将给定的观察者添加到给定所有者的生命周期内的观察者列表中
     */
    @MainThread
    public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<? super T> observer) {
        assertMainThread("observe");
        // 判断了当前Lifecycler的状态，当状态为DESTROYED时即观察者不处于活跃状态，不用接收数据
        if (owner.getLifecycle().getCurrentState() == DESTROYED) {
            // ignore
            return;
        }
        // 创建LifecycleBoundObserver实例保存传入的LifecycleOwner和Observer，并保存在mObservers
        LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
        ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
        if (existing != null && !existing.isAttachedTo(owner)) {
            throw new IllegalArgumentException("Cannot add the same observer"
                    + " with different lifecycles");
        }
        if (existing != null) {
            return;
        }
        // 添加一个LifecycleObserver，当LifecycleOwner更改状态时，将通知该观察者
        owner.getLifecycle().addObserver(wrapper);
    }

    /**
     * 将给定的没有LifecycleOwner的观察者添加到观察者列表中
     */
    @MainThread
    public void observeForever(@NonNull Observer<? super T> observer) {
        assertMainThread("observeForever");
        AlwaysActiveObserver wrapper = new AlwaysActiveObserver(observer);
        ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
        if (existing instanceof LifecycleBoundObserver) {
            throw new IllegalArgumentException("Cannot add the same observer"
                    + " with different lifecycles");
        }
        if (existing != null) {
            return;
        }
        wrapper.activeStateChanged(true);
    }

    /**
     * 从观察者列表中删除给定的观察者
     */
    @MainThread
    public void removeObserver(@NonNull final Observer<? super T> observer) {
        assertMainThread("removeObserver");
        ObserverWrapper removed = mObservers.remove(observer);
        if (removed == null) {
            return;
        }
        // 分离观察者
        removed.detachObserver();
        // 处理观察者的活动状态改变
        removed.activeStateChanged(false);
    }

    /**
     * 删除与给定的{@link LifecycleOwner}关联的所有观察者
     */
    @SuppressWarnings("WeakerAccess")
    @MainThread
    public void removeObservers(@NonNull final LifecycleOwner owner) {
        assertMainThread("removeObservers");
        for (Map.Entry<Observer<? super T>, ObserverWrapper> entry : mObservers) {
            if (entry.getValue().isAttachedTo(owner)) {
                removeObserver(entry.getKey());
            }
        }
    }

    /**
     * 设置值(用于在非主线程中更新数据)
     */
    protected void postValue(T value) {
        boolean postTask;
        synchronized (mDataLock) {
            postTask = mPendingData == NOT_SET;
            mPendingData = value;
        }
        if (!postTask) {
            return;
        }
        // 通过ArchTaskExecutor将操作切换到主线程，mPostValueRunnable内部最终调用的setValue()方法
        ArchTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);
    }

    /**
     * 设置值(如果有活动的观察者，那么值将分发给它们)
     * <p>
     * 该方法必须从主线程调用。如果需要通过后台线程设置值，则可以使用postValue()分发
     */
    @MainThread
    protected void setValue(T value) {
        assertMainThread("setValue");
        mVersion++;
        mData = value;
        dispatchingValue(null);
    }

    /**
     * 返回当前值
     * <p>
     * 在后台线程上调用此方法并不能保证将接收到最新的值集
     */
    @SuppressWarnings("unchecked")
    @Nullable
    public T getValue() {
        Object data = mData;
        if (data != NOT_SET) {
            return (T) data;
        }
        return null;
    }

    int getVersion() {
        return mVersion;
    }

    /**
     * 当活动观察者的数量从0变为1时调用
     */
    protected void onActive() {

    }

    /**
     * 当活动观察者的数量从1变为0时调用
     */
    protected void onInactive() {

    }

    /**
     * 判断LiveData是否具有观察者
     */
    @SuppressWarnings("WeakerAccess")
    public boolean hasObservers() {
        return mObservers.size() > 0;
    }

    /**
     * 判断LiveData是否具有活动的观察者
     */
    @SuppressWarnings("WeakerAccess")
    public boolean hasActiveObservers() {
        return mActiveCount > 0;
    }

    /**
     * LifecycleBoundObserver主要利用Lifecycler的生命周期观察者GenericLifecycleObserver，
     * 前面设置了owner.getLifecycle().addObserver(wrapper)后，当生命周期改变时会回调onStateChange()方法，在生命周期为DESTROYED时移除Observer
     */
    class LifecycleBoundObserver extends ObserverWrapper implements LifecycleEventObserver {
        @NonNull
        final LifecycleOwner mOwner;    // 保存LifecycleOwner

        LifecycleBoundObserver(@NonNull LifecycleOwner owner, Observer<? super T> observer) {
            super(observer);// 调用父类ObserverWrapper的构造函数传递Owner
            mOwner = owner;
        }

        @Override
        boolean shouldBeActive() {
            return mOwner.getLifecycle().getCurrentState().isAtLeast(STARTED);
        }

        /**
         * 实现LifecycleEventObserver当生命周期改变时回调onStateChanged
         */
        @Override
        public void onStateChanged(@NonNull LifecycleOwner source, @NonNull Lifecycle.Event event) {
            // 当生命周期改变时会回调onStateChange()方法，在生命周期为DESTROYED时移除Observer
            if (mOwner.getLifecycle().getCurrentState() == DESTROYED) {
                removeObserver(mObserver);
                return;
            }
            // 处理观察者的活动状态改变
            activeStateChanged(shouldBeActive());
        }

        @Override
        boolean isAttachedTo(LifecycleOwner owner) {
            return mOwner == owner;
        }

        @Override
        void detachObserver() {
            mOwner.getLifecycle().removeObserver(this);
        }
    }

    /**
     * ObserverWrapper在Owner活跃状态改变时回调onActive()和onInactive()方法
     */
    private abstract class ObserverWrapper {
        final Observer<? super T> mObserver;
        boolean mActive;
        int mLastVersion = START_VERSION;

        ObserverWrapper(Observer<? super T> observer) {
            mObserver = observer;// 保存观察者Observer
        }

        abstract boolean shouldBeActive();

        boolean isAttachedTo(LifecycleOwner owner) {
            return false;
        }

        void detachObserver() {
        }

        void activeStateChanged(boolean newActive) {
            if (newActive == mActive) {
                return;
            }
            // immediately set active state, so we'd never dispatch anything to inactive
            // owner
            mActive = newActive;
            boolean wasInactive = LiveData.this.mActiveCount == 0;
            LiveData.this.mActiveCount += mActive ? 1 : -1;
            if (wasInactive && mActive) {
                // 当Owner为活跃状态时回调onActive()
                onActive();
            }
            if (LiveData.this.mActiveCount == 0 && !mActive) {
                // 当Owner未活跃状态时回调onInactive()
                onInactive();
            }
            if (mActive) {
                dispatchingValue(this);
            }
        }
    }

    /**
     * 一直活跃的观察者
     */
    private class AlwaysActiveObserver extends ObserverWrapper {

        AlwaysActiveObserver(Observer<? super T> observer) {
            super(observer);
        }

        @Override
        boolean shouldBeActive() {
            return true;
        }
    }

    static void assertMainThread(String methodName) {
        if (!ArchTaskExecutor.getInstance().isMainThread()) {
            throw new IllegalStateException("Cannot invoke " + methodName + " on a background"
                    + " thread");
        }
    }
}
```

### MutableLiveData ###
`MutableLiveData` 是 `LiveData` 的一个子类，它的源码如下：
```java
public class MutableLiveData<T> extends LiveData<T> {

    public MutableLiveData(T value) {
        super(value);
    }

    public MutableLiveData() {
        super();
    }

    @Override
    public void postValue(T value) {
        super.postValue(value);
    }

    @Override
    public void setValue(T value) {
        super.setValue(value);
    }

}
```

`MutableLiveData` 重写了 `LiveData` 中的 `postValue()` 和 `setValue()` 方法，内部也只是调用了 `super.postValue()` 和 `super.setValue()`，也就是说所有的方法都是在 `LiveData` 中实现。

## 参考 ##
[LiveData Overview](https://developer.android.com/topic/libraries/architecture/livedata)

