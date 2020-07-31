---
title: Android Jetpack 组件之 App Startup 详解
categories: Android
tags:
  - Android
  - Jetpack
abbrlink: 48ae060
date: 2020-07-10 12:36:32
---

`App Startup` 和 `Lifecycle` 、`DataBinding` 一样都是 `Jetpack` 的组件之一。

官方网址：[https://developer.android.com/topic/libraries/app-startup](https://developer.android.com/topic/libraries/app-startup)

## App Startup 是什么？ ##
按照 Google 官方给出的定义：`App Startup` 是 `Android Jetpack` 最新成员，提供了在 App 启动时初始化组件简单、高效的方法，无论是 Library 开发人员还是 App 开发人员都可以使用 App Startup 显示的设置初始化顺序。

简单的说就是 `App Startup` 提供了一个 `ContentProvider` 来运行所有依赖项的初始化，避免每个第三方库单独使用 `ContentProvider` 进行初始化，从而提高了应用的程序的启动速度。

## App Startup 解决了什么问题？ ##
在实际的开发工作中，无论是 Google 提供的库还是第三方库，App 启动运行时会初始化一些逻辑，它们为了方便开发者使用，避免开发者手动调用，使用 ContentProvider 进行初始化。比如 `WorkManager` 和 `Lifecycle` 在 APP 启动时通过 `ContentProvider` 进行初始化。通过 `ContentProvider`，一旦 APP 冷启动后，在调用 `Application.onCreate()` 之前，`ContentProvider` 就可以自动执行初始化。

现在我们有三个库分别 LibraryA、LibraryB、和 LibraryC 它们使用自己的 ContentProviders 进行初始化，如下图所示(图片来自[Husayn Hakeem](https://proandroiddev.com/androidx-app-startup-698855342f80))：
![LibraryA, LibraryB, and LibraryC initialized using their own ContentProviders](https://henleylee.github.io/medias/android/startup_alone.png)

而 App Startup 提供了一个 ContentProvider 来运行所有依赖项的初始化（LibraryA、LibraryB、和 LibraryC），如下图所示：
![LibraryA, LibraryB, and LibraryC initialized by AndroidX Startup](https://henleylee.github.io/medias/android/startup_unity.png)

## 如何正确使用 AndroidX App Startup？ ##

### 添加依赖 ###
要在您的应用中使用 `Android Startup`，请将以下内容添加到 `build.gradle` 文件中：
```gradle
dependencies {
    implementation "androidx.startup:startup-runtime:1.0.0-alpha01"
}
```

### 初始化组件 ###
可以通过创建实现 [`Initializer<T>`](https://developer.android.com/reference/kotlin/androidx/startup/Initializer) 接口的类来定义每个组件的初始化。该接口定义了两个重要的方法：
 - `create(context)：`其中包含初始化组件并返回 `T` 的实例的所有必需操作。
 - `dependency()：`返回初始化程序依赖的其他 `Initializer<T>` 对象的列表，可以使用此方法来控制应用程序在启动时运行初始化程序的顺序。

假设应用程序依赖于 `WorkManager`，并且需要在启动时对其进行初始化。定义一个实现 `Initializer<WorkManager>` 的 `WorkManagerInitializer` 类：
```kotlin
// Initializes WorkManager.
class WorkManagerInitializer : Initializer<WorkManager> {
    override fun create(context: Context): WorkManager {
        val configuration = Configuration.Builder().build()
        WorkManager.initialize(context, configuration)
        return WorkManager.getInstance(context)
    }
    override fun dependencies(): List<Class<out Initializer<*>>> {
        // No dependencies on other libraries.
        return emptyList()
    }
}
```
`dependency()` 方法返回一个空列表，因为 `WorkManager` 不依赖任何其他库。

假设应用程序还依赖于名为 `ExampleLogger` 的库，而该库又依赖于 `WorkManager`。这种依赖性意味着需要确保 `App Startup`首先初始化 `WorkManager`。定义一个实现 `Initializer<ExampleLogger>` 的 `ExampleLoggerInitializer` 类：
```kotlin
// Initializes ExampleLogger.
class ExampleLoggerInitializer : Initializer<ExampleLogger> {
    override fun create(context: Context): ExampleLogger {
        // WorkManager.getInstance() is non-null only after
        // WorkManager is initialized.
        return ExampleLogger(WorkManager.getInstance(context))
    }

    override fun dependencies(): List<Class<out Initializer<*>>> {
        // Defines a dependency on WorkManagerInitializer so it can be
        // initialized after WorkManager is initialized.
        return listOf(WorkManagerInitializer::class.java)
    }
}
```
因为在 `dependencies()` 方法中包括了 `WorkManagerInitializer`，所以 `App Startup` 在 `Logger` 之前初始化 `WorkManager`。

> 注意：如果以前使用 `ContentProviders` 来初始化应用程序中的组件，请确保在使用 `App Startup` 时删除这些 `ContentProviders`。

### 自动组件初始化 ###
`App Startup` 包含一个特殊的 `ContentProvider`，称为 `InitializationProvider`，它用来找到并且调用你的组件初始化器。那么这个过程是什么样的呢？
 - 首先，通过检查 `InitializationProvider` 清单标签下的 `<meta-data>` 标签，找到组件初始化器；
 - `App Startup` 调用它找到的所有组件初始化器的 `dependencies()` 方法。

这就意味着如果想要让 `App Startup` 找到组件初始化器，必须满足下面的一个条件：
 - 组件初始化器在 `InitializationProvider` 清单标签下配置了相应的 `<meta-data>` 标签；
 - 组件初始化器在一个已被找到的组件初始化器的 `dependencies()` 方法中被列出；

从上面写过的 `WorkManagerInitializer` 和 `ExampleLoggerInitializer` 这两个例子中来说，为了确保初始化器能被实现，需要在清单文件中配置如下代码：
```xml
<provider
    android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    android:exported="false"
    tools:node="merge">
    <!-- This entry makes ExampleLoggerInitializer discoverable. -->
    <meta-data
          android:name="com.example.ExampleLoggerInitializer"
          android:value="androidx.startup" />
</provider>
```

根据代码可以发现，只配置了 `ExampleLoggerInitializer` 的 `<meta-data>` 标签，因为它满足了第二条：在 `dependencies()` 方法中列出了它依赖于 `WorkManagerInitializer`，如果 `ExampleLoggerInitializer` 能被找到，那么 `WorkManagerInitializer` 一定也能被找到。

> `tools:node="merge"` 属性是为了确保[清单合并工具](https://developer.android.com/studio/build/manifest-merge)正确解决可能造成的冲突问题。

`App Startup` 库包含了一系列的 lint 规则，通过这些规则，你能够检查是否正确定义了组件初始化器。可以在终端通过 `./gradlew :app:lintDebug` 命令执行 lint 检查。

### 手动初始化组件 ###
通常来说，当使用 `App Startup` 时，`InitializationProvider` 对象就会使用 `AppInitializer` 在 App 启动时来自动寻找并且运行初始化器。但是，也可以直接调用 `AppInitializer` 来手动初始化不需要在启动时就调用的组件初始化器。这个操作被称作延迟初始化，它能够减少程序启动的时间。要想实现手动初始化，必须先禁止掉想要手动初始化的组件的自动初始化功能。

#### 禁用单个组件的自动初始化 ####
为了禁止掉单个组件的自动初始化功能，可以在清单文件中移除那个组件的 `<meta-data>` 标签，举例来说，如果想禁止 `ExampleLogger` 的自动初始化：
```xml
<provider
    android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    android:exported="false"
    tools:node="merge">
    <meta-data
              android:name="com.example.ExampleLoggerInitializer"
              tools:node="remove" />
</provider>
```

使用 `tools:node="remove"` 而不是直接移除这个标签是为了确保清单合并工具能够移除所有合并清单文件中的这个标签。

> 注意：禁用组件的自动初始化也会禁用该组件的依赖项的自动初始化。

#### 禁用所有组件的自动初始化 ####
现在知道了如何禁止单个组件的自动初始化，那么如何禁止所有组件的自动初始化，转而手动初始化呢？Android 官方提供了这个写法：
```xml
<provider
    android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    tools:node="remove" />
```

这种写法会移除清单文件中的 `InitializationProvider`。

#### 手动调用组件初始化程序 ####
如果为组件禁用了自动初始化，则可以使用 `AppInitializer` 手动初始化该组件及其依赖项。

例如，以下代码调用 `AppInitializer` 并手动初始化 `ExampleLogger`：
```kotlin
AppInitializer.getInstance(context)
    .initializeComponent(ExampleLoggerInitializer::class.java)
```

通过上述代码，`App Startup` 也对 `WorkManager` 进行了初始化，因为 `ExampleLogger` 依赖了 `WorkManager`。

## 源码分析 ##
`App Startup` 包中代码并不多，只有五个类：`Initializer`、`InitializationProvider`、`AppInitializer`、`StartupException` 和 `StartupLogger`。

其中最核心的类就是 `InitializationProvider`，它是继承了 `ContentProvider`，在 `onCreate()`方法中，可以看到它其实是调用了 `AppInitializer` 这个类中的 `discoverAndInitialize()` 方法，源码如下：
```java
public final class InitializationProvider extends ContentProvider {
    
    @Override
    public boolean onCreate() {
        Context context = getContext();
        if (context != null) {
            // 发现并初始化Initializer
            AppInitializer.getInstance(context).discoverAndInitialize();
        } else {
            throw new StartupException("Context cannot be null");
        }
        return true;
    }
    
}
```

`AppInitializer` 也是其中一个核心类，主要用于初始化所有发现的实现 `Initializer<T>` 接口的组件，源码如下：
```java
public final class AppInitializer {

    // Tracing
    private static final String SECTION_NAME = "Startup";
    private static AppInitializer sInstance;
    private static final Object sLock = new Object();
    final Map<Class<?>, Object> mInitialized;
    final Context mContext;

    /** 构造方法 */
    AppInitializer(@NonNull Context context) {
        mContext = context.getApplicationContext();
        mInitialized = new HashMap<>();
    }

    /**
     * 返回{@link AppInitializer}的实例
     */
    @NonNull
    @SuppressWarnings("UnusedReturnValue")
    public static AppInitializer getInstance(@NonNull Context context) {
        synchronized (sLock) {
            if (sInstance == null) {
                sInstance = new AppInitializer(context);
            }
            return sInstance;
        }
    }

    /**
     * 初始化{@link Initializer}
     */
    @NonNull
    @SuppressWarnings("unused")
    public <T> T initializeComponent(@NonNull Class<? extends Initializer<T>> component) {
        return doInitialize(component, new HashSet<Class<?>>());
    }

    @NonNull
    @SuppressWarnings({"unchecked", "TypeParameterUnusedInFormals"})
    <T> T doInitialize(
            @NonNull Class<? extends Initializer<?>> component,
            @NonNull Set<Class<?>> initializing) {
        synchronized (sLock) {
            boolean isTracingEnabled = Trace.isEnabled();
            try {
                if (isTracingEnabled) {
                    // Use the simpleName here because section names would get too big otherwise.
                    Trace.beginSection(component.getSimpleName());
                }
                if (initializing.contains(component)) {
                    String message = String.format(
                            "Cannot initialize %s. Cycle detected.", component.getName()
                    );
                    throw new IllegalStateException(message);
                }
                Object result;
                if (!mInitialized.containsKey(component)) {
                    initializing.add(component);
                    try {
                        Object instance = component.getDeclaredConstructor().newInstance();
                        Initializer<?> initializer = (Initializer<?>) instance;
                        // 获取该初始化器的依赖项
                        List<Class<? extends Initializer<?>>> dependencies =
                                initializer.dependencies();

                        // 先判断是否有依赖项
                        if (!dependencies.isEmpty()) {
                            // 遍历执行依赖项的初始化
                            for (Class<? extends Initializer<?>> clazz : dependencies) {
                                if (!mInitialized.containsKey(clazz)) {
                                    doInitialize(clazz, initializing);
                                }
                            }
                        }
                        if (StartupLogger.DEBUG) {
                            StartupLogger.i(String.format("Initializing %s", component.getName()));
                        }
                        // 执行初始化器自身的初始化
                        result = initializer.create(mContext);
                        if (StartupLogger.DEBUG) {
                            StartupLogger.i(String.format("Initialized %s", component.getName()));
                        }
                        initializing.remove(component);
                        mInitialized.put(component, result);
                    } catch (Throwable throwable) {
                        throw new StartupException(throwable);
                    }
                } else {
                    result = mInitialized.get(component);
                }
                return (T) result;
            } finally {
                Trace.endSection();
            }
        }
    }

    @SuppressWarnings("unchecked")
    void discoverAndInitialize() {
        try {
            Trace.beginSection(SECTION_NAME);
            ComponentName provider = new ComponentName(mContext.getPackageName(),
                    InitializationProvider.class.getName());
            // 解析InitializationProvider的数据
            ProviderInfo providerInfo = mContext.getPackageManager()
                    .getProviderInfo(provider, GET_META_DATA);
            // 获取<meta-data>标签中的数据
            Bundle metadata = providerInfo.metaData;
            String startup = mContext.getString(R.string.androidx_startup);
            if (metadata != null) {
                Set<Class<?>> initializing = new HashSet<>();
                Set<String> keys = metadata.keySet();
                // 遍历<meta-data>标签中配置的初始化器
                for (String key : keys) {
                    String value = metadata.getString(key, null);
                    if (startup.equals(value)) {
                        Class<?> clazz = Class.forName(key);
                        if (Initializer.class.isAssignableFrom(clazz)) {
                            Class<? extends Initializer<?>> component =
                                    (Class<? extends Initializer<?>>) clazz;
                            if (StartupLogger.DEBUG) {
                                StartupLogger.i(String.format("Discovered %s", key));
                            }
                            // 初始化每个初始化器
                            doInitialize(component, initializing);
                        }
                    }
                }
            }
        } catch (PackageManager.NameNotFoundException | ClassNotFoundException exception) {
            throw new StartupException(exception);
        } finally {
            Trace.endSection();
        }
    }

}
```

通过上述代码，可以发现 `App Startup` 就是解析出 `InitializationProvider` 中的 `<meta-data>` 标签，然后遍历 `<meta-data>` 标签配置的初始化器，然后调用每个初始化器的初始化方法，也就是 `doInitialize()` 方法。在 `doInitialize()` 方法中可以看到，在执行初始化的时候，先判断了是否有依赖项，有的话先执行依赖项的初始化，最后才执行自身的初始化。`initializeComponent()` 方法用于初始化传入的初始化器，内部实际也是调用了 `doInitialize()` 方法。

## 总结 ##
 - `App Startup` 是 `Jetpack` 的新成员，是为了解决因 App 启动时运行多个 `ContentProvider` 会增加 App 的启动时间的问题。
 - 使用了一个 `InitializationProvider` 管理多个依赖项，消除了每个库单独使用 `ContentProvider` 成本，减少初始化时间。
 - `App Startup` 允许自定义组件初始化顺序。
 - `App Startup` 可以自动初始化 `AndroidManifest.xml` 文件中 `InitializationProvider` 下面的 `<meta-data>` 声明要初始化的组件。
 - `App Startup` 提供了一种延迟初始化组件的方法，减少 App 初始化时间。

## 参考 ##
[App Startup](https://developer.android.com/topic/libraries/app-startup)

