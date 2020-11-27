---
title: Android 内存分配和内存泄漏
categories: Android
tags:
  - Android
  - Java
abbrlink: '20774213'
date: 2018-07-29 18:55:45
---

## 1. 内存分配 ##

`Java 虚拟机`在执行 `Java` 程序的过程中会把它所管理的内存划分为若干个不同的数据区域。这些区域都有各自的用途、创建和销毁的时间，有一些是随虚拟机的启动而创建，随虚拟机的退出而销毁，有些则是与线程一一对应，随线程的开始和结束而创建和销毁。

`Java 虚拟机`所管理的内存将会包括以下几个运行时数据区域，如下图（图片来自网络）：
![Java 虚拟机内存区域](https://henleylee.github.io/medias/java/java_memory_partition.png)

### 1.1 程序计数器（Program Counter Register） ###
程序计数器是一块较小的内存空间，它的作用可以看做是当前线程所执行的字节码的信号指示器。字节码解释器就是通过改变该计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需依赖计数器来完成。

每一个 JVM 线程都有独立的程序计数器，各线程间的计数器互不影响，独立存储，确保线程切换后能够恢复到正确的执行位置。

在任意时刻，一条 JVM 线程只会执行一个方法的代码。该方法称为该线程的当前方法(Current Method)，如果该方法是 Java 方法，那计数器保存 JVM 正在执行的字节码指令的地址；如果该方法是 Native，那 PC 寄存器的值为空(Undefined)。
    
此内存区域是唯一一个在 Java 虚拟机规范中没有规定任何 OutOfMemoryError 情况的区域。
    
### 1.2 Java虚拟机栈（Java Virtual Machine Stack） ###
Java 虚拟机栈与程序计数器一样，也是线程私有的，其生命周期与线程相同。虚拟机栈描述的是 Java 方法执行的内存模型：每个方法被执行的时候都会同时创建一个栈帧(Stack Frame)用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法被调用直至执行完成的过程就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。
 
局部变量表存放了编译期可知的各种基本数据类型(boolean、byte、char、short、int、float、long、double)、对象引用(Reference 类型)和 returnAddress 类型（指向了一条字节码指令的地址）。其中64位长度的 long 和 double 会占用2个局部变量空间(Slot)，其余数据类型只占用1个。局部变量表所需的空间在编译期间完成分配，当进入一个方法时，其需要在帧中分配多大的局部变量空间是确定的，方法运行期间不会改变局部变量表的大小。
 
Java 虚拟机规范中对该区域规定了两种异常情况：
 - 如线程请求的深度大于虚拟机所允许的深度，抛出 StackOverflowError 异常。
 - 虚拟机栈动态扩展无法申请到足够的内存时，抛出 OutOfMemoryError 异常。
    
### 1.3 本地方法栈（Native Method Stack） ###
Java 虚拟机可能会使用到传统的栈来支持 native 方法（使用 Java 语言以外的其它语言编写的方法）的执行，这个栈就是本地方法栈(Native Method Stack)。本地方法栈与虚拟机栈非常类似，区别是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，二本地方法栈则为虚拟机使用到的Native方法服务。虚拟机规范对本地方法栈中的方法是用语言、使用方式与数据结构没强制规定，因此虚拟机可以自由实现，如Sun HotSpot虚拟机直接把本地方法栈和虚拟机栈合二为一。

Java 虚拟机规范中对该区域规定了两种异常情况：
 - 如线程请求的深度大于虚拟机所允许的深度，抛出 StackOverflowError 异常。
 - 虚拟机栈动态扩展无法申请到足够的内存时，抛出 OutOfMemoryError 异常。

### 1.4 Java堆（Java Heap） ###
Java 堆是 Java 虚拟机管理内存中最大的一块，是所有线程共享的内存区域，随虚拟机的启动而创建。该区域唯一目的是存放对象实例，几乎所有对象的实例都在堆里面分配。Java 堆是垃圾收集器管理的主要区域，被称作“GC堆”。
 
Java 虚拟机规范规定，Java 堆可以出于物理上物理上不连续的内存空间中，只要逻辑上连续即可，如同磁盘空间一样，既可以实现成固定大小，也可以是扩展的，当前主流虚拟机都是按照扩展来实现的（通过 -Xmx 和 -Xms 控制）。

Java 虚拟机规范中对该区域规定了 OutOfMemoryError 异常：如果堆中没有内存完成实例分配，并且堆无法再扩展则抛出 OutOfMemoryError 异常。
    
### 1.5 方法区（Method Area） ###
方法区与 Java 堆一样，是各个线程共享的内存区域，用于存储一杯虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。Java 虚拟机对这个区域的限制非常宽松，处理和 Java 对一样不需要连续的内存和可以选择固定大小或者可扩展外，还可以选择不实现垃圾收集。

Java 虚拟机规范中对该区域规定了 OutOfMemoryError 异常： 如果方法区的内存空间不能满足内存分配请求，那 Java 虚拟机将抛出一个 OutOfMemoryError 异常。
    
### 1.6 运行时常量池（Runtime Constant Pool） ###
运行时常量池是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等信息外，还有一项信息是常量池，用于存放编译期生成的各种字面常量和符号引用，这部分内容在类加载后存放到方法区的常量池中。

Java 虚拟机规范中对该区域规定了 OutOfMemoryError 异常： 当常量池无法申请到内存时抛出 OutOfMemoryError 异常。
    
### 1.7 直接内存（Direct Memory） ###
直接内存并不是虚拟机运行时数据区域的一部分，也不是Java虚拟规范中定义的内存区域，但这部分内存也被频繁使用，并且可能导致 OutOfMemoryError 异常出现。
Java虚拟机需要根据实际内存的大小来设置-Xmx等参数信息，如果忽略了直接内存，使得各个内存区域的总和大于物理内存限制，从而导致动态扩展时抛出 OutOfMemoryError 异常。

## 2. 内存优化 ##

内存优化的目的就是让我们在开发中有效的避免应用出现内存泄漏的问题。内存泄漏大家都不陌生了，简单粗俗的讲，就是该被释放的对象没有释放，一直被某个或某些实例所持有却不再被使用导致 GC 不能回收。

像 Java 这样具有垃圾回收功能的语言的好处之一，就是程序员无需手动管理内存分配。这减少了段错误（Segmentation fault，即访问的内存超出了系统所给这个程序的内存空间）导致的闪退，也减少了内存泄漏导致的堆空间膨胀，让编写的代码更加安全。然而，Java中依然有可能发生内存泄漏。所以你的 APP 依然有可能浪费了大量的内存，甚至由于内存耗尽（OOM）导致闪退，内存优化也就至关重要。

## 3. 内存泄漏 ##

传统的内存泄漏是由忘记释放分配的内存导致的，而逻辑上的内存泄漏则是由于忘记在对象不再被使用的时候释放对其的引用导致的。如果一个对象仍然存在强引用，垃圾回收器就无法对其进行垃圾回收。

Android程序开发中，如果一个对象已经不需要被使用了，本该被回收时，而这时另一个对象还在持有对该对象的引用，这样就会导致无法被GC回收，就会出现内存泄漏的情况。内存泄漏时Android程序中出现OOM问题的主要原因之一。所以我们在编写代码时，一定要细心处理好这一类的问题。下面介绍一下Android开发中最常见的内存泄漏问题：

### 3.1 单例设计模式导致内存泄漏 ###
单例设计模式的静态特性会使他的生命周期和应用程序的生命周期一样长，这就说明了如果一个对象不在使用了，而这时单例对象还在持有该对象的引用，这时GC就会无法回收该对象，造成了内存泄露的情况。

下面是错误的示例：
```java
public class AppManager {

    private static AppManager sInstance;
    private Context context;
    
    private AppManager(Context context) {
        this.context = context;
    }

    public static AppManager getInstance(Context context) {
        if (sInstance == null) {
            sInstance = new AppManager(context);
        }
        return sInstance;
    }    
}
```
这是一个普通的单例模式，当创建这个单例的时候，由于需要传入一个 `Context` ，所以这个 `Context` 的生命周期的长短至关重要：
 - 如果此时传入的是 `Application` 的 Context，因为 Application 的生命周期就是整个应用的生命周期，所以这将没有任何问题。
 - 如果此时传入的是 `Activity` 的 Context，当这个 Context 所对应的 Activity 退出时，由于该 Context 的引用被单例对象所持有，其生命周期等于整个应用程序的生命周期，所以当前 Activity 退出时它的内存并不会被回收，这就造成泄漏了。

解决方案：可以通过 `this.context = context.getApplicationContext()` 获取 `Application` 的 `Context`，这样就不会造成内存泄漏了。

### 3.2 非静态内部类/匿名内部类导致内存泄漏 ###
在 Java 中，非静态内部类和匿名内部类都会隐式的持有外部类的引用，而且它们的生命周期甚至比外部类更长，这便埋下了内存泄露的隐患。

下面是错误的示例：
```java
public class Outer {

    int outerValue = 0;

    class Inner {
        void innerMethod() {
            int innerValue = outerValue;
        }
    }

}
```
解决方案：尽量使用静态内部类来替代内部类。

### 3.3 Thread/AsyncTask导致内存泄漏 ###
直接创建一个Thread/AsyncTask对象执行耗时任务，这种方式新建的子线程Thread和AsyncTask都是匿名内部类对象，默认就隐式的持有外部 Activity 的引用，外部 Activity 销毁时耗时任务如果还没有执行完毕，Activity 实例不会被销毁了，于是导致内存泄漏。

下面是错误的示例：
```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        executeTask();
    }

    private void executeTask() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                // 模拟耗时操作
                try {
                    Thread.sleep(10 * 1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

}
```
解决方案：尽量使用静态内部类来替代内部类，同时避免让长期运行的任务（ 线程 ）持有 `Activity` 的引用。

### 3.4 Timer和TimerTask导致内存泄露 ###
Timer和 TimerTask 在Android中通常会被用来做一些计时或循环任务，当 Activity 销毁时，有可能 Timer 还在继续等待执行 TimerTask，它持有 Activity 的引用不能被回收：

下面是错误的示例：
```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        executeTask();
    }

    private void executeTask() {
        Timer timer = new Timer();
        timer.schedule(new TimerTask() {
            public void run() {
                // do something ...
            }
        }, 1000 , 1000);
    }

}
```
解决方案：Activity 销毁的时候要立即 cancel 掉 Timer 和 TimerTask，以避免发生内存泄漏。

### 3.5 Handler 造成的内存泄漏 ###
定义匿名的 Handler ，用匿名类 Handler 执行匿名的 Runnable。Runnable 内部类会持有外部类的隐式强引用，被传递到 Handler 的消息队列 MessageQueue 中，在 Message 消息没有被处理之前， Activity 实例不会被销毁了，于是导致内存泄漏。

下面是错误的示例：
```java
public class MainActivity extends AppCompatActivity {

    private Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            // do something ...
        }
    };

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mHandler.postDelayed(new Runnable() {
            @Override
            public void run() {
                // do something ...
            }
        }, 10 * 1000);
        mHandler.sendMessageDelayed(Message.obtain(), 10 * 1000);
    }

}
```
解决方案：采用静态内部类来替代非静态内部类，并且使用 `WeakReference` 来引用外部类对象，如果对象只存在弱引用的话，GC 是会回收这部分内存的。

### 3.6 静态变量导致内存泄露 ###
有的时候我们可能会在启动频繁的Activity中，为了避免重复创建相同的数据资源，或者为了使某个变量在别的类中也可以使用，可能会出现这种写法： 
```java
public class MainActivity extends AppCompatActivity {

    private static Context mContext;
    private static AppInfo mAppInfo;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mContext = this;
        if (mAppInfo == null) {
            mAppInfo = new AppInfo();
        }
    }

    private class AppInfo {
        // ...
    }

}
```
解决方案：尽量少地使用静态持有的变量，在适当的时候讲静态量重置为null，使其不再持有引用，这样也可以避免内存泄露。

### 3.7 未取消注册或回调导致内存泄露 ###
在开发中我们可能需要使用到 `BroadcastReceiver` 或 `EventBus`，如果注册了之后，在 `Activity/Fragment` 销毁时没有反注册，`Activity/Fragment` 实例不会被销毁了，于是导致内存泄漏。： 
```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction(ConnectivityManager.CONNECTIVITY_ACTION);
        this.registerReceiver(mReceiver, intentFilter);
    }

    private BroadcastReceiver mReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            if (ConnectivityManager.CONNECTIVITY_ACTION.equals(intent.getAction())) {
                // do something ...
            }
        }
    };

}
```
解决方案：在使用`BroadcastReceiver` 或 `EventBus`时，一定要及得在`Activity/Fragment` 被销毁时发注册。

### 3.7 集合中的对象未清理导致内存泄露 ###
如果一个对象放入到 `ArrayList`、`HashSet` 等集合中，这个集合就会持有该对象的引用。当我们不再需要这个对象时，也并没有将它从集合中移除，这样只要集合还在使用（而此对象已经无用了），这个对象就造成了内存泄露。并且如果集合被静态引用的话，集合里面那些没有用的对象就会造成内存泄。

解决方案：在使用集合时要及时将不用的对象从集合 remove，或者 clear 集合，从而避免内存泄露。

### 3.8 资源未关闭或释放导致内存泄露 ###
在使用`IO流`、`File` 流或者 `Sqlite`、`Cursor` 等资源时要及时关闭。这些资源在进行读写操作时通常都使用了缓冲，如果及时不关闭，这些缓冲对象就会一直被占用而得不到释放，以致发生内存泄露。

解决方案：在不需要使用`IO流`、`File` 流或者 `Sqlite`、`Cursor` 等资源时要及时关闭，以便缓冲能及时得到释放，从而避免内存泄露。

### 3.9 WebView 导致内存泄露 ###
Android 混合开发时经常用到 `WebView` 加载 `html` 等页面，而 `WebView` 的内存泄漏就是最经常遇到的问题，尤其是当项目中需要用 `WebView`加载的页面比较多时。

解决方案：

 - 通过 `WebView webView = new WebView(getApplicationContext())` 动态创建 `WebView` 代替在 `xml` 中定义。
 - 页面销毁时先将 `WebView` 从父容器中移除，然后再销毁 `WebView` 。
```java
    public void destroyWebView() {
        if (webView != null) {
            ViewParent parent = webView.getParent();
            if (parent != null && parent instanceof ViewGroup) {
                ((ViewGroup) parent).removeView(webView);
            }
            webView.stopLoading();          // 停止加载
            webView.clearMatches();         // 清除创建的高亮显示文本匹配
            webView.clearHistory();         // 清除历史记录
            webView.clearFormData();        // 清除表单数据
            webView.clearAnimation();       // 取消视图动画
            webView.removeAllViews();       // 移除子视图
            webView.destroy();              // 销毁WebView
        }
    }
```

## 4. 内存泄漏检测 ##
### 4.1 Android Profiler ###
`Android Studio 3.0` 采用全新的 `Android Profiler` 窗口取代 `Android Monitor` 工具。 这些全新的分析工具能够提供关于应用 `CPU`、`内存`和`网络` Activity 的实时数据。[参考](https://developer.android.google.cn/studio/profile/android-profiler)

### 4.2 Android Lint ###
`Android Studio` 提供一个名为 `Lint` 的代码扫描工具，可帮助您发现并纠正代码结构质量的问题，而无需实际执行该应用，也不必编写测试用例。该工具会报告其检测到的每个问题并提供该问题的描述消息和严重级别，以便您可以快速确定需要优先进行哪些关键改进。此外，您可以调低问题的严重级别，忽略与项目无关的问题，也可以调高严重级别，以突出特定问题。[参考](https://developer.android.google.cn/studio/write/lint)

### 4.3 StrictMode ###
`StrictMode` 是 Android 系统提供的 `API` ，在开发环境下引入可以更早的暴露发现问题，能够动态的检测内存泄露。[参考](https://developer.android.google.cn/reference/android/os/StrictMode)
 
### 4.4 LeakCanary ###
`LeakCanary` 是 Android 查找内存泄漏的主要工具，由 Square 公司开发，可以直接在手机端查看内存泄露的工具。[参考](https://github.com/square/leakcanary)

## 5. 参考 ##
1. [Java虚拟机运行时数据区域](https://blog.csdn.net/nms312/article/details/37361121)
2. [Android内存优化——常见内存泄露及优化方案](https://www.jianshu.com/p/ab4a7e353076)

