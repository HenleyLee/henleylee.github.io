---
title: Android Jetpack 组件之 WorkManager 详解
categories: Android
tags:
  - Android
  - Jetpack
abbrlink: 5d46ccc
date: 2019-12-07 12:56:12
---

**`WorkManager`** 组件是由 Google 提供的用来管理后台工作任务。Android 已经有很多管理后台任务的类了，比如 `JobScheduler`、`AlarmManger`，再比如 `AsyncTask`、`ThreadPool`。`WorkManager` 的优势在哪里，为什么要使用 `WorkManager`。下面从两个方面来说明 `WorkManager` 的优势：
 - `WorkManager` 对比 `JobScheduler`、`AlarmManger` 的优势：虽然 `AlarmManager`是一直存在但是 `JobScheduler` 是 Android 5.0 之后才有的。`WorkManager` 的底层实现，会根据你的设备 API 的情况，自动选用 `JobScheduler` 或 `AlarmManager` 来实现后台任务。
 - `WorkManager` 对比 `AsyncTask`、`ThreadPool`的优势：`WorkManager` 里面的任务在应用退出之后还可以继续执行，`AsyncTask` 和 `ThreadPool` 里面的任务在应用退出之后不会执行。

> 使用 WorkManager API 可以轻松地调度即使在应用退出或设备重启时仍应运行的可延迟异步任务。

## 功能 ##
**`WorkManager`**具有以下功能：
 - 最高向后兼容到 API 14
  - 在运行 API 23 及以上级别的设备上使用 JobScheduler
  - 在运行 API 14-22 的设备上结合使用 BroadcastReceiver 和 AlarmManager
 - 添加网络可用性或充电状态等工作约束
 - 调度一次性或周期性异步任务
 - 监控和管理计划任务
 - 将任务链接起来
 - 确保任务执行，即使应用或设备重启也同样执行任务
 - 遵循低电耗模式等省电功能

`WorkManager` 旨在用于可延迟运行（即不需要立即运行）并且在应用退出或设备重启时必须能够可靠运行的任务。例如：
 - 向后端服务发送日志或分析数据
 - 定期将应用数据与服务器同步

`WorkManager` 不适用于应用进程结束时能够安全终止的运行中后台工作，也不适用于需要立即执行的任务。

> **注意：**要将 `WorkManager` 库导入您的 Android 项目，请按照 [WorkManager 版本说明](https://developer.android.com/jetpack/androidx/releases/work#declaring_dependencies)中的说明声明依赖项。

## 依赖 ##
要将 `WorkManager` 库引入到 Android 项目中，请将以下依赖项添加到应用或模块的 `build.gradle` 文件中：
```gradle
dependencies {
    def work_version = "2.3.4"

    // (Java only)
    implementation "androidx.work:work-runtime:$work_version"

    // Kotlin + coroutines
    implementation "androidx.work:work-runtime-ktx:$work_version"

    // optional - RxJava2 support
    implementation "androidx.work:work-rxjava2:$work_version"

    // optional - GCMNetworkManager support
    implementation "androidx.work:work-gcm:$work_version"

    // optional - Test helpers
    androidTestImplementation "androidx.work:work-testing:$work_version"
  }
```

## 相关类介绍 ##
**`WorkManager`** 架构组件主要由 `Worker`、`WorkRequest` 和 `WorkManager` 三个最主要的类组成。

### Worker ###
**`Worker`**是个抽象类，用于指定需要执行的具体任务。`Worker` 类的源码如下：
```java
public abstract class Worker extends ListenableWorker {

    // Package-private to avoid synthetic accessor.
    SettableFuture<ListenableWorker.Result> mFuture;

    @Keep
    @SuppressLint("BanKeepAnnotation")
    public Worker(@NonNull Context context, @NonNull WorkerParameters workerParams) {
        super(context, workerParams);
    }

    /**
     * 任务逻辑
     *
     * @return 返回任务的执行结果：成功、失败、稍后重试
     */
    @WorkerThread
    public abstract @NonNull Result doWork();

    /**
     * 异步执行任务，在WorkManager管理的后台线程中执行
     */
    @Override
    public final @NonNull ListenableFuture<Result> startWork() {
        mFuture = SettableFuture.create();
        getBackgroundExecutor().execute(new Runnable() {
            @Override
            public void run() {
                try {
                    Result result = doWork();
                    mFuture.set(result);
                } catch (Throwable throwable) {
                    mFuture.setException(throwable);
                }

            }
        });
        return mFuture;
    }
}
```

`doWork()` 方法实现了具体任务的逻辑，从 `doWork()` 返回的 `Result` 会通知 `WorkManager` 任务执行结果：
 - **`Result.success()：`**任务已成功完成
 - **`Result.failure()：`**任务已永久失败
 - **`Result.retry()：`**任务已暂时性失败，需要稍后重试

> `Worker` 是在 `WorkManager` 提供的后台线程上同步执行任务的类。

### WorkRequest ###
**`WorkRequest`** 代表一个单独的任务，是对 `Worker` 任务的包装，一个 `WorkRequest` 对应一个 `Worker` 类。可以通过 `WorkRequest` 来给 `Worker` 类添加约束细节，比如指定任务应该运行的环境、任务的输入参数、任务只有在有网的情况下执行、任务执行一次还是多次等等。
`WorkRequest` 是一个抽象类，组件里面也给两个相应的子类：`OneTimeWorkRequest`(一次性任务)、`PeriodicWorkRequest`(周期性任务)。`WorkRequest` 类的源码如下：
```java
public abstract class WorkRequest {

    /**
     * 获取 WorkRequest对应的 UUID
     */
    public @NonNull UUID getId() {
        return mId;
    }

    /**
     * 获取 WorkRequest对应的UUID string
     */
    @RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)
    public @NonNull String getStringId() {
        return mId.toString();
    }

    /**
     * 获取WorkRequest对应的WorkSpec(包含了任务的一些详细信息)
     */
    @RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)
    public @NonNull WorkSpec getWorkSpec() {
        return mWorkSpec;
    }

    /**
     * 获取WorkRequest对应的tag
     */
    @RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)
    public @NonNull Set<String> getTags() {
        return mTags;
    }

    public abstract static class Builder<B extends WorkRequest.Builder<?, ?>, W extends WorkRequest> {

        /**
         * 设置任务的退避/重试策略。比如我们在Worker类的doWork()函数返回Result.retry()，让该任务又重新入队
         */
        public final @NonNull
        B setBackoffCriteria(@NonNull BackoffPolicy backoffPolicy, long backoffDelay, @NonNull TimeUnit timeUnit) { }

        /**
         * 设置任务的退避/重试策略。比如我们在Worker类的doWork()函数返回Result.retry()，让该任务又重新入队
         */
        @RequiresApi(26)
        public final @NonNull
        B setBackoffCriteria(@NonNull BackoffPolicy backoffPolicy, @NonNull Duration duration) { }

        /**
         * 设置任务的运行的限制条件，比如有网的时候执行任务，不是低电量的时候执行任务
         */
        public final @NonNull B setConstraints(@NonNull Constraints constraints) { }

        /**
         * 设置任务的输入参数
         */
        public final @NonNull B setInputData(@NonNull Data inputData) { }

        /**
         * 设置任务的tag
         */
        public final @NonNull B addTag(@NonNull String tag) { }

        /**
         * 设置任务结果保存时间
         */
        public final @NonNull B keepResultsForAtLeast(long duration, @NonNull TimeUnit timeUnit) { }

        /**
         * 设置任务结果保存时间
         */
        @RequiresApi(26)
        public final @NonNull B keepResultsForAtLeast(@NonNull Duration duration) { }

        /**
         * 设置任务初始延迟时间
         */
        public @NonNull B setInitialDelay(long duration, @NonNull TimeUnit timeUnit) { }

        /**
         * 设置任务初始延迟时间
         */
        @RequiresApi(26)
        public @NonNull B setInitialDelay(@NonNull Duration duration) { }

    }
}
```

`Data` 用于给 `ListenableWorker` 设置输入参数和输出参数。`Data` 是一个轻量级的容器(不能超过10KB)，`Data` 通过 `key-value` 的形式来保存信息。`Data` 的源码如下：
```java
public final class Data {
    
    /** 没有元素的空Data对象 */
    public static final Data EMPTY = new Data.Builder().build();

    /** Data的最大字节数 */
    @SuppressLint("MinMaxConstant")
    public static final int MAX_DATA_BYTES = 10 * 1024;    // 10KB

    @SuppressWarnings("WeakerAccess") /* synthetic access */
    Map<String, Object> mValues;

    Data() {    // stub required for room
    }

    public Data(@NonNull Data other) {
        mValues = new HashMap<>(other.mValues);
    }

    Data(@NonNull Map<String, ?> values) {
        mValues = new HashMap<>(values);
    }

}
```

`BackoffPolicy` 指定任务的重试策略，有 `LINEAR` 和 `EXPONENTIAL` 两个值。`BackoffPolicy` 的源码如下：
```java
public enum BackoffPolicy {
    /**
     * 每次重试时间指数增加
     */
    EXPONENTIAL,

    /**
     * 每次重试时间线性增加
     */
    LINEAR
}
```

`Constraints` 指定 `WorkRequest` 运行的限制条件，默认情况下 `WorkRequest` 没有任何限制条件。使用 `Constraint.Builder` 来创建 `Constraints`，并在创建 `WorkRequest` 之前把 `Constraints` 传给 `WorkRequest.Builder` 的 `setConstraints()` 方法。`Constraints` 的源码如下：
```java
public final class Constraints {

    /** 没有要求的Constraints对象 */
    public static final Constraints NONE = new Constraints.Builder().build();

    /** Constraints对象的生成器 */
    public static final class Builder {

        /**
         * 设置是否在充电状态下执行任务
         */
        public @NonNull Builder setRequiresCharging(boolean requiresCharging);

        /**
         * 设置是否在设备空闲的时候执行
         */
        @RequiresApi(23)
        public @NonNull Builder setRequiresDeviceIdle(boolean requiresDeviceIdle);

        /**
         * 设置指定网络状态执行任务
         * NetworkType.NOT_REQUIRED：对网络没有要求
         * NetworkType.CONNECTED：网络连接的时候执行
         * NetworkType.UNMETERED：不计费的网络比如WIFI下执行
         * NetworkType.NOT_ROAMING：非漫游网络状态
         * NetworkType.METERED：计费网络比如3G，4G下执行。
         */
        public @NonNull Builder setRequiredNetworkType(@NonNull NetworkType networkType);

        /**
         * 设置是否可以在电量不足时执行任务
         */
        public @NonNull Builder setRequiresBatteryNotLow(boolean requiresBatteryNotLow);

        /**
         * 设置是否可以在存储容量不足时执行
         */
        public @NonNull Builder setRequiresStorageNotLow(boolean requiresStorageNotLow);

        /**
         * 设置当Uri有更新的时候是否执行任务
         */
        @RequiresApi(24)
        public @NonNull Builder addContentUriTrigger(@NonNull Uri uri, boolean triggerForDescendants);

        /**
         * 设置从检测到Uri更改到任务执行所允许的延迟时间
         */
        @RequiresApi(24)
        @NonNull
        public Builder setTriggerContentUpdateDelay(long duration, @NonNull TimeUnit timeUnit);

        /**
         * 设置从检测到Uri更改到任务执行所允许的延迟时间
         */
        @RequiresApi(26)
        @NonNull
        public Builder setTriggerContentUpdateDelay(Duration duration);

        /**
         * 设置从第一次检测到Uri更改到任务执行所允许的最大延迟时间
         */
        @RequiresApi(24)
        @NonNull
        public Builder setTriggerContentMaxDelay(long duration, @NonNull TimeUnit timeUnit);

        /**
         * 设置从第一次检测到Uri更改到任务执行所允许的最大延迟时间
         */
        @RequiresApi(26)
        @NonNull
        public Builder setTriggerContentMaxDelay(Duration duration);

        /**
         * 生成Constraints对象
         */
        public @NonNull Constraints build() {
            return new Constraints(this);
        }
    }
}
```

> **注意：**`PeriodicWorkRequest` 可以指定的最小间隔时间为 15 分钟，最小伸缩时间为 5 分钟。

### WorkManager ###
**`WorkManager`**管理 `WorkRequest` 队列，并根据设备和其他条件选择执行的具体方式。在大部分情况在如果没有给队列加 `Contraints`，`WorkManager` 会立即执行任务。使用时需要把 `WorkRequest` 对象传给 `WorkManager` 以便将任务编入队列，通过 `WorkManager` 来调度任务，以分散系统资源的负载。`WorkManager` 类的源码如下：
```java
public abstract class WorkManager {

    /**
     * 使任务入队以进行后台处理
     */
    @NonNull
    public final Operation enqueue(@NonNull WorkRequest workRequest) {
        return enqueue(Collections.singletonList(workRequest));
    }

    /**
     * 使任务入队以进行后台处理
     */
    @NonNull
    public abstract Operation enqueue(@NonNull List<? extends WorkRequest> requests);

    /**
     * 从一个或多个OneTimeWorkRequest开始链式结构调用
     * 比如有A,B,C三个任务需要顺序执行，那么就可以WorkManager.getInstance().beginWith(A).then(B).then(C).enqueue();
     */
    public final @NonNull WorkContinuation beginWith(@NonNull OneTimeWorkRequest work) {
        return beginWith(Collections.singletonList(work));
    }

    /**
     * 从一个或多个OneTimeWorkRequest开始链式结构调用
     * 比如有A,B,C三个任务需要顺序执行，那么就可以WorkManager.getInstance().beginWith(A).then(B).then(C).enqueue();
     */
    public abstract @NonNull WorkContinuation beginWith(@NonNull List<OneTimeWorkRequest> work);

    /**
     * 创建一个唯一的工作队列，唯一工作队列里面的任务不能重复添加
     */
    public final @NonNull WorkContinuation beginUniqueWork(
            @NonNull String uniqueWorkName,
            @NonNull ExistingWorkPolicy existingWorkPolicy,
            @NonNull OneTimeWorkRequest work) {
        return beginUniqueWork(uniqueWorkName, existingWorkPolicy, Collections.singletonList(work));
    }

    /**
     * 创建一个唯一的工作队列，唯一工作队列里面的任务不能重复添加
     */
    public abstract @NonNull WorkContinuation beginUniqueWork(
            @NonNull String uniqueWorkName,
            @NonNull ExistingWorkPolicy existingWorkPolicy,
            @NonNull List<OneTimeWorkRequest> work);

    /**
     * 将一个OneTimeWorkRequest任务放到唯一的工作序列里面去。
     */
    @NonNull
    public Operation enqueueUniqueWork(
            @NonNull String uniqueWorkName,
            @NonNull ExistingWorkPolicy existingWorkPolicy,
            @NonNull OneTimeWorkRequest work) {
        return enqueueUniqueWork(
                uniqueWorkName,
                existingWorkPolicy,
                Collections.singletonList(work));
    }

    /**
     * 将多个个OneTimeWorkRequest任务放到唯一的工作序列里面去。
     */
    @NonNull
    public abstract Operation enqueueUniqueWork(
            @NonNull String uniqueWorkName,
            @NonNull ExistingWorkPolicy existingWorkPolicy,
            @NonNull List<OneTimeWorkRequest> work);

    /**
     * 将一个PeriodicWorkRequest任务放到唯一的工作序列里面去。
     */
    @NonNull
    public abstract Operation enqueueUniquePeriodicWork(
            @NonNull String uniqueWorkName,
            @NonNull ExistingPeriodicWorkPolicy existingPeriodicWorkPolicy,
            @NonNull PeriodicWorkRequest periodicWork);

    /**
     * 通过UUID取消任务
     */
    public abstract @NonNull Operation cancelWorkById(@NonNull UUID id);

    /**
     * 通过tag取消任务
     */
    public abstract @NonNull Operation cancelAllWorkByTag(@NonNull String tag);

    /**
     * 取消唯一队列里面所有的任务
     */
    public abstract @NonNull Operation cancelUniqueWork(@NonNull String uniqueWorkName);

    /**
     * 取消所有的任务
     */
    public abstract @NonNull Operation cancelAllWork();


    /**
     * 通过UUID获取任务的WorkInfo，利用LiveData变量可以通过注册监听器来观察WorkInfo的变化
     */
    public abstract @NonNull LiveData<WorkInfo> getWorkInfoByIdLiveData(@NonNull UUID id);

    /**
     * 通过UUID获取任务的WorkInfo
     */
    public abstract @NonNull ListenableFuture<WorkInfo> getWorkInfoById(@NonNull UUID id);

    /**
     * 通过tag获取任务的WorkInfo，利用LiveData变量可以通过注册监听器来观察WorkInfo的变化
     */
    public abstract @NonNull LiveData<List<WorkInfo>> getWorkInfosByTagLiveData(@NonNull String tag);

    /**
     * 通过tag获取任务的WorkInfo
     */
    public abstract @NonNull ListenableFuture<List<WorkInfo>> getWorkInfosByTag(@NonNull String tag);

    /**
     * 唯一队列里面所有任务的WorkInfo，利用LiveData变量可以通过注册监听器来观察WorkInfo的变化
     */
    public abstract @NonNull LiveData<List<WorkInfo>> getWorkInfosForUniqueWorkLiveData(@NonNull String uniqueWorkName);

    /**
     * 唯一队列里面所有任务的WorkInfo
     */
    public abstract @NonNull ListenableFuture<List<WorkInfo>> getWorkInfosForUniqueWork(@NonNull String uniqueWorkName);

}
```

`WorkInfo` 包含了 `WorkRequest` 的信息，包含 `WorkRequest` 的 ID、状态，输出，标签和任务进度等信息。`WorkInfo` 的源码如下：
```java
public final class WorkInfo {

    private @NonNull UUID mId;            // 任务ID
    private @NonNull State mState;        // 任务状态
    private @NonNull Data mOutputData;    // 任务输入
    private @NonNull Set<String> mTags;   // 任务标签
    private @NonNull Data mProgress;      // 任务进度
    private int mRunAttemptCount;         // 任务尝试次数

}
```

`WorkManager` 的使用分为几个步骤：
 - 继承 `Worker`，处理任务的具体逻辑。
 - `OneTimeWorkRequest` 或者 `PeriodicWorkRequest` 包装 `Worker`，设置 `Worker` 的一些约束添加或者输入参数。
 - 任务入队执行(如果是多个任务可以形成任务链在入队执行)。
 - 监听任务的输出(`LiveData`的使用)。

## 参考 ##
[Schedule tasks with WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager)

