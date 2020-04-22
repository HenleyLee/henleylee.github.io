---
title: Android 中的 ANR 原理分析及解决办法
categories: Android
tags:
  - Android
abbrlink: d8e5ccc6
date: 2019-07-20 13:56:50
---


有过 Android 开发经历的人都不会对 ANR 陌生，它和崩溃一样是程序设计的问题。本文将以较为深入的视角来介绍什么是 ANR，出现场景，如何避免以及如何定位分析 ANR，希望可以帮助大家在编写程序时有所帮助。

## ANR 简介 ##
**`ANR`** 是 `Android` 中一个独有的概念，它的全称是 **`Application Not Responding`**，意思就是应用程序无响应。

`ANR` 是 `Android` 的一种自我保护措施，当主线程出现卡顿时候，`Android` 系统会给用户一个弹窗提示，让用户手动选择继续等待还是强制关闭当前程序。

`ANR` 的直观体验是用户在操作 APP 的过程中，感觉界面卡顿，比如按下某个按钮，打开某个页面等，当卡顿超过一定时间(一般是5秒)时就会出现 `ANR` 对话框。

对于高质量的代码，`ANR` 在开发者自测过程中可能不会经常遇到，但一旦测试人员进行 `Monkey` 测试，`ANR` 出现的概率就比较高了。`ANR` 对于一个应用来说是不能承受之通，其影响并不比应用发生 `Crash` 小，如何快速分析定位并解决，是开发者的必修课。

## ANR 原因 ##
在 `Android` 中，应用程序响应由 `ActivityManagerService(简称AMS)` 和 `WindowManagerService(简称WMS)` 系统服务进行监控。如果应用程序在特定时间无法响应屏幕触摸或键盘输入事件，或者特定事件没有处理完毕，就会出现 `ANR`。

只有当应用程序的 `UI` 线程响应超时才会引起 `ANR`，超时产生原因一般有两种：
1. 当前的事件没有机会得到处理，例如 UI 线程正在响应另外一个事件，当前事件由于某种原因被阻塞了。
2. 当前的事件正在处理，但是由于耗时太长没能及时完成。

## ANR 类型 ##
根据 `ANR`产生的原因不同，超时时间也不尽相同，从本质上讲，产生 `ANR` 的原因有四种，大致可以对应到 `Android` 中四大组件：
 - **`InputDispatching Timeout：`**最常见的一种类型，原因是 View 的按键事件或者触摸事件在特定的时间(5秒)内无法得到响应；
 - **`BroadcastQueue Timeout：`**原因是 BroadcastReceiver 的 onReceiver() 方法运行在主线程中，在特定的时间(前台为10秒，后台为60秒)内无法完成处理；
 - **`Service Timeout：`**比较少出现的一种类型，原因是 Service 的各个生命周期函数在特定时间(前台服务为20秒，后台服务为200秒)内无法完成处理；
 - **`ContentProvider Timeout：`**ContentProvider 的操作在特定的时间(20s)内无法完成处理。

## ANR 分析 ##
为了具体分析 ANR 的产生，我们可以在 `Activity` 的 `onCreate()` 方法通过调用 `SystemClock.sleep()` 方法让主线程休眠，制造一个 ANR，然后具体分析。
```java
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_anr_test);
    // 这是Android提供线程休眠函数，与Thread.sleep()最大的区别是
    // 该使用该函数不会抛出InterruptedException异常。
    SystemClock.sleep(20 * 1000);
}
```

> 在 Android **`开发者选项—>高级—>显示所有“应用程序无响应”(ANR)`**勾选即可对后台 ANR 也进行弹窗显示，方便查看了解程序运行情况。

### Logcat日志信息 ###
产生 `ANR` 后，立即查看 Log，可以看到 `logcat` 清晰地记录了 `ANR` 发生的时间，以及线程的 `tid` 和一句话概括原因：`WaitingInMainSignalCatcherLoop`，大概意思为主线程等待异常。
最后一句 `The application may be doing too much work on its main thread.`告知可能在主线程做了太多的工作。

### Traces.txt日志信息 ###
刚才的 log 有第二句 `Wrote stack traces to '/data/anr/traces.txt'`，说明 `ANR` 异常已经输出到 `traces.txt` 文件，可以通过终端 Termianl 中执行 `adb pull`命令从手机的内部存储中拷贝到电脑中查看即可，最新的 `ANR` 信息在最开始部分。

一般来说 `traces.txt` 文件记录的东西会比较多，分析的时候需要有针对性地去找相关记录。
```shell
----- pid 23346 at 2019-07-08 11:33:57 -----
Cmd line: com.android.anrtest
Build fingerprint: 'google/marlin/marlin:8.0.0/OPR3.170623.007/4286350:user/release-keys'
ABI: 'arm64'
Build type: optimized
Zygote loaded classes=4681 post zygote classes=106
Intern table: 42675 strong; 137 weak
JNI: CheckJNI is on; globals=526 (plus 22 weak)
Libraries: /system/lib64/libandroid.so /system/lib64/libcompiler_rt.so 
/system/lib64/libjavacrypto.so
/system/lib64/libjnigraphics.so /system/lib64/libmedia_jni.so /system/lib64/libsoundpool.so
/system/lib64/libwebviewchromium_loader.so libjavacore.so libopenjdk.so (9)
Heap: 22% free, 1478KB/1896KB; 21881 objects

...

"main" prio=5 tid=1 Sleeping
  | group="main" sCount=1 dsCount=0 flags=1 obj=0x733d0670 self=0x74a4abea00
  | sysTid=23346 nice=-10 cgrp=default sched=0/0 handle=0x74a91ab9b0
  | state=S schedstat=( 391462128 82838177 354 ) utm=33 stm=4 core=3 HZ=100
  | stack=0x7fe6fac000-0x7fe6fae000 stackSize=8MB
  | held mutexes=
  at java.lang.Thread.sleep(Native method)
  - sleeping on <0x053fd2c2> (a java.lang.Object)
  at java.lang.Thread.sleep(Thread.java:373)
  - locked <0x053fd2c2> (a java.lang.Object)
  at java.lang.Thread.sleep(Thread.java:314)
  at android.os.SystemClock.sleep(SystemClock.java:122)
  at com.android.anrtest.ANRTestActivity.onCreate(ANRTestActivity.java:20)
  at android.app.Activity.performCreate(Activity.java:6975)
  at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1213)
  at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2770)
  at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2892)
  at android.app.ActivityThread.-wrap11(ActivityThread.java:-1)
  at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1593)
  at android.os.Handler.dispatchMessage(Handler.java:105)
  at android.os.Looper.loop(Looper.java:164)
  at android.app.ActivityThread.main(ActivityThread.java:6541)
  at java.lang.reflect.Method.invoke(Native method)
  at com.android.internal.os.Zygote$MethodAndArgsCaller.run(Zygote.java:240)
  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:767)
```

在文件中使用 Ctrl + F 查找包名可以快速定位相关代码。
通过上方 log 可以看出相关问题：
 - 进程id和包名：pid 23346 com.android.anrtest
 - 造成ANR的原因：Sleeping
 - 造成ANR的具体行数：ANRTestActivity.java:20 类的第20行

如果是线上版本引起的，Google Play后台有相关的数据可以帮助查看分析并解决问题。

## ANR 的避免 ##
造成 `ANR` 的原因还有很多，下面简单介绍几种常见的 `ANR` 的解决办法：
 - 主线程阻塞或主线程数据读取
> 解决办法：避免死锁的出现，使用子线程来处理耗时操作或阻塞任务。尽量避免在主线程 query provider、不要滥用 SharedPreferences
 - CPU 满负荷，I/O 阻塞
> 解决办法：文件读写或数据库操作放在子线程异步操作。
 - 内存不足
> 解决办法：AndroidManifest.xml 文件<application>中可以设置 android:largeHeap="true"，以此增大 APP 使用内存。不过不建议使用此法，从根本上防止内存泄漏，优化内存使用才是正道。
 - 各大组件 ANR
> 各大组件生命周期中也应避免耗时操作，注意 Activity、BroadcastReciever、Service 和 ContentProvider 也不要执行耗时的任务。

## ANR 的检测 ##
为了避免在开发中引入可能导致应用发生 `ANR` 的问题，除了切记不要在主线程中作耗时操作，也可以借助于一些工具来进行检测，从而更有效的避免 `ANR` 的引入。

### StrictMode ###
**`StrictMode(严格模式)`**是 Android SDK 提供的一个用来检测代码中是否存在违规操作的工具类，`StrictMode` 主要检测两大类问题：
 - 线程策略：ThreadPolicy：
  - detectNetwork：检测是否存在网络操作
  - detectDiskReads：检测是否存在磁盘读取操作
  - detectDiskWrites：检测是否存在磁盘写入操作
  - detectCustomSlowCalls：检测自定义耗时操作
 - 虚拟机策略：VmPolicy
  - detectActivityLeaks：检测是否存在 Activity 泄漏
  - setClassInstanceLimit：检测类实例个数是否超过限制
  - detectLeakedSqliteObjects：检测是否存在 Sqlite 对象泄漏
  - detectLeakedClosableObjects：检测是否存在未关闭的 Closable 对象泄漏

### BlockCanary ###
**`BlockCanary`**是一个轻量的、非侵入式的性能监控函数库，它的用法和 `LeakCanary` 类似，只不过后者监控应用的内存泄漏，而 `BlockCanary` 主要用来监控应用主线程的卡顿，并可通过组件提供的各种信息分析出原因并进行修复。它的原理是利用主线程的消息队列处理机制，通过对比消息分发开始和结束的时间点来判断是否超过设定的时间，如果是，则判断为主线程卡顿。

## 参考 ##
[Keeping your app responsive](https://developer.android.com/training/articles/perf-anr.html)

