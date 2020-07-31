---
title: Android Jetpack 组件之 ViewModel 详解
categories: Android
tags:
  - Android
  - Jetpack
abbrlink: 6efd8b7a
date: 2019-11-27 18:36:32
---

Android 框架可以管理 UI 控制器(`Activity` 和 `Fragment`)的生命周期。Android 框架可能会决定销毁或重新创建 UI 控制器，以响应完全不受控制的某些用户操作或设备事件。

如果系统销毁或重新创建 UI 控制器，则存储在其中的任何临时性 UI 相关数据都会丢失。例如，应用的某个 `Activity` 中可能包含用户列表，因配置更改而重新创建 `Activity` 后，新 `Activity` 必须重新提取用户列表。对于简单的数据，`Activity` 可以使用 `onSaveInstanceState()` 方法从 `onCreate()` 方法中的 `Bundle` 恢复其数据，但此方法仅适合可以序列化再反序列化的少量数据，而不适合数量可能较大的数据，如用户列表或位图。

`ViewModel` 类被设计用于以感知生命周期的方式来管理和存储 UI 界面相关的数据，例如当屏幕发生旋转、App 权限被动态修改、系统语言发生改变(Configuration changes)的时候，`Activity` 重建的时候，`ViewModel` 可以保留 UI 界面的数据。

## 优点 ##
`ViewModel` 具有以下优点：
 - 存储和管理数据
 - 避免内存泄漏
 - 解耦
 - 共享数据

## 依赖 ##
要将 `ViewModel` 库引入到 Android 项目中，请将以下依赖项添加到应用或模块的 `build.gradle` 文件中：
```gradle
dependencies {
    def lifecycle_version = "2.2.0"

    // ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel:$lifecycle_version"
    // KTX for ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
    // Saved state module for ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel-savedstate:$lifecycle_version"
}
```

## 使用 ##
架构组件为界面控制器提供了 `ViewModel` 辅助类，该类负责为 UI 界面准备数据。在配置更改期间会自动保留 `ViewModel` 对象，以便它们存储的数据立即可供下一个 Activity 或 Fragment 实例使用。

如果需要在应用中显示用户列表，请确保将获取和保留该用户列表的责任分配给 `ViewModel`，而不是 `Activity` 或 `Fragment`，示例代码如下所示：
```java
public class UserViewModel extends ViewModel {

    private MutableLiveData<List<User>> users;

    public LiveData<List<User>> getUsers() {
        if (users == null) {
            users = new MutableLiveData<List<User>>();
            loadUsers();
        }
        return users;
    }

    private void loadUsers() {
        // Do an asynchronous operation to fetch users.
    }

}
```

然后，可以从 `AppCompatActivity` 访问 `ViewModel` 中的数据，如下所示：
```java
public class UserActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // Create a ViewModel the first time the system calls an activity's onCreate() method.
        // Re-created activities receive the same UserViewModel instance created by the first activity.
        UserViewModel model = new ViewModelProvider(this).get(UserViewModel.class);
        model.getUsers().observe(this, new Observer<List<User>>() {
            @Override
            public void onChanged(List<User> userList) {
                // update UI
            }
        });
    }

}
```

如果重新创建了该 `Activity`，它接收的 `ViewModel` 实例与第一个 `Activity` 创建的实例相同。当所有者 `Activity` 销毁时，框架会调用 `ViewModel` 对象的 `onCleared()` 方法以便于释放资源。

> **注意：**`ViewModel` 绝不能引用 `View`、`Lifecycle` 或可能持有对 `Activity` 上下文的引用的任何类。

`ViewModel` 对象可以包含 `LifecycleObserver`，如 `LiveData` 对象。如果 `ViewModel` 需要持有 `Application` 上下文(可以查找系统服务)，则可以继承 `AndroidViewModel` 类并设置用于接收 `Application` 的构造函数。

## 生命周期 ##
`ViewModel` 对象存在的时间范围是获取 `ViewModel` 时传递给 `ViewModelProvider` 的 `Lifecycle`。`ViewModel` 将一直留在内存中，直到限定其存在时间范围的 `Lifecycle` 永久消失(Activity 被销毁或 Fragment 被移除)。`ViewModel` 的生命周期如下图所示：
![ViewModel的生命周期](https://henleylee.github.io/medias/android/viewmodel_lifecycle.png)

通常在系统首次调用 `Activity` 对象的 `onCreate()` 方法时创建 `ViewModel`。系统可能会在 `Activity` 的整个生命周期内多次调用 `onCreate()` 方法(如在旋转设备屏幕时)。`ViewModel` 存在的时间范围是从首次创建 `ViewModel` 直到 `Activity` 被销毁。

## 源码分析 ##
### ViewModelProviders ###
可以通过 `ViewModelProviders.of().get(modelClass)` 创建 `ViewModel`。`ViewModelProviders` 的源码如下：
```java
public class ViewModelProviders {

    /**
     * 通过正在活动的Fragment对象创建ViewModelProvider对象
     */
    @Deprecated
    @NonNull
    @MainThread
    public static ViewModelProvider of(@NonNull Fragment fragment) {
        return new ViewModelProvider(fragment);
    }

    /**
     * 通过正在活动的Activity对象创建ViewModelProvider对象
     */
    @Deprecated
    @NonNull
    @MainThread
    public static ViewModelProvider of(@NonNull FragmentActivity activity) {
        return new ViewModelProvider(activity);
    }

    /**
     * 通过正在活动的Fragment对象和指定Factory创建ViewModelProvider对象
     */
    @Deprecated
    @NonNull
    @MainThread
    public static ViewModelProvider of(@NonNull Fragment fragment, @Nullable Factory factory) {
        if (factory == null) {
            factory = fragment.getDefaultViewModelProviderFactory();
        }
        return new ViewModelProvider(fragment.getViewModelStore(), factory);
    }

    /**
     * 通过正在活动的Activity对象和指定Factory创建ViewModelProvider对象
     */
    @Deprecated
    @NonNull
    @MainThread
    public static ViewModelProvider of(@NonNull FragmentActivity activity,
                                       @Nullable Factory factory) {
        if (factory == null) {
            factory = activity.getDefaultViewModelProviderFactory();
        }
        return new ViewModelProvider(activity.getViewModelStore(), factory);
    }

}
```

`ViewModelProviders` 内部将 `Fragment` 和 `Activity` 转化为 `ViewModelStore`，实际还是通过 `ViewModelProvider` 类创建 `ViewModel`，并且 `ViewModelProviders` 类已被标记为过时。

### ViewModelProvider ###
可以通过 `new ViewModelProvider(owner).get(modelClass)` 创建 `ViewModel`。`ViewModelProvider` 的源码如下：
```java
public class ViewModelProvider {

    private static final String DEFAULT_KEY = "androidx.lifecycle.ViewModelProvider.DefaultKey";

    /** 实现负责实例化ViewModel的工厂 */
    public interface Factory {
        /**
         * 创建ViewModel实例
         */
        @NonNull
        <T extends ViewModel> T create(@NonNull Class<T> modelClass);
    }

    static class OnRequeryFactory {
        void onRequery(@NonNull ViewModel viewModel) {
        }
    }

    /** 根据Key获取ViewModel实例的Factory实现类 */
    abstract static class KeyedFactory extends OnRequeryFactory implements Factory {
        /**
         * 创建ViewModel实例
         */
        @NonNull
        public abstract <T extends ViewModel> T create(@NonNull String key, @NonNull Class<T> modelClass);

        @NonNull
        @Override
        public <T extends ViewModel> T create(@NonNull Class<T> modelClass) {
            throw new UnsupportedOperationException("create(String, Class<?>) must be called on "
                    + "implementaions of KeyedFactory");
        }
    }

    private final Factory mFactory;                 // 实现负责实例化ViewModel的工厂
    private final ViewModelStore mViewModelStore;   // 用于存储ViewModel的类

    /**
     * 构造方法
     */
    public ViewModelProvider(@NonNull ViewModelStoreOwner owner) {
        this(owner.getViewModelStore(), owner instanceof HasDefaultViewModelProviderFactory
                ? ((HasDefaultViewModelProviderFactory) owner).getDefaultViewModelProviderFactory()
                : NewInstanceFactory.getInstance());
    }

    /**
     * 构造方法
     */
    public ViewModelProvider(@NonNull ViewModelStoreOwner owner, @NonNull Factory factory) {
        this(owner.getViewModelStore(), factory);
    }

    /**
     * 构造方法
     */
    public ViewModelProvider(@NonNull ViewModelStore store, @NonNull Factory factory) {
        mFactory = factory;
        mViewModelStore = store;
    }

    /**
     * 返回与该ViewModelProvider关联的现有ViewModel或创建新的ViewModel
     */
    @NonNull
    @MainThread
    public <T extends ViewModel> T get(@NonNull Class<T> modelClass) {
        String canonicalName = modelClass.getCanonicalName();
        if (canonicalName == null) {
            throw new IllegalArgumentException("Local and anonymous classes can not be ViewModels");
        }
        // 创建、存储ViewModel使用的key
        return get(DEFAULT_KEY + ":" + canonicalName, modelClass);
    }

    /**
     * 返回与该ViewModelProvider关联的现有ViewModel或创建新的ViewModel
     */
    @SuppressWarnings("unchecked")
    @NonNull
    @MainThread
    public <T extends ViewModel> T get(@NonNull String key, @NonNull Class<T> modelClass) {
        // 从ViewModelStore中通过key获取ViewModel对象
        ViewModel viewModel = mViewModelStore.get(key);
        // 通过key获取的ViewModel对象与想要的Class相同直接返回
        if (modelClass.isInstance(viewModel)) {
            return (T) viewModel;
        } else {
            //noinspection StatementWithEmptyBody
            if (viewModel != null) {
                // TODO: log a warning.
            }
        }
        // 通过ViewModelFactory创建ViewModel类
        if (mFactory instanceof KeyedFactory) {
            viewModel = ((KeyedFactory) (mFactory)).create(key, modelClass);
        } else {
            viewModel = (mFactory).create(modelClass);
        }
        // 将创建的ViewModel对象添加到ViewModelStore中
        mViewModelStore.put(key, viewModel);
        return (T) viewModel;
    }

    /** 简单工厂(调用给定Class的空构造函数) */
    public static class NewInstanceFactory implements Factory {

        private static NewInstanceFactory sInstance;

        /**
         * 获取NewInstanceFactory实例(单例模式)
         */
        @NonNull
        static NewInstanceFactory getInstance() {
            if (sInstance == null) {
                sInstance = new NewInstanceFactory();
            }
            return sInstance;
        }

        @SuppressWarnings("ClassNewInstance")
        @NonNull
        @Override
        public <T extends ViewModel> T create(@NonNull Class<T> modelClass) {
            try {
                // 调用给定Class的空构造函数创建ViewModel实例
                return modelClass.newInstance();
            } catch (InstantiationException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            } catch (IllegalAccessException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            }
        }
    }

}
```

### ViewModelStore ###
`ViewModelStore` 类用于存储创建的 `ViewModel`，避免 `ViewModel` 被重复创建。`ViewModelStore` 的源码如下：
```java
public class ViewModelStore {

    private final HashMap<String, ViewModel> mMap = new HashMap<>();

    /**
     * 根据key存储ViewModel对象
     */
    final void put(String key, ViewModel viewModel) {
        ViewModel oldViewModel = mMap.put(key, viewModel);
        if (oldViewModel != null) {
            oldViewModel.onCleared();
        }
    }

    /**
     * 根据key获取存储的ViewModel对象
     */
    final ViewModel get(String key) {
        return mMap.get(key);
    }

    Set<String> keys() {
        return new HashSet<>(mMap.keySet());
    }

    /**
     * 清除内部存储的ViewModel并通知它们不再使用
     */
    public final void clear() {
        for (ViewModel vm : mMap.values()) {
            vm.clear();
        }
        mMap.clear();
    }

}
```
`ViewModelStore` 内部其实就是一个存储 `ViewModel` 的 `HashMap`，并且提供 `clear()` 方法清空所有内容。

### SavedStateViewModelFactory ###
`Fragment` 和 `ComponentActivity` 都实现了 `HasDefaultViewModelProviderFactory` 和 `ViewModelStoreOwner` 接口，因此 `ViewModelStore` 类中创建 `ViewModel` 的 `Factory` 都是通过 `Fragment` 和 `ComponentActivity` 的 `getDefaultViewModelProviderFactory()` 方法得到的。`getDefaultViewModelProviderFactory()` 方法的源码如下：
```java
public ViewModelProvider.Factory getDefaultViewModelProviderFactory() {
    if (getApplication() == null) {
        throw new IllegalStateException("Your activity is not yet attached to the "
                + "Application instance. You can't request ViewModel before onCreate call.");
    }
    if (mDefaultFactory == null) {
        mDefaultFactory = new SavedStateViewModelFactory(
                getApplication(),
                this,
                getIntent() != null ? getIntent().getExtras() : null);
    }
    return mDefaultFactory;
}
```

通过分析源码可以发现，最终的创建 `ViewModel` 的 `Factory` 就是 `SavedStateViewModelFactory` 类，`SavedStateViewModelFactory` 继承了 `ViewModelProvider` 中的 `KeyedFactory` 抽象类。`SavedStateViewModelFactory` 的源码如下：
```java
public final class SavedStateViewModelFactory extends ViewModelProvider.KeyedFactory {

    private final Application mApplication;
    private final ViewModelProvider.AndroidViewModelFactory mFactory; // 创建ViewModel的工厂
    private final Bundle mDefaultArgs;                     // 来自Fragment/Activity对象
    private final Lifecycle mLifecycle;                    // 来自Fragment/Activity对象
    private final SavedStateRegistry mSavedStateRegistry;  // 来自Fragment/Activity对象

    /**
     * 构造方法
     */
    public SavedStateViewModelFactory(@NonNull Application application,
                                      @NonNull SavedStateRegistryOwner owner,
                                      @Nullable Bundle defaultArgs) {
        mSavedStateRegistry = owner.getSavedStateRegistry();
        mLifecycle = owner.getLifecycle();
        mDefaultArgs = defaultArgs;
        mApplication = application;
        mFactory = ViewModelProvider.AndroidViewModelFactory.getInstance(application);
    }

    /**
     * 创建ViewModel
     */
    @NonNull
    @Override
    public <T extends ViewModel> T create(Class<T> modelClass) {
        // 用作创建ViewModel的key
        String canonicalName = modelClass.getCanonicalName();
        if (canonicalName == null) {
            throw new IllegalArgumentException("Local and anonymous classes can not be ViewModels");
        }
        return create(canonicalName, modelClass);
    }

    /**
     * 创建ViewModel
     */
    @NonNull
    @Override
    public <T extends ViewModel> T create(@NonNull String key, @NonNull Class<T> modelClass) {
        // 判断是否为AndroidViewModel的实现类
        boolean isAndroidViewModel = AndroidViewModel.class.isAssignableFrom(modelClass);
        // 查找匹配的构造方法
        Constructor<T> constructor;
        if (isAndroidViewModel) {
            constructor = findMatchingConstructor(modelClass, ANDROID_VIEWMODEL_SIGNATURE);
        } else {
            constructor = findMatchingConstructor(modelClass, VIEWMODEL_SIGNATURE);
        }
        // doesn't need SavedStateHandle
        if (constructor == null) {
            // 这里调用的是 AndroidViewModelFactory 的 create 方法
            return mFactory.create(modelClass);
        }
        // SavedStateHandleController实现LifecycleEventObserver接口
        SavedStateHandleController controller = SavedStateHandleController.create(
                mSavedStateRegistry, mLifecycle, key, mDefaultArgs);
        try {
            // 创建ViewModel对象
            T viewmodel;
            if (isAndroidViewModel) {
                viewmodel = constructor.newInstance(mApplication, controller.getHandle());
            } else {
                viewmodel = constructor.newInstance(controller.getHandle());
            }
            // 保存controller到ViewModel实例中
            viewmodel.setTagIfAbsent(TAG_SAVED_STATE_HANDLE_CONTROLLER, controller);
            return viewmodel;
        } catch (IllegalAccessException e) {
            throw new RuntimeException("Failed to access " + modelClass, e);
        } catch (InstantiationException e) {
            throw new RuntimeException("A " + modelClass + " cannot be instantiated.", e);
        } catch (InvocationTargetException e) {
            throw new RuntimeException("An exception happened in constructor of "
                    + modelClass, e.getCause());
        }
    }


    private static final Class<?>[] ANDROID_VIEWMODEL_SIGNATURE = new Class[]{Application.class,
            SavedStateHandle.class};
    private static final Class<?>[] VIEWMODEL_SIGNATURE = new Class[]{SavedStateHandle.class};

    @SuppressWarnings("unchecked")
    private static <T> Constructor<T> findMatchingConstructor(Class<T> modelClass,
                                                              Class<?>[] signature) {
        for (Constructor<?> constructor : modelClass.getConstructors()) {
            Class<?>[] parameterTypes = constructor.getParameterTypes();
            if (Arrays.equals(signature, parameterTypes)) {
                return (Constructor<T>) constructor;
            }
        }
        return null;
    }
    
}
```

### SavedStateHandleController ###
`SavedStateViewModelFactory` 在创建 `ViewModel` 时用到了 `SavedStateViewModelFactory` 类，`SavedStateViewModelFactory` 实际上是用于保存页面数据及根据生命周期事件来对自己进行管理。`SavedStateViewModelFactory` 的 `create()` 方法源码如下：
```java
static SavedStateHandleController create(SavedStateRegistry registry, Lifecycle lifecycle,
                                             String key, Bundle defaultArgs) {
    // 消耗先前由SavedStateProvider提供的保存状态，该状态是通过key存储的Bundle对象
    Bundle restoredState = registry.consumeRestoredStateForKey(key);
    // 将之前的保存状态和现在进行合并，放到SavedStateHandle中
    SavedStateHandle handle = SavedStateHandle.createHandle(restoredState, defaultArgs);
    // 创建持有key和handle的SavedStateHandleController对象
    SavedStateHandleController controller = new SavedStateHandleController(key, handle);
    // 添加监听和注册SavedStateProvider到SavedStateRegistry里
    controller.attachToLifecycle(registry, lifecycle);
    tryToAddRecreator(registry, lifecycle);
    return controller;
}
```

### AndroidViewModelFactory ###
`AndroidViewModelFactory` 继承了 `NewInstanceFactory`，用于创建 `AndroidViewModel` 子类的实例，满足那些需要持有 `Context` 实例的 `ViewModel` 能快速实现。

`AndroidViewModelFactory` 采用单例模式，通过反射调用带 `Application` 参数的构造器方法创建 `ViewModel` 实例，如果创建失败则调用默认构造器创建。`AndroidViewModelFactory` 的源码如下：
```java
/** 用于创建AndroidViewModel和ViewModel的工厂 */
public static class AndroidViewModelFactory extends ViewModelProvider.NewInstanceFactory {

    private static AndroidViewModelFactory sInstance;

    /**
     * 获取AndroidViewModelFactory实例(单例模式)
     */
    @NonNull
    public static AndroidViewModelFactory getInstance(@NonNull Application application) {
        if (sInstance == null) {
            sInstance = new AndroidViewModelFactory(application);
        }
        return sInstance;
    }

    private Application mApplication;

    /**
     * 创建AndroidViewModelFactory
     */
    public AndroidViewModelFactory(@NonNull Application application) {
        mApplication = application;
    }

    @NonNull
    @Override
    public <T extends ViewModel> T create(@NonNull Class<T> modelClass) {
        // 判断modelClass对象表示的类是否为AndroidViewModel的子类
        if (AndroidViewModel.class.isAssignableFrom(modelClass)) {
            try {
                // 通过反射调用带Application参数的构造器方法创建AndroidViewModel实例
                return modelClass.getConstructor(Application.class).newInstance(mApplication);
            } catch (NoSuchMethodException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            } catch (IllegalAccessException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            } catch (InstantiationException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            } catch (InvocationTargetException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            }
        }
        // 调用父类的create()方法创建ViewModel实例(内部实际上是调用默认构造器创建)
        return super.create(modelClass);
    }
    
}
```


## 参考 ##
[ViewModel Overview](https://developer.android.google.cn/topic/libraries/architecture/viewmodel)

