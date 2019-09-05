---
title: Android Activity 启动流程详解
categories: Android
tags:
  - Android
abbrlink: 6497dc60
date: 2019-05-06 18:30:55
---

## 主要对象介绍 ##
 - **`ActivityManagerService：`**负责系统中所有 `Activity` 的生命周期；
 - **`ActivityThread：`**App 的真正入口，当 App 启动后，会调用其 `main()` 方法开始执行，开启消息循环队列。是传说中的 UI 线程，即主线程。与 `ActivityManagerService` 配合，一起完成 `Activity` 的管理工作；
 - **`ApplicationThread：`**用来实现 `ActivityManagerService` 与 `ActivityThread` 之间的交互。在 `ActivityManagerService` 需要管理相关 `Application` 中的 `Activity` 的生命周期，通过 `ApplicationThread` 的代理对象与 `ActivityThread` 通讯；
 - **`ApplictationThreadProxy：`**是 `ApplicationThread` 在服务端的代理对象，负责和客户端的 `ApplicationThread` 通讯。AMS 就是通过这个代理对象与 `ActivityThread` 进行通信的（Android 8.0 上以删除该类，采用 `AIDL`接口的方式来进行 `IPC`，实现 `RPC` 操作）；
 - **`Instrumentation：`**每一个应用程序都只有一个 `Instrumentation` 对象，每个 `Activity` 内都有一个对该对象的引用。`Instrumentation` 可以理解为应用进程的管家，`ActivityThread` 要创建或者打开某个 `Activity` 时，都需要通过 `Instrumentation` 来进行具体的操作；
 - **`ActvitityStack：`**`Activity` 在 AMS 的栈管理，用来记录已经启动的 `Activity` 的先后关系、状态信息等。通过 `ActivityStack` 决定是否需要启动新的进程；
 - **`ActivityRecord：`**`ActivityStatck` 的管理对象，每个 `Activity` 在AMS对应的一个 `ActivityRecord`，来记录 `Activity` 的状态以及其他信息。可以理解为 `Activity` 在服务端的 `Activity` 对象的映射；
 - **`ActivityClientRecord：`**与ActivityRecord是在服务端（AMS）的记录相对应，是 `Activity` 在客户端（ActivityThread）的记录；
 - **`TaskRecord：`**AMS抽象出来的任务栈的概念。一个 `TaskRecord` 包含若干个 `ActivityRecord`。ASM 用它来确保 `Activity` 启动和退出顺序。它与 `Activity` 的启动模式直接相关。
 - **`ActivityStarter：`**启动 `Activity` 的控制器，主要用于用来将 `Intent` 和 `flags` 转换成 `activity` 和相关任务栈；
 - **`ActivityStackSupervisor：`**主要管理着 `mHomeStack` 和 `mFocusedStack` 两个 `ActivityStack` 等相关信息；

## Binder 通信 ##
 - 在 Android 8.0 以前，Binder 通信的流程如下：
  - 客户端： ActivityManagerProxy -> Binder 驱动  ->  服务端：ActivityManagerService
  - 服务端： ApplicationThreadProxy  -> Binder 驱动  ->  客户端：ApplicationThread

 - Android 8.0 开始，删除了 `ActivityManagerNative`，AMS 的继承类也发生了变化，继承了 `IActivityManager.Stub` 接口类。了解 Android 的 Binder 机制应该知道，这是 `IActivityManager.aidl` 生成的接口类。Android8.0 开始，把一些 Binder 代码转化为了 AIDL 模板方式：
```java
public class ActivityManagerService extends IActivityManager.Stub
        implements Watchdog.Monitor, BatteryStatsImpl.BatteryCallback {
}
```
ActivityManager 中的代码：
```java
    public static IActivityManager getService() {
        return IActivityManagerSingleton.get();
    }

    private static final Singleton<IActivityManager> IActivityManagerSingleton =
            new Singleton<IActivityManager>() {
                @Override
                protected IActivityManager create() {
                    final IBinder b = ServiceManager.getService(Context.ACTIVITY_SERVICE);
                    final IActivityManager am = IActivityManager.Stub.asInterface(b);
                    return am;
                }
            };
```

 - 而在 Android 7.0 及其以前的版本上，则是客户端通过 `ActivityManaagerProxy` 与服务端的 `ActivityManager` 进行通信的。
  - `ActivityManagerNative` 的核心代码如下：
	```java
	static public IActivityManager getDefault() {
	    return gDefault.get();
	}

	static public IActivityManager asInterface(IBinder obj) {
	    if (obj == null) {
		return null;
	    }
	    IActivityManager in = (IActivityManager)obj.queryLocalInterface(descriptor);
	    if (in != null) {
		return in;
	    }
	    return new ActivityManagerProxy(obj);
	}
	```

  - `ActivityManager` 的核心代码如下：
	```java
	private static final Singleton<IActivityManager> gDefault = new Singleton<IActivityManager>() {
		protected IActivityManager create() {
		    IBinder b = ServiceManager.getService("activity");
		    if (false) {
			Log.v("ActivityManager", "default service binder = " + b);
		    }
		    IActivityManager am = asInterface(b);//注意这一行
		    if (false) {
			Log.v("ActivityManager", "default service = " + am);
		    }
		    return am;
		}
	};
	```

 > 可以看出来，两个写法不一样，本质上都是一样的，Android 8.0 可能使用了 AIDL 方式进行 IPC。

 - 同理，`ApplicationThread` 同上述一样做了更改：
```java
private class ApplicationThread extends IApplicationThread.Stub {
    ...
}
```

> 所以在 Android 8.0 上不存在 `ActivityManagerProxy` 和 `ApplicationThreadProxy`，而是采用了 `AIDL` 接口的方式来进行通信的。

## 源码调用栈(基于Android 8.0) ##
按照自上而下的调用栈的顺序进行调用，以类文件为单位来计步，方便大家去了解调用流程，主要涉及的几个源文件：`Instumentation`、`ActivityMangerService`、`ActivityStack`、`ActivityStarter`、`ActivityStackSupervisor`、`ActivityThread`、`Activity` 等。

### 1. Activity ###
```java
    public void startActivity(Intent intent) {
        this.startActivity(intent, null);
    }

    public void startActivity(Intent intent, @Nullable Bundle options) {
        if (options != null) {
            startActivityForResult(intent, -1, options);
        } else {
            startActivityForResult(intent, -1);
        }
    }

    public void startActivityForResult(@RequiresPermission Intent intent, int requestCode) {
        startActivityForResult(intent, requestCode, null);
    }

    public void startActivityForResult(@RequiresPermission Intent intent, int requestCode,
            @Nullable Bundle options) {
        if (mParent == null) {
            options = transferSpringboardActivityOptions(options);
            Instrumentation.ActivityResult ar =
                mInstrumentation.execStartActivity(
                    this, mMainThread.getApplicationThread(), mToken, this,
                    intent, requestCode, options);
            if (ar != null) {
                mMainThread.sendActivityResult(
                    mToken, mEmbeddedID, requestCode, ar.getResultCode(),
                    ar.getResultData());
            }
            if (requestCode >= 0) {
                mStartedActivity = true;
            }

            cancelInputsAndStartExitTransition(options);
        } else {
            if (options != null) {
                mParent.startActivityFromChild(this, intent, requestCode, options);
            } else {
                mParent.startActivityFromChild(this, intent, requestCode);
            }
        }
    }
```

其实 `Activity$startActivity()` 有几个其他重载的方法，但是最终都会执行到 `Activity$startActivityForResult()` 方法。如果是调用 `startActivity()` 启动 `Activity`，那么 `requestCode` 参数则传入 `-1`，表示当前 `Activity` 启动一个新的 `Activity` 后，不需要获取新的 `Activity` 返回来的数据。关于调用 `startActivityForResult()` 方法启动 `Activity` 的情况，就不多介绍了，继续下一步。

### 2. Intrumentation ###
```java
    public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {
        ...
        try {
            intent.migrateExtraStreamToClipData();
            intent.prepareToLeaveProcess(who);
            int result = ActivityManager.getService()
                .startActivity(whoThread, who.getBasePackageName(), intent,
                        intent.resolveTypeIfNeeded(who.getContentResolver()),
                        token, target != null ? target.mEmbeddedID : null,
                        requestCode, 0, null, options);
            checkStartActivityResult(result, intent);
        } catch (RemoteException e) {
            throw new RuntimeException("Failure from system", e);
        }
        return null;
    }
```

### 3. ActivityManagerService ###
```java
    @Override
    public final int startActivity(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, 
            int requestCode, int startFlags, ProfilerInfo profilerInfo, Bundle bOptions) {
        return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo,
                resultWho, requestCode, startFlags, profilerInfo, bOptions,
                UserHandle.getCallingUserId());
    }

    @Override
    public final int startActivityAsUser(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, 
	    int requestCode, int startFlags, ProfilerInfo profilerInfo, Bundle bOptions, int userId) {
        enforceNotIsolatedCaller("startActivity");
        userId = mUserController.handleIncomingUser(Binder.getCallingPid(), Binder.getCallingUid(),
                userId, false, ALLOW_FULL_ONLY, "startActivity", null);
        return mActivityStarter.startActivityMayWait(caller, -1, callingPackage, intent,
                resolvedType, null, null, resultTo, resultWho, requestCode, startFlags,
                profilerInfo, null, null, bOptions, false, userId, null, "startActivityAsUser");
    }
```

### 4. ActivityStarter ###
```java
    final int startActivityMayWait(IApplicationThread caller, int callingUid,
            String callingPackage, Intent intent, String resolvedType,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            IBinder resultTo, String resultWho, int requestCode, int startFlags,
            ProfilerInfo profilerInfo, WaitResult outResult,
            Configuration globalConfig, Bundle bOptions, boolean ignoreTargetSecurity, 
	    int userId, TaskRecord inTask, String reason) {
        synchronized (mService) {
            ...
            int res = startActivityLocked(caller, intent, ephemeralIntent, resolvedType,
                    aInfo, rInfo, voiceSession, voiceInteractor,
                    resultTo, resultWho, requestCode, callingPid,
                    callingUid, callingPackage, realCallingPid, realCallingUid, startFlags,
                    options, ignoreTargetSecurity, componentSpecified, outRecord, inTask,
                    reason);
            ...
            mSupervisor.mActivityMetricsLogger.notifyActivityLaunched(res, outRecord[0]);
            return res;
        }
    }        

    int startActivityLocked(IApplicationThread caller, Intent intent, Intent ephemeralIntent,
            String resolvedType, ActivityInfo aInfo, ResolveInfo rInfo,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            IBinder resultTo, String resultWho, int requestCode, int callingPid, int callingUid,
            String callingPackage, int realCallingPid, int realCallingUid, int startFlags,
            ActivityOptions options, boolean ignoreTargetSecurity, boolean componentSpecified,
            ActivityRecord[] outActivity, TaskRecord inTask, String reason) {
        ...
        mLastStartActivityResult = startActivity(caller, intent, ephemeralIntent, resolvedType,
                aInfo, rInfo, voiceSession, voiceInteractor, resultTo, resultWho, requestCode,
                callingPid, callingUid, callingPackage, realCallingPid, realCallingUid, startFlags,
                options, ignoreTargetSecurity, componentSpecified, mLastStartActivityRecord,
                inTask);
        ...
        return mLastStartActivityResult != START_ABORTED ? mLastStartActivityResult : START_SUCCESS;
    }
    
    private int startActivity(IApplicationThread caller, Intent intent, Intent ephemeralIntent,
            String resolvedType, ActivityInfo aInfo, ResolveInfo rInfo,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            IBinder resultTo, String resultWho, int requestCode, int callingPid, int callingUid,
            String callingPackage, int realCallingPid, int realCallingUid, int startFlags,
            ActivityOptions options, boolean ignoreTargetSecurity, boolean componentSpecified,
            ActivityRecord[] outActivity, TaskRecord inTask) {
        ...
        doPendingActivityLaunchesLocked(false);
        return startActivity(r, sourceRecord, voiceSession, voiceInteractor, startFlags, true,
                options, inTask, outActivity);
    }
            

    private int startActivity(final ActivityRecord r, ActivityRecord sourceRecord,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            int startFlags, boolean doResume, ActivityOptions options, TaskRecord inTask,
            ActivityRecord[] outActivity) {
        ...
        result = startActivityUnchecked(r, sourceRecord, voiceSession, voiceInteractor,
                    startFlags, doResume, options, inTask, outActivity);
        ...
        return result;
    }
    
    private int startActivityUnchecked(final ActivityRecord r, ActivityRecord sourceRecord,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            int startFlags, boolean doResume, ActivityOptions options, TaskRecord inTask,
            ActivityRecord[] outActivity) {
        ...
        if (mDoResume) {
            mSupervisor.resumeFocusedStackTopActivityLocked();
        }
        ...
        return START_SUCCESS;
    }
```

### 5. ActivityStackSupervisor ###
```java
    boolean resumeFocusedStackTopActivityLocked() {
        return resumeFocusedStackTopActivityLocked(null, null, null);
    }

    boolean resumeFocusedStackTopActivityLocked(
            ActivityStack targetStack, ActivityRecord target, ActivityOptions targetOptions) {
        ...
        final ActivityRecord r = mFocusedStack.topRunningActivityLocked();
        if (r == null || r.state != RESUMED) {
            mFocusedStack.resumeTopActivityUncheckedLocked(null, null);
        }
        ...
        return false;
    }
```

### 6. ActivityStack ###
```java
    boolean resumeTopActivityUncheckedLocked(ActivityRecord prev, ActivityOptions options) {
        ...
        result = resumeTopActivityInnerLocked(prev, options);
        ...
        return result;
    }

    private boolean resumeTopActivityInnerLocked(ActivityRecord prev, ActivityOptions options) {
        ...
        if (mResumedActivity != null) {
            // 同步等待pause当前Activity的结果
            pausing |= startPausingLocked(userLeaving, false, next, false);
        }
        ...
        // 开始启动下一个Activity
        mStackSupervisor.startSpecificActivityLocked(next, true, false);
        ...
        return true;
    }

    final boolean startPausingLocked(boolean userLeaving, boolean uiSleeping,
            ActivityRecord resuming, boolean pauseImmediately) {
        ...
        // 去当前Activity所在应用进程暂停当前Activity
        prev.app.thread.schedulePauseActivity(prev.appToken, prev.finishing,
                        userLeaving, prev.configChangeFlags, pauseImmediately);
        ...
    }
```

### 7. ActivityThread$ApplicationThread ###
```java
    public final void schedulePauseActivity(IBinder token, boolean finished, boolean userLeaving, 
					    int configChanges, boolean dontReport) {
        ...
        sendMessage(
                finished ? H.PAUSE_ACTIVITY_FINISHING : H.PAUSE_ACTIVITY,
                token,
                (userLeaving ? USER_LEAVING : 0) | (dontReport ? DONT_REPORT : 0),
                configChanges,
                seq);
    }
```

### 8. ActivityThread ###
```java
    private void sendMessage(int what, Object obj, int arg1, int arg2, int seq) {
        Message msg = Message.obtain();
        ...
        mH.sendMessage(msg);
    }
```

### 9. ActivityThread$H ###
```java
    public void handleMessage(Message msg) {
        switch (msg.what) {
            ...
            case PAUSE_ACTIVITY: {
                SomeArgs args = (SomeArgs) msg.obj;
                handlePauseActivity((IBinder) args.arg1, false,
                        (args.argi1 & USER_LEAVING) != 0, args.argi2,
                        (args.argi1 & DONT_REPORT) != 0, args.argi3);
            }
            break;
            ...
        }
        ...
    }
```

### 10. ActivityThread ###
```java
    private void handlePauseActivity(IBinder token, boolean finished, boolean userLeaving, 
				     int configChanges, boolean dontReport, int seq) {
        ...
        performPauseActivity(token, finished, r.isPreHoneycomb(), "handlePauseActivity");
        ...
        // 执行完后通知AMS当前Activity已经pause
        ActivityManager.getService().activityPaused(token);
        ...
    }

    final Bundle performPauseActivity(ActivityClientRecord r, boolean finished,
                                      boolean saveState, String reason) {
        ...
        performPauseActivityIfNeeded(r, reason);
        ...
    }

    private void performPauseActivityIfNeeded(ActivityClientRecord r, String reason) {
        ...
        mInstrumentation.callActivityOnPause(r.activity);
        ...
    }
```

### 11. Instrumentation ###
```java
    public void callActivityOnPause(Activity activity) {
        activity.performPause();
    }
```

### 12. Activity ###
```java
    final void performPause() {
        ...
        onPause();
        ...
    }
```

由于在 `ActivityThread` 中 `handlePauseActivity` 的方法里，在 `pause` 成功后，需要通知 `AMS` 已经 `pause` 成功，所以接着分析 `ActivityManagerService.activityPaused()` 方法。

### 13. ActivityManagerService ###
```java
    public final void activityPaused(IBinder token) {
        final long origId = Binder.clearCallingIdentity();
        synchronized(this) {
            ActivityStack stack = ActivityRecord.getStackLocked(token);
            if (stack != null) {
                stack.activityPausedLocked(token, false);
            }
        }
        Binder.restoreCallingIdentity(origId);
    }
```

### 14. ActivityStack ###
```java
    final void activityPausedLocked(IBinder token, boolean timeout) {
        ...
        mHandler.removeMessages(PAUSE_TIMEOUT_MSG, r);
        ...
        completePauseLocked(true /* resumeNext */, null /* resumingActivity */);
        ...
    }

    private void completePauseLocked(boolean resumeNext, ActivityRecord resuming) {
        ...
        mStackSupervisor.resumeFocusedStackTopActivityLocked(topStack, prev, null);
        ...
    }
```

### 15. ActivityStackSupervisor ###
```java
    boolean resumeFocusedStackTopActivityLocked(
            ActivityStack targetStack, ActivityRecord target, ActivityOptions targetOptions) {

        // 如果启动Activity和要启动的Activity在同一个ActivityStack中，调用targetStack对象的方法
        if (targetStack != null && isFocusedStack(targetStack)) {
            return targetStack.resumeTopActivityUncheckedLocked(target, targetOptions);
        }

        // 如果不在同一个ActivityStack中，则调用mFocusStack对象的方法
        if (r == null || r.state != RESUMED) {
            mFocusedStack.resumeTopActivityUncheckedLocked(null, null);
        }
        ...

        return false;
    }
```

### 16. ActivityStack ###
```java
    boolean resumeTopActivityUncheckedLocked(ActivityRecord prev, ActivityOptions options) {
        ...
        result = resumeTopActivityInnerLocked(prev, options);
        ...
        return result;
    }

    private boolean resumeTopActivityInnerLocked(ActivityRecord prev, ActivityOptions options) {
        ...
        if (mResumedActivity != null) {
            // 同步等待pause当前Activity的结果,但在pause时，已经把mResumedActivity置为了null，所以不走这里
            pausing |= startPausingLocked(userLeaving, false, next, false);
        }
        ...
        // 开始启动下一个Activity
        mStackSupervisor.startSpecificActivityLocked(next, true, false);
        ...
        return true;
    }
```

### 17. ActivityStackSupervisor ###
```java
    void startSpecificActivityLocked(ActivityRecord r,
            boolean andResume, boolean checkConfig) {
        ...

        if (app != null && app.thread != null) {
            ...
            realStartActivityLocked(r, app, andResume, checkConfig);
            return;
        }
        ...
        // 启动跨进程的Activity需要先开启新的应用进程
        mService.startProcessLocked(r.processName, r.info.applicationInfo, true, 0,
                "activity", r.intent.getComponent(), false, false, true);
    }

    final boolean realStartActivityLocked(ActivityRecord r, ProcessRecord app,
            boolean andResume, boolean checkConfig) throws RemoteException {
        ...
        app.thread.scheduleLaunchActivity(new Intent(r.intent), r.appToken,
                        System.identityHashCode(r), r.info,
                        // TODO: Have this take the merged configuration instead of separate global
                        // and override configs.
                        mergedConfiguration.getGlobalConfiguration(),
                        mergedConfiguration.getOverrideConfiguration(), r.compat,
                        r.launchedFromPackage, task.voiceInteractor, app.repProcState, r.icicle,
                        r.persistentState, results, newIntents, !andResume,
                        mService.isNextTransitionForward(), profilerInfo);
        ...
        return true;
    }
```

### 18. ActivityThread$ApplicationThread ###
```java
    public final void scheduleLaunchActivity(Intent intent, IBinder token, int ident,
            ActivityInfo info, Configuration curConfig, Configuration overrideConfig,
            CompatibilityInfo compatInfo, String referrer, IVoiceInteractor voiceInteractor,
            int procState, Bundle state, PersistableBundle persistentState,
            List<ResultInfo> pendingResults, List<ReferrerIntent> pendingNewIntents,
            boolean notResumed, boolean isForward, ProfilerInfo profilerInfo) {
        ...
        sendMessage(H.LAUNCH_ACTIVITY, r);
    }
```

### 19. ActivityThread ###
```java
    private void sendMessage(int what, Object obj) {
        sendMessage(what, obj, 0, 0, false);
    }

    private void sendMessage(int what, Object obj, int arg1, int arg2, boolean async) {
        ...
        mH.sendMessage(msg);
    }
```

### 20. ActivityThread$H ###
```java
    public void handleMessage(Message msg) {
        ...
        switch (msg.what) {
            case LAUNCH_ACTIVITY: {
                ...
                handleLaunchActivity(r, null, "LAUNCH_ACTIVITY");
                ...
            }
            break;
            ...
        }
        ...
    }
```

### 21. ActivityThread ###
```java
    private void handleLaunchActivity(ActivityClientRecord r, Intent customIntent, String reason) {
        ...
        Activity a = performLaunchActivity(r, customIntent);
        ...
    }

    private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
        ...
        if (r.isPersistable()) {
            mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
        } else {
            mInstrumentation.callActivityOnCreate(activity, r.state);
        }
        ...
        return activity;
    }
```

### 22. ActivityThread ###
```java
    public void callActivityOnCreate(Activity activity, Bundle icicle,
            PersistableBundle persistentState) {
        prePerformCreate(activity);
        activity.performCreate(icicle, persistentState);
        postPerformCreate(activity);
    }
```

### 23. Activity ###
```java
   final void performCreate(Bundle icicle) {
        performCreate(icicle, null);
    }

    final void performCreate(Bundle icicle, PersistableBundle persistentState) {
        ...
        if (persistentState != null) {
            onCreate(icicle, persistentState);
        } else {
            onCreate(icicle);
        }
        ...
    }
```

## Activity 启动流程 ##
为了叙述方便，我们假设 Activity A 要启动 Activity B。
1. Activity A 向 AMS 发送一个启动 Activity B 的进程间通信请求；
2. AMS 会将要启动的 Activity B 的组件信息保存下来，然后通过 Binder 通信（ApplicationThread 及其接口定义语言），让 Activity A 执行 pause 操作；
3. Activity B 完成 pause 操作后，通过 Binder 通信（ActivityManagerService及其接口定义语言）通知 AMS，可以执行启动 Activity B 的操作了（要启动的activity信息保存在了栈顶）；
4. 在启动之前，如果发现 Activity B 的应用程序进程不存在，会先启动一个新的进程（上述调用栈没涉及，同学们可自行查看源码）；
5. AMS 执行一系列启动 Activity B 的操作，并通过 Binder 通信（ApplicationThread 及其接口定义语言）进行跨进程调用，将 Activity B 启动起来；

## Zygote 进程 ##
在 Android 系统里面，`zygote` 是一个进程的名字。Android 是基于 Linux 系统的，当你的手机开机的时候，Linux 的内核加载完成之后就会启动一个叫 `init` 的进程。在 Linux 系统里面，所有的进程都是由 `init` 进程 `fork` 出来的，`zygote` 进程也不例外。

我们都知道，每一个 App 其实都是
 - 一个单独的 dalvik 虚拟机
 - 一个单独的进程

所以当系统里面的第一个 `zygote` 进程运行之后，在这之后再开启 App，就相当于开启一个新的进程。而为了实现资源共用和更快的启动速度，Android 系统开启新进程的方式，是通过 `fork` 第一个 `zygote` 进程实现的。所以说，除了第一个 `zygote` 进程，其他应用所在的进程都是 `zygote` 的子进程。

## SystemServer 进程 ##
`SystemServer` 也是一个进程，而且是由 `zygote` 进程 `fork` 出来的，这个进程是 Android Framework 里面两大非常重要的进程之一——另外一个进程就是上面的 `zygote` 进程。

为什么说 `SystemServer` 非常重要呢？因为系统里面重要的服务都是在这个进程里面开启的，比如 `ActivityManagerService`、`PackageManagerService`、`WindowManagerService` 等等，看着是不是都挺眼熟的？

那么这些系统服务是怎么开启起来的呢？

在 `zygote` 开启的时候，会调用 `ZygoteInit.main()` 进行初始化：
```java
public class ZygoteInit {
    private static final String TAG = "Zygote";

    public static void main(String argv[]) {
        ZygoteServer zygoteServer = new ZygoteServer();
        ...
        try {
            ...
            // 在加载首个zygote的时候，会传入初始化参数，使得startSystemServer = true
            boolean startSystemServer = false;
            String socketName = "zygote";
            String abiList = null;
            boolean enableLazyPreload = false;
            for (int i = 1; i < argv.length; i++) {
                if ("start-system-server".equals(argv[i])) {
                    startSystemServer = true;
                } else if ("--enable-lazy-preload".equals(argv[i])) {
                    enableLazyPreload = true;
                } else if (argv[i].startsWith(ABI_LIST_ARG)) {
                    abiList = argv[i].substring(ABI_LIST_ARG.length());
                } else if (argv[i].startsWith(SOCKET_NAME_ARG)) {
                    socketName = argv[i].substring(SOCKET_NAME_ARG.length());
                } else {
                    throw new RuntimeException("Unknown command line argument: " + argv[i]);
                }
            }
            ...
            // 开始fork SystemServer进程
            if (startSystemServer) {
                Runnable r = forkSystemServer(abiList, socketName, zygoteServer);
                ...
            }
        } catch (Throwable ex) {
            throw ex;
        } finally {
            zygoteServer.closeServerSocket();
        }
        ...
    }

    /**
     * 准备参数并 fork SystemServer
     */
    private static Runnable forkSystemServer(String abiList, String socketName,
                                             ZygoteServer zygoteServer) {
        ...
        int pid;
        try {
            parsedArgs = new ZygoteConnection.Arguments(args);
            ZygoteConnection.applyDebuggerSystemProperty(parsedArgs);
            ZygoteConnection.applyInvokeWithSystemProperty(parsedArgs);

            // fork SystemServer
            pid = Zygote.forkSystemServer(
                    parsedArgs.uid, parsedArgs.gid,
                    parsedArgs.gids,
                    parsedArgs.debugFlags,
                    null,
                    parsedArgs.permittedCapabilities,
                    parsedArgs.effectiveCapabilities);
        } catch (IllegalArgumentException ex) {
            throw new RuntimeException(ex);
        }
        ...
        return null;
    }

}
```

## ActivityManagerService ##
`ActivityManagerService`简称 `AMS`，服务端对象，负责系统中所有 `Activity` 的生命周期。

`ActivityManagerService` 进行初始化的时机很明确，就是在 `SystemServer` 进程开启的时候，就会初始化 `ActivityManagerService`：
```java
public final class SystemServer {
    private static final String TAG = "SystemServer";

    /**
     * The main entry point from zygote.
     * SystemServer进程是有Zygote进程fork出来的
     */
    public static void main(String[] args) {
        new SystemServer().run();
    }

    public SystemServer() {
        ...
    }

    private void run() {
        ...

        // 加载本地系统服务库，并进行初始化 
        System.loadLibrary("android_servers");

        // 创建系统上下文
        createSystemContext();

        // 初始化SystemServiceManager对象
        // 下面的系统服务开启都需要调用SystemServiceManager.startService(Class<T>)方法通过反射来启动
        mSystemServiceManager = new SystemServiceManager(mSystemContext);

        // 开启服务
        try {
            startBootstrapServices();
            startCoreServices();
            startOtherServices();
        } catch (Throwable ex) {
            throw ex;
        }
        ...
    }

    /**
     * 初始化系统上下文对象mSystemContext，并设置默认的主题,mSystemContext实际上是一个ContextImpl对象
     * 调用ActivityThread.systemMain()的时候，会调用ActivityThread.attach(true)，而在attach()里面，则创建了Application对象，并调用了Application.onCreate()。
     */
    private void createSystemContext() {
        ActivityThread activityThread = ActivityThread.systemMain();
        mSystemContext = activityThread.getSystemContext();
        mSystemContext.setTheme(DEFAULT_SYSTEM_THEME);
        ...
    }

    /**
     * 在这里开启了几个核心的服务，因为这些服务之间相互依赖，所以都放在了这个方法里面
     */
    private void startBootstrapServices() {
        ...
        // 初始化ActivityManagerService
        mActivityManagerService = mSystemServiceManager.startService(
                ActivityManagerService.Lifecycle.class).getService();
        mActivityManagerService.setSystemServiceManager(mSystemServiceManager);

        // 初始化PowerManagerService，因为其他服务需要依赖这个Service，因此需要尽快的初始化
        mPowerManagerService = mSystemServiceManager.startService(PowerManagerService.class);

        // 现在电源管理已经开启，ActivityManagerService负责电源管理功能
        mActivityManagerService.initPowerManagement();

        ...

        // 初始化DisplayManagerService
        mDisplayManagerService = mSystemServiceManager.startService(DisplayManagerService.class);

        ...

        // 初始化PackageManagerService
        mPackageManagerService = PackageManagerService.main(mSystemContext, installer,
                mFactoryTestMode != FactoryTest.FACTORY_TEST_OFF, mOnlyCore);
        
        ...
    }

}
```

经过上面这些步骤，我们的 `ActivityManagerService` 对象已经创建好了，并且完成了成员变量初始化。而且在这之前，调用 `createSystemContext()` 创建系统上下文的时候，也已经完成了 `mSystemContext` 和 `ActivityThread` 的创建。注意，这是系统进程开启时的流程，在这之后，会开启系统的 `Launcher` 程序，完成系统界面的加载与显示。

## AMS与APP进程的通信 ##
1. APP 进程通过 `IActivityManager.aidl` 接口向 `AMS` 进程进行通信。
`ActivityManager.getService()` 获得 `AMS` 的 `Binder` 接口，再通过 `Stub.asInterface` 的方式，转成 `IActivityManager` 的接口，通过 `IActivityManager` 与 `AMS` 进行通信，实现 `RPC` 远程跨进程调用。
 - Instrumentation
	```java
	int result = ActivityManager.getService()
			.startActivity(whoThread, who.getBasePackageName(), intent,
				intent.resolveTypeIfNeeded(who.getContentResolver()),
				token, target, requestCode, 0, null, options);
	```
 - ActivityManager
	```java
	public static IActivityManager getService() {
		return IActivityManagerSingleton.get();
	}
	 
	private static final Singleton<IActivityManager> IActivityManagerSingleton =
	    new Singleton<IActivityManager>() {
		@Override
		protected IActivityManager create() {
		    final IBinder b = ServiceManager.getService(Context.ACTIVITY_SERVICE);
		    final IActivityManager am = IActivityManager.Stub.asInterface(b);
		    return am;
		}
	};
	```

2. `AMS` 进程通过 `IApplication.aidl` 接口向 APP 进程进行通信。
在 `AMS` 内部持有每个 `ActivityThread` 的 `IApplicatinThread` 接口实例，用时可以直接调用。同 `IActivityManager`，也是通过 `Binder` 进行进程间通信。

## Launcher ##
`Launcher` 本质上也是一个应用程序，和我们的 App 一样，也是继承自 `Activity`，它的作用用来显示和管理手机上其他 App。当我们点击手机桌面上的图标的时候，App 就由 `Launcher` 开始启动了。

packages/apps/Launcher3/src/com/android/launcher3/Launcher.java：
```java
public class Launcher extends Activity implements LauncherExterns, View.OnClickListener, OnLongClickListener,
                   LauncherModel.Callbacks, View.OnTouchListener, LauncherProviderChangeListener,
                   AccessibilityManager.AccessibilityStateChangeListener {
}
```

`Launcher` 实现了点击、长按等回调接口，来接收用户的输入。既然是普通的 App，那么我们的开发经验在这里就仍然适用，比如，我们点击图标的时候，是怎么开启的应用呢？如果让你，你怎么做这个功能呢？捕捉图标点击事件，然后 `startActivity()` 发送对应的 `Intent` 请求。

## 致谢 ##
[Activity启动流程](https://juejin.im/entry/5abdcdea51882555784e114d)
[Activity 启动过程全解析](https://blog.csdn.net/zhaokaiqiang1992/article/details/49428287)
[Android Activity启动流程](https://www.jianshu.com/p/6719238ae5f0)

