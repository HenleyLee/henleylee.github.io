---
title: Android 后台服务 JobIntentService 使用详解
categories: Android
tags:
  - Android
abbrlink: 7418a898
date: 2019-06-06 18:25:35
---

Android 8.0(API 26) 对系统资源的管控更加严格，添加了后台限制规则。

如果满足以下任意条件，应用将被视为处于前台：
 - 具有可见 Activity(不管该 Activity 已启动还是已暂停)；
 - 具有前台服务；
 - 另一个前台应用已关联到该应用(不管是通过绑定到其中一个服务，还是通过使用其中一个内容提供程序)；
 - IME；
 - 壁纸服务；
 - 通知侦听器；
 - 语音或文本服务。

如果以上条件均不满足，应用将被视为处于后台。

Android 8.0(API 26) 以后系统不允许后台应用创建后台服务。因此，Android 8.0 引入了一种全新的方法，即 `Context.startForegroundService()`，以在前台启动新服务。
在系统创建服务后应用有五秒的时间来调用该服务的 `startForeground()` 方法以显示新服务的用户可见通知。如果应用在此时间限制内未调用 `startForeground()` 方法，则系统将停止服务并声明此应用为 `ANR`。

特殊情况是可以创建后台服务的：
1. 处理对用户可见的任务时，应用将被置于白名单中，后台应用将被置于一个临时白名单中并持续数分钟。位于白名单中时，应用可以无限制地启动服务，并且其后台服务也可以运行。
  - 处理一条高优先级 Firebase 云消息传递 (FCM) 消息；
  - 接收广播，例如短信/彩信消息；
  - 从通知执行 PendingIntent。
2. bindService() 方法不受后台限制。

> 由于 Android 8.0(API 26) 以后不能使用后台服务，使用 `Service` 需要使用 `ContextCompat.startForegroundService()` 启动前台服务，而且通知栏有 `Notification` 显示该 `Service` 正在运行，这可能会带来不好的用户体验。如果还是希望使用服务在后台默默工作、通过使用服务开启子进程等等，可以使用 `JobIntentService`。

## 简介 ##
**`JobIntentService`**实质为 `Service`，其继承关系如下所示：
```java
 java.lang.Object
    ↳   android.content.Context
       ↳    android.content.ContextWrapper
           ↳    android.app.Service
               ↳    androidx.core.app.JobIntentService
```

`JobIntentService` 在官方文档解释如下：
```java
Helper for processing work that has been enqueued for a job/service. When running on Android O or later, the work will be dispatched as a job via JobScheduler.enqueue. When running on older versions of the platform, it will use Context.startService.
```
中文解释为：`JobIntentService` 用于处理被加入到队列的 job/service 任务的一个辅助工具。当运行在 Android O 或更高版本时，任务将作为 `job` 通过 `JobScheduler.enqueue()` 进行分发；当运行在较老版本的平台上时，任务仍旧会使用 `Context.startService()` 执行。

## 源码分析 ##
Android 8.0 及以上版本 `JobIntentService` 和 `JobService` 做的事情是相同的，都是等着 `JobScheduler` 分配任务来执行。

不同点在于，`JobService` 使用的 `handler` 使用的是主线程的 `Looper`，因此需要在 `onStartJob()` 方法中手动创建 `AsyncTask` 去执行耗时任务，而 `JobIntentService` 则帮我们处理这一过程，使用它只需要写需要做的任务逻辑即可，不用担心阻塞主线程的问题。另外，向 `JobScheduler` 传递任务操作也更简单了，不需要在指定 `JobInfo` 中的参数，直接 `enqueueWork()` 方法就可以了。这有点像 `Service` 和 `IntentService` 的关系。

下面来简单分析一下 `JobIntentService` 的源码：
```java
public static void enqueueWork(@NonNull Context context, @NonNull Class cls, int jobId,
                                   @NonNull Intent work) {
    enqueueWork(context, new ComponentName(context, cls), jobId, work);
}

public static void enqueueWork(@NonNull Context context, @NonNull ComponentName component,
                                   int jobId, @NonNull Intent work) {
    if (work == null) {
        throw new IllegalArgumentException("work must not be null");
    }
    synchronized (sLock) {
        // 根据版本获取不同的WorkEnqueuer
        WorkEnqueuer we = getWorkEnqueuer(context, component, true, jobId);
        // 每个JobIntentService对应一个唯一JobId，所有给这个Service的work都必须相同
        we.ensureJobId(jobId);
        // 将work加入队列中
        we.enqueueWork(work);
    }
}

static WorkEnqueuer getWorkEnqueuer(Context context, ComponentName cn, boolean hasJobId,
                                        int jobId) {
    WorkEnqueuer we = sClassWorkEnqueuer.get(cn);
    if (we == null) {
        if (Build.VERSION.SDK_INT >= 26) {
            if (!hasJobId) {
                throw new IllegalArgumentException("Can't be here without a job id");
            }
            // Android 8.0及以上版本
            we = new JobWorkEnqueuer(context, cn, jobId);
        } else {
            // Android 8.0以下版本
            we = new CompatWorkEnqueuer(context, cn);
        }
        sClassWorkEnqueuer.put(cn, we);
    }
    return we;
}
```

```java
/**
 * Android 8.0及以上版本
 */
static final class JobWorkEnqueuer extends JobIntentService.WorkEnqueuer {
        private final JobInfo mJobInfo;
        private final JobScheduler mJobScheduler;

        JobWorkEnqueuer(Context context, ComponentName cn, int jobId) {
            super(cn);
            ensureJobId(jobId);
            // jobId和mComponentName来自于JobIntentService的enqueueWork()方法
	    JobInfo.Builder b = new JobInfo.Builder(jobId, mComponentName);
            mJobInfo = b.setOverrideDeadline(0).build();
            mJobScheduler = (JobScheduler) context.getApplicationContext().getSystemService(
                    Context.JOB_SCHEDULER_SERVICE);
        }

        @Override
        void enqueueWork(Intent work) {
            if (DEBUG) Log.d(TAG, "Enqueueing work: " + work);
            // 调用JobScheduler.enqueue()方法
            mJobScheduler.enqueue(mJobInfo, new JobWorkItem(work));
        }
}
```

```java
/**
 * Android 8.0以下版本
 */
static final class CompatWorkEnqueuer extends WorkEnqueuer {
    private final Context mContext;
    private final PowerManager.WakeLock mLaunchWakeLock;
    private final PowerManager.WakeLock mRunWakeLock;
    boolean mLaunchingService;
    boolean mServiceProcessing;

    CompatWorkEnqueuer(Context context, ComponentName cn) {
        super(cn);
        mContext = context.getApplicationContext();
        // Make wake locks.  We need two, because the launch wake lock wants to have
        // a timeout, and the system does not do the right thing if you mix timeout and
        // non timeout (or even changing the timeout duration) in one wake lock.
        // 使用指定的级别和标志创建一个新的唤醒锁
        PowerManager pm = ((PowerManager) context.getSystemService(Context.POWER_SERVICE));
        mLaunchWakeLock = pm.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK,
                cn.getClassName() + ":launch");
        mLaunchWakeLock.setReferenceCounted(false);
        mRunWakeLock = pm.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK,
                cn.getClassName() + ":run");
        mRunWakeLock.setReferenceCounted(false);
    }

    @Override
    void enqueueWork(Intent work) {
        Intent intent = new Intent(work);
        // mComponentName来自于JobIntentService的enqueueWork()方法
        intent.setComponent(mComponentName);
        if (DEBUG) Log.d(TAG, "Starting service for work: " + work);
        // 执行Context的startService()方法
        if (mContext.startService(intent) != null) {
            synchronized (this) {
                if (!mLaunchingService) {
                    mLaunchingService = true;
                    if (!mServiceProcessing) {
                        // If the service is not already holding the wake lock for
                        // itself, acquire it now to keep the system running until
                        // we get this work dispatched.  We use a timeout here to
                        // protect against whatever problem may cause it to not get
                        // the work.
                        mLaunchWakeLock.acquire(60 * 1000);
                    }
                }
            }
        }
    }

    @Override
    public void serviceStartReceived() {
        synchronized (this) {
            // Once we have started processing work, we can count whatever last
            // enqueueWork() that happened as handled.
            mLaunchingService = false;
        }
    }

    @Override
    public void serviceProcessingStarted() {
        synchronized (this) {
            // We hold the wake lock as long as the service is processing commands.
            if (!mServiceProcessing) {
                mServiceProcessing = true;
                // Keep the device awake, but only for at most 10 minutes at a time
                // (Similar to JobScheduler.)
                mRunWakeLock.acquire(10 * 60 * 1000L);
                mLaunchWakeLock.release();
            }
        }
    }

    @Override
    public void serviceProcessingFinished() {
        synchronized (this) {
            if (mServiceProcessing) {
                // If we are transitioning back to a wakelock with a timeout, do the same
                // as if we had enqueued work without the service running.
                if (mLaunchingService) {
                    mLaunchWakeLock.acquire(60 * 1000);
                }
                mServiceProcessing = false;
                mRunWakeLock.release();
            }
        }
    }
}
```

## 案例演示 ##
下面是 `JobIntentService` 的一个实现示例：
```java
import android.content.Context;
import android.content.Intent;
import android.os.Handler;
import android.os.SystemClock;
import android.util.Log;
import android.widget.Toast;

import androidx.core.app.JobIntentService;

/**
 * Example implementation of a JobIntentService.
 */
public class SimpleJobIntentService extends JobIntentService {
    /**
     * Unique job ID for this service.
     */
    static final int JOB_ID = 1000;

    /**
     * Convenience method for enqueuing work in to this service.
     */
    static void enqueueWork(Context context, Intent work) {
        enqueueWork(context, SimpleJobIntentService.class, JOB_ID, work);
    }

    @Override
    protected void onHandleWork(Intent intent) {
        // We have received work to do.  The system or framework is already
        // holding a wake lock for us at this point, so we can just go.
        Log.i("SimpleJobIntentService", "Executing work: " + intent);
        String label = intent.getStringExtra("label");
        if (label == null) {
            label = intent.toString();
        }
        toast("Executing: " + label);
        for (int i = 0; i < 5; i++) {
            Log.i("SimpleJobIntentService", "Running service " + (i + 1) + "/5 @ " 
                    + SystemClock.elapsedRealtime());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
            }
        }
        Log.i("SimpleJobIntentService", "Completed service @ " + SystemClock.elapsedRealtime());
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        toast("All work complete");
    }

    final Handler mHandler = new Handler();

    // Helper for showing tests
    void toast(final CharSequence text) {
        mHandler.post(new Runnable() {
            @Override public void run() {
                Toast.makeText(SimpleJobIntentService.this, text, Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

使用 `JobIntentService` 时需要注意以下几点：
1. 当运行在 Android O 上时，`JobScheduler` 将为任务处理唤醒锁(从排队等待任务开始，直到任务被分派，以及在运行时持有唤醒锁)；当运行在以前版本的平台上时，这个唤醒锁处理在类中通过直接调用 `PowerManager` 来模拟，因此应用程序必须请求 `android.Manifest.permission.WAKE_LOCK` 权限：
```xml
<uses-permission android:name="android.permission.WAKE_LOCK" />
```

2. `JobIntentService` 本质上也是一个 `Service`，因此需要在 `AndroidManifest.xml` 声明，以便系统与之交互：
```xml
<service android:name="SimpleJobIntentService"
        android:permission="android.permission.BIND_JOB_SERVICE" >
        ...
</service>
```

3. `JobIntentService` 具体使用起来非常简单，它已经封装了大量的内部逻辑，只需要调用 `enqueueWork()` 方法就可以了：
```java
Intent intent = new Intent();
SimpleJobIntentService.enqueueWork(context, intent);
```

## 总结 ##
由于 Android O 的后台限制，创建后台服务需要使用 `JobScheduler` 来由系统进行调度任务的执行，而使用 `JobService` 的方式比较繁琐，Android 8.0 或更高版本时提供了 `JobIntentService` 帮助开发者更方便的将任务交给 `JobScheduler` 调度，其本质是 `Service` 后台任务在它的 `onHandleWork()` 方法中进行，子类重写该方法即可，使用较简单。

> **`注意：`**
>  - `JobIntentService` 处理了亮屏/锁屏，因此需要添加 `android.Manifest.permission.WAKE_LOCK` 权限；
>  - `JobIntentService` 本质上也是一个 `Service`，因此需要在 `AndroidManifest.xml` 声明，以便系统与之交互；
>  - `JobIntentService` 使用 `enqueueWork()` 来为分发和处理的新任务，它将在 `onHandleWork()` 方法中执行，该方法在工作线程中执行。

## 参考 ##
[Google 官方文档](https://developer.android.com/reference/androidx/core/app/JobIntentService.html)

