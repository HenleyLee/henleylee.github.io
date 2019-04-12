---
title: Android 性能优化之启动优化
categories: Android
tags:
  - Android
  - 性能优化
abbrlink: 985e20fe
date: 2019-01-21 18:46:35
---

用户希望应用程序能够快速响应并加载。启动时间较慢的应用程序无法满足此预期，并且可能会令用户失望。这种糟糕的体验可能会导致用户在应用商店中对您的应用评分不佳，甚至卸载此应用。

因此想要优化应用程序的启动时间，需要以下几个步骤。首先，我们需要了解应用启动的内部原理。接下来，我们会讨论如何分析启动性能。最后，最后我们会介绍一些影响启动性能的常见问题，并提供一些相应的解决办法。

## 应用启动原理 ##
应用程序启动可以在三种状态之一中进行，每种状态都会影响应用程序对用户可见所需的时间：冷启动，热启动或热启动。在冷启动时，您的应用程序从头开始。在其他状态中，系统需要将正在运行的应用程序从后台运行到前台。我们建议您始终根据冷启动的假设进行优化。这样做也可以改善热启动和热启动的性能。

为了优化您的应用程序以实现快速启动，了解系统和应用程序级别发生的情况以及它们在每种状态下的交互方式非常有用。

应用程序启动的类型可以分为三种，每种类型所花费的时间是不一样的：
 - **`冷启动：`**当启动应用时，后台没有该应用的进程，这时系统会首先会创建一个新的进程分配给该应用，这种启动方式就是冷启动。
 - **`热启动：`**当启动应用时，后台已有该应用的进程，比如按下 Home 键，这种在已有进程的情况下，会从已有的进程中来启动应用，这种启动方式叫热启动。
 - **`温启动：`**当启动应用时，后台已有该应用的进程，但是启动的入口 Activity 被干掉了，比如按了 Back 键，应用虽然退出了，但是该应用的进程是依然会保留在后台，这种启动方式叫温启动。

冷启动模式下，应用进程完全不存在，系统要新建应用进程。在另外两种模式下，系统只需要将正在运行的应用程序从后台切换到前台。建议您始终根据冷启动的假设进行优化。冷启动速度得到提升了，这样做同样也可以改善热启动和热启动的性能。

那么在应用启动过程中，Android 系统和应用层都做了那些操作呢？理解了它们的内部原理，将会帮助我们做好启动性能优化。

### 冷启动 ###
冷启动指应用重新开始创建：在启动之前，系统进程尚未创建应用程序的进程。冷启动通常发生在自设备启动以来首次启动应用程序或者系统主动杀掉了应用程序的情况下。和其他启动方式相比，冷启动模式需要系统和应用做更多的初始化操作，所以优化起来也有一定的挑战。

在冷启动的开始阶段，系统需要执行以下三个任务：
1. 加载并启动应用程序。
2. 启动后立即显示应用程序的空白启动窗口。
3. 创建应用程序进程。

一旦系统创建了应用程序进程，应用程序进程就会执行下面步骤：
1. 创建应用程序对象。
2. 启动主线程。
3. 创建 Main Activity。
4. 初始化构造 View。
5. 在屏幕上布局。
6. 执行初始化绘制操作。

应用程序进程完成第一次绘制后，系统进程会用 Main Activity 来替换之前已经生成的背景窗口。这个时候，用户就可以使用应用程序了。

冷启动应用程序时，系统和应用程序处理彼此之间的工作的重要部分的直观表示，如下如所示：
![冷启动应用程序重要部分的直观表示](https://henleylee.github.io/medias/android/cold_launch.png)

> 冷启动性能问题可能会出现在应用创建和 Main Activity 创建过程中。

#### Application 创建 ####
当应用程序启动时，空白的启动窗口将会一直保留在屏幕上，直到系统首次完成应用程序的绘制操作。此时，系统进程会替换掉应用程序的启动窗口，允许用户开始与应用程序进行交互。

如果在自己的应用程序中重载 [Application.onCreate()][1] 方法，系统将会在应用程序对象上调用 `onCreate()` 方法。之后，应用程序会创建主线程(也称为 UI 线程)，并执行创建 Main Activity 的过程。

从这个时候开始,系统和应用程序级别的进程将按照[应用程序生命周期阶段](https://developer.android.com/guide/topics/processes/process-lifecycle.html)进行。

#### Activity 创建 ####
当应用进程创建了Activity 后，Activity 会执行以下操作：
1. 初始化值。
2. 调用构造方法。
3. 调用当前生命周期的回调函数(例如 [Activity.onCreate()][2])。

通常情况下，[onCreate()][2] 方法对加载时间的影响最大，因为它要执行的操作更加繁重：加载和构造 View，还有初始化 Activity 运行所需的对象。

### 热启动 ###
应用程序的热启动和冷启动相比，更加简单，开销也更少。在热启动过程中，系统要做的只是把应用程序的 Activity 切换到前台来。如果应用程序的所有 Activity 都驻留在内存中，那么应用就可以避免重复进行对象初始化，布局加载和渲染。

但是，如果系统执行了内存回收操作并触发了回收事件，例如 [onTrimMemory()][3]，那么热启动时仍然需要重新创建这些对象。

在屏幕窗口的操作上，热启动操作具有和冷启动的一样的过程：系统进程将会一直显示一个空白屏幕直到应用完成对 Activity 的渲染。

### 温启动 ###
暖启动过程包含了冷启动过程的部分步骤，同时开销比热启动更小。

暖启动发生在以下几个场景下：
 - 用户退出了应用程序，但随后又重新启动它。这种情况下，应用进程可以继续运行，但是应用程序必须通过调用 [onCreate()][2] 重建 Activity。
 - 应用程序内存被系统回收，然后用户又重新打开它。这个时候，应用程序进程和 Activity 都需要被创建，但应用可以从 Activity 的 [onCreate()][2] 方法的 Bundle 类型参数中拿到系统保存的实例。

## 发现并定位问题 ##
Android 提供了多种方式能让你能够发现并定位 App 的问题。Android vitals 可以给出问题告警，然后诊断工具可以帮助定位出问题。

### Android vitals ###
Android vitals 可以通过 [Play Console](https://play.google.com/apps/publish/signup/) 提醒您应用的启动时间过长，从而帮助您提高应用的性能。Android vitals 判断启动时间过长的标准如下：
 - 冷启动花费 5 秒或更长。
 - 暖启动花费 2 秒或更长。
 - 热启动花费 1.5 秒或更长。

daily session 指的是 App 当天的启动情况。

Android vitals 并不会报告热启动的数据。关于 Google Play 如何收集 Android vitals 数据的详细信息，请参考 [Play Console](https://support.google.com/googleplay/android-developer/answer/7385505) 文档。

### 测量慢启动耗时 ###
为了正确评估应用启动性能，可以关注一些能够显示应用启动时长的数据指标。

#### 初始化显示耗时 ####
在 Android 4.4(API level 19) 和更高的版本中，Logcat 包含了一个名为 `Displayed` 值的输出行。这个值代表了从应用进程启动到完成 Activity 绘制所花费的时间。这个过程包含了以事件序列：
1. 启动应用进程。
2. 初始化对象。
3. 创建和初始化 Activity。
4. 构建布局。
6. 首次绘制应用。

报告的日志行类似于以下示例：
```
ActivityManager：显示com.android.myexample / .StartupTiming：+ 3s534ms
```

如果要从命令行或者终端中追踪 logcat 输出，那么很容易能够找到这个时间值。需要注意的是要在 Android Studio 中查找已用时间，必须在 logcat 视图中禁用过滤器。禁用过滤器是必须的，因为输出此日志的是系统，而不是应用程序本身。

完成设置后，就可以很轻松地找到输出的时间值。下图显示了如何禁用过滤器，并在底部的 logcat 日志输出中显示了 `Displayed` 的值：
![禁用过滤器并输出了 Displayed 的值](https://henleylee.github.io/medias/android/displayed_logcat.png)

Logcat 输出中显示的 Displayed 的值并不一定是所有资源都加载完成后显示的总耗时，它并不包括布局文件中没有引用的资源及初始化对象所引用的资源的加载时间，因为这个加载过程是一个内部过程，不阻塞应用初始内容的显示。

有时候，Logcat 输出中显示的 Displayed 所在行包含了一个附加字段 `total`。例如：
```
ActivityManager: Displayed com.android.myexample/.StartupTiming: +3s534ms (total +1m22s643ms)
```

这种情况下，第一个时间值表示绘制出第一个可见 Activity 的耗时。后面的 `total` 时间指从应用进程的启动开始，可能会包含另一个 Activity 的启动，但这个 Activity 并不可见。`total` 时间只会在启动单个 Activity 时长和总启动时长不一样才显示。

还可以使用 [ADB Shell Activity Manager](https://developer.android.com/studio/command-line/shell.html#am) 命令运行应用程序来测量初始显示的时间。示例如下：
```shell
adb [-d|-e|-s <serialNumber>] shell am start -S -W
com.example.app/.MainActivity
-c android.intent.category.LAUNCHER
-a android.intent.action.MAIN
```

`Displayed` 依旧会之前那样在 logcat 中输出，同时终端窗口也会有以下输出：
```
Starting: Intent
Activity: com.example.app/.MainActivity
ThisTime: 2044
TotalTime: 2044
WaitTime: 2054
Complete
```

`-c` 和 `-a` 参数是可选的，可以指定 Intent 的 [<category>](https://developer.android.com/guide/topics/manifest/category-element.html) 和 [<action>](https://developer.android.com/guide/topics/manifest/action-element.html)。

#### 完全显示耗时 ####
可以使用 [reportFullyDrawn()][4] 方法来测量应用启动到所有资源和视图层次结构的完整显示之间所经过的时间，该方法在应用使用延迟加载的情况下是很有用的。在延迟加载中，应用程序不会阻止窗口的初始绘制，而是异步加载资源并更新视图层次结构。

如果由于延迟加载，应用的初始显示并不包括所有的资源，则可以将所有资源和视图的加载和显示视为单独的度量标准：例如：用户界面可能已经完成了文本的加载，但又必须从网络获取图像。

要解决这个问题，可以手动调用 [reportFullyDrawn()][4]，让系统知道 Activity 已经完成了它的延迟加载。使用此方法时，logcat 将显示出从创建应用对象到调用 [reportFullyDrawn()][4] 方法的时间。下面是 logcat 的输出示例：
```
system_process I/ActivityManager: Fully drawn {package}/.MainActivity: +1s54ms
```

如果确定出显示耗时要比预期的长，可以进一步继续尝试找出启动过程中的性能瓶颈。

#### 确定性能瓶颈 ####
确定性能瓶颈的最好方法是使用 `Android Studio CPU Profiler`。有关信息，请参阅 [Inspect CPU activity with CPU Profiler](https://developer.android.com/studio/profile/cpu-profiler)。

还可以通过应用程序和 Activity 的 `onCreate()` 方法中的内联跟踪来深入了解潜在的瓶颈。要了解内联跟踪，请参阅 [Trace](https://developer.android.com/reference/android/os/Trace.html) 函数和 [Systrace](https://developer.android.com/studio/profile/systrace-commandline.html) 工具。

## 影响启动性能的常见问题 ##
下面讨论经常影响应用程序启动性能的几个问题。主要是关注应用与 Activity 对象的初始化以及画面的加载。

### APP 初始化开销大 ###
Application 的创建过程中，如果执行复杂的逻辑或者初始化大量的对象，将会影响应用的启动体验。具体来说，就是如果继承了 Application 并在初始化时执行了不必要的代码。有些初始化可能是完全不需要的，比如：保存一个实际上由 Intent 启动的 APP 中的 Main Activity 的状态信息，通过 Intent 启动 Activitiy，应用只会使用先前初始化的状态数据的一部分。

其它在 APP 启动期间影响性能的操作还有数量众多的垃圾收集事件，或者在初始化过程中同时发生磁盘I/O，从而进一步阻塞初始化过程。垃圾收集是 Dalvik 运行时特别需要考虑的问题，Art 运行时并发地执行垃圾收集，最大限度地减少了操作的影响。

#### 问题诊断 ####
这种情况下，可以使用方法跟踪或内联跟踪来诊断问题。

#### 方法跟踪 ####
运行 CPU Profiler 会发现 [callApplicationOnCreate()][5] 方法最终调用 `com.example.CustomApplication.onCreate()` 方法。

如果工具显示这些方法需要很长时间才能完成执行，那么就应该好好看看这些方法到底做了哪些操作。 

#### 内联跟踪 ####
使用内联跟踪来找出可能的引发问题的元凶，包括：
 - 应用的初始 [onCreate()][1] 函数。
 - 应用初始化的任何全局单例对象。
 - 任何瓶颈期间可能发生的磁盘 I/O，反序列化，或紧凑的循环操作。

#### 解决方案 ####
不论问题是否由非必要的初始化或磁盘 I/O 引起，解决方案都是要延迟初始化对象：仅初始化那些立即使用的对象。例如，采用单例模式而不是创建全局静态对象，在该模式中，应用程序仅在第一次访问时才创建对象。也可以考虑使用 [Dagger](http://google.github.io/dagger/) 之类的依赖注入框架，当第一次被注入时才创建对象和依赖。

### Activity 初始化操作复杂 ###
Activity 的创建通常包含大量高负载工作。一般来说，可以通过考虑以下方面的问题来优化工作提高性能：
 - 加载比较大或复杂的布局；
 - 磁盘或网络 I/O 阻塞屏幕绘制；
 - 加载和解码 Bitmap；
 - 渲染 [VectorDrawable](https://developer.android.com/reference/android/graphics/drawable/VectorDrawable.html) 对象；
 - Activity 中子系统的初始化。

#### 问题诊断 ####
这种情况下，同样可以使用方法跟踪或内联跟踪来诊断问题。

#### 方法跟踪 ####
当运行 CPU Profiler 工具时，主要的关注点是 APP 中 Application 子类的构造方法和 `com.example.CustomApplication.onCreate()` 方法。

如果工具显示这些方法需要很长时间才能完成执行，那么就应该好好看看这些方法到底做了哪些操作。 

#### 内联跟踪 ####
使用内联跟踪来找出可能的引发问题的元凶，包括：
 - 应用的初始 [onCreate()][1] 函数。
 - 应用初始化的任何全局单例对象。
 - 任何瓶颈期间可能发生的磁盘 I/O，反序列化，或紧凑的循环操作。

#### 解决方案 ####
有很多可能的瓶颈，两个常见的问题和解决方法如下：
 - 视图的层级越多，布局的时候就会花更多时间，可以通过两个步骤优化：
   - 通过减少重复和布局嵌套是布局变得扁平化。
   - 不要加载启动时对用户不可见的布局，可以使用 [ViewStub](https://developer.android.com/reference/android/view/ViewStub.html)。

 - 把所有资源的初始化放到主线程中也会减慢启动速度，可以这样优化：
   - 使用懒加载或非主线程初始化资源。
   - 允许 APP 先展示 视图，然后再更新 Bitmap 或其它可见资源。

### 设置启动页的主题 ###
可以让 APP 使用自定义主题加载，使 APP 的启动屏幕与其它界面保持主题一致，而不是使用系统主题，这样做也可以使 Activity 的启动看起来没那么慢。

通常实现启动屏幕主题化的方法是使用 [`windowDisablePreview`](https://developer.android.com/reference/android/R.attr.html#windowDisablePreview) 属性去关闭初次启动时的白屏。然而，使用这种方法会导致更长的启动时间，并且，当用户点击启动图标后可能会因为无界面反馈而使用户感到疑惑。

#### 问题诊断 ####
通常可以通过观察 APP 启动时是不是没有响应来确定是不是存在问题，这种情况下，屏幕仿佛被冰冻一样卡住，对输入事件也不会有响应。

#### 解决方案 ####
相比于禁用 preview window，可以遵循 [Material Design](http://www.google.com/design/spec/patterns/launch-screens.html#) 模式。可以使用 Acitivity 的 [`windowBackground`](https://developer.android.com/reference/android/R.attr.html#windowBackground) 属性为启动屏幕提供一个简单的背景。 

例如，可以创建一个 xml 文件：
```xml
<layer-list xmlns:android="http://schemas.android.com/apk/res/android" android:opacity="opaque">
  <!-- The background color, preferably the same as your normal theme -->
  <item android:drawable="@android:color/white"/>
  <!-- Your product logo - 144dp color version of your app icon -->
  <item>
    <bitmap
      android:src="@drawable/product_logo_144dp"
      android:gravity="center"/>
  </item>
</layer-list>
```

对应的 Manifest 文件：
```xml
<activity ...
    android:theme="@style/AppTheme.Launcher" />
```

最简单的切换为正常主题的方式是在调用 `super.onCreate()` 和 `setContentView()` 前调用 [setTheme(R.style.AppTheme)][6] ：
```java
public class MyMainActivity extends AppCompatActivity {
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    // Make sure this is before calling super.onCreate
    setTheme(R.style.Theme_MyApp);
    super.onCreate(savedInstanceState);
    // ...
  }
}
```

## 参考 ##
[App startup time](https://developer.android.com/topic/performance/vitals/launch-time)

[1]: https://developer.android.com/reference/android/app/Application.html#onCreate()
[2]: https://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle)
[3]: https://developer.android.com/reference/android/content/ComponentCallbacks2.html#onTrimMemory(int)
[4]: https://developer.android.com/reference/android/app/Activity.html#reportFullyDrawn()
[5]: https://developer.android.com/reference/android/app/Instrumentation.html#callApplicationOnCreate(android.app.Application)
[6]: https://developer.android.com/reference/android/view/ContextThemeWrapper.html#setTheme(int)

