---
title: Android 性能优化之布局优化
categories: Android
tags:
  - Android
  - 性能优化
abbrlink: d59595e2
date: 2019-01-23 18:36:40
---

在 Android 中系统对 View 进行测量、布局和绘制时，都是通过对 View 树的遍历来进行操作的。如果一个 View 树的高度太高就会严重影响测量、布局和绘制的速度。Google 也在其 API 文档中建议 View 高度不宜超过10层。

## 优化思路 ##
布局影响 Android 性能的实质就是影响页面的`测量`和`绘制`时间。
> 一个页面通过递归完成测量和绘制过程，即 `measure`、`layout` 过程。

因此可以从以下几个方面进行优化，从而提高 Android 应用中的页面显示速度：
 - 选择合适的布局类型
 - 减少布局层级
 - 提高布局复用性
 - 减少测量绘制时间

## 优化方案 ##
### 选择合适的布局类型 ###
通过选择合理的布局类型，从而减少嵌套并提高布局性能。
> 即：完成复杂的 UI 效果时，尽可能选择一个功能复杂的布局(如：RelativeLayout、ConstraintLayout)完成，而不要选择多个功能简单的布局(如：FrameLayout、LinearLayout)通过嵌套完成。

提高布局性能就是要选择耗费性能(CPU 资源和时间)较少的布局。
 - 性能耗费低的布局：功能简单(如：FrameLayout、LinearLayout)
 - 性能耗费高的布局：功能复杂(如：RelativeLayout、ConstraintLayout)

> 大约在 Android 4.0 之前，新建工程的默认的布局中根节点是 `LinearLayout`，而在之后已经改为 `RelativeLayout`，因为 `RelativeLayout` 性能更优，且可以简单实现 `LinearLayout` 嵌套才能实现的布局。2017 年 Google 发布了 Android Studio 2.3 正式版，在 Android Studio 2.3 版本中新建的项目中默认的根布局就是 `ConstraintLayout`，因为 `ConstraintLayout` 的性能比 `RelativeLayout` 更好。

### 尽可能少用 wrap_content ###
布局属性 `wrap_content` 会增加布局测量时计算成本，应尽可能少用。在已知宽高为固定值的情况下，不使用 `wrap_content` 属性。

### 使用布局标签 ###
#### <include\> 标签 ####
`<include>` 标签常用于将布局中的公共部分提取出来供其他布局共用，以实现布局模块化，这在布局编写方便提供了大大的便利。

下面以在一个布局 main.xml 中用 `include` 引入另一个布局 foot.xml 为例。main.mxl 代码如下：
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <ListView
        android:id="@+id/simple_list_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_marginBottom="@dimen/dp_80" />

    <include layout="@layout/foot.xml" />

</RelativeLayout>
```

其中 `include` 引入的 foot.xml 为公用的页面底部，代码如下：
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <Button
        android:id="@+id/button"
        android:layout_width="match_parent"
        android:layout_height="@dimen/dp_40"
        android:layout_above="@+id/text"/>

    <TextView
        android:id="@+id/text"
        android:layout_width="match_parent"
        android:layout_height="@dimen/dp_40"
        android:layout_alignParentBottom="true"
        android:text="@string/app_name" />

</RelativeLayout>
```

`<include>` 标签唯一需要的属性是 `layout` 属性，指定需要包含的布局文件。可以定义 `android:id` 和 `android:layout_*` 属性来覆盖被引入布局根节点的对应属性值。注意重新定义 `android:id` 后，子布局的顶结点 `id` 就变化了。

#### <viewstub\> 标签 ####
`<viewstub>` 标签同 `include` 标签一样可以用来引入一个外部布局，不同的是，`viewstub` 引入的布局默认不会扩张，即既不会占用显示也不会占用位置，从而在解析布局时节省 CPU 和内存。`<viewstub>` 标签常用来引入那些默认不会显示，只在特殊情况下显示的布局，如进度布局、网络失败显示的刷新布局、信息出错出现的提示布局等。

下面以在一个布局 main.xml 中加入网络错误时的提示页面 network_error.xml 为例。main.mxl 代码如下：
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

	……

    <ViewStub
        android:id="@+id/network_error_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout="@layout/network_error" />

</RelativeLayout>
```

其中 `viewstub` 引入的 network_error.xml 为只有在网络错误时才需要显示的布局，默认不会被解析，示例代码如下：
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <Button
        android:id="@+id/network_setting"
        android:layout_width="@dimen/dp_160"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"
        android:text="@string/network_setting" />

    <Button
        android:id="@+id/network_refresh"
        android:layout_width="@dimen/dp_160"
        android:layout_height="wrap_content"
        android:layout_below="@+id/network_setting"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="@dimen/dp_10"
        android:text="@string/network_refresh" />

</RelativeLayout>
```

在 Java 中通过 `ViewStub viewStub = findViewById(id)` 找到 `ViewStub`，只有当一个 ViewStub 的 `inflate()` 方法被调用或者被设为可见时，这个 `ViewStub` 所指向的布局文件才会被加载，并替换当前 `ViewStub` 的位置。因此，ViewStub 存在于视图层次，直到 `setVisibility(int)` 或 `inflate()`方法被调用，否则是不加载控件的，所以消耗的资源小。通常也叫它为“懒惰的 include”。

> 注意：对于一个 `ViewStub` 而言，当 `setVisibility(int)` 或 `inflate()` 方法被调用之后，这个 `ViewStub` 在布局中将被使用指定的 `View` 替换，所以被调用过 `inflate()` 方法的 `ViewStub`，如果被隐藏之后再次想要显示，将不能使用 `inflate()` 方法，但是可以再次使用 `setVisibility(int)` 方法设置为可见，这就是这两个方法的区别。


#### <merge\> 标签 ####
在使用了 `include` 后可能导致布局嵌套过多，多余不必要的 layout 节点，从而导致解析变慢，不必要的节点和嵌套可通过 `hierarchy viewer` 或 `设置->开发者选项->显示布局边界`查看。

`<merge>` 标签可用于两种典型情况：
 - 布局顶结点是 `FrameLayout` 且不需要设置 `background` 或 `padding` 等属性，可以用 `merge` 代替，因为 `Activity` 根节点就是个 `FrameLayout`，所以可以用 `merge` 消除只剩一个。
 - 某布局作为子布局被其他布局 `include` 时，使用 `merge` 当作该布局的根节点，这样在被引入时根结点会自动被忽略，而将其子节点全部合并到主布局中。

`<merge>` 只能作为 XML 布局的根标签使用。当Inflate以 `<merge>` 开头的布局文件时，必须指定一个父 `ViewGroup`，并且必须设定 `attachToRoot` 为 `true`。

#### 总结 ####

| 标签         | 功能             | 描述                                                           |
|--------------|------------------|----------------------------------------------------------------|
| `<include>`  | 提高布局复用性   | 提取布局间的公共部分，通过提高布局的复用性从而减少测量绘制时间 |
| `<viewstub>` | 减少测量绘制时间 | 减少测量绘制时间，可直接提高布局性能                           |
| `<merge>`    | 减少布局层级     | 减少布局层级，可直接减少测量绘制时间，提高布局性能             |

### 其他优化 ###
#### 用 SurfaceView 或 TextureView 代替普通 View ####
`SurfaceView` 或 `TextureView` 可以通过将绘图操作移动到另一个单独线程上提高性能。普通 View 的绘制过程都是在主线程(UI 线程)中完成，如果某些绘图操作影响性能就不好优化了，这时可以考虑使用 `SurfaceView` 和 `TextureView`，他们的绘图操作发生在 UI 线程之外的另一个线程上。

因为 `SurfaceView` 在常规视图系统之外，所以无法像常规试图一样移动、缩放或旋转一个 `SurfaceView`。`TextureView` 是 Android 4.0 引入的，除了与 `SurfaceView` 一样在单独线程绘制外，还可以像常规视图一样被改变。

#### 使用 RenderJavascript ####
`RenderScript` 是 Adnroid 3.0 引进的用来在 Android 上写高性能代码的一种语言，语法给予 C 语言的 C99 标准，他的结构是独立的，所以不需要为不同的 CPU 或者 GPU 定制代码。

#### 使用 OpenGL 绘图 ####
Android 支持使用 `OpenGL` API 的高性能绘图，这是 Android 可用的最高级的绘图机制，在游戏类对性能要求较高的应用中得到广泛使用。

Android 4.3 最大的改变，就是支持 `OpenGL ES 3.0`。相比 2.0，3.0 有更多的缓冲区对象、增加了新的着色语言、增加多纹理支持等等，将为 Android 游戏带来更出色的视觉体验。

## 布局调优工具 ##
### Hierarchy Viewer ###
`Hierarchy Viewer` 是 Android Studio 提供的 UI 性能检测工具。可以帮助开发者可视化获得 UI 布局设计结构和各种属性信息，帮助优化布局设计。Hierarchy Viewer 可以方便地查看 Activity 布局，各个 View 的属性、布局测量和绘制的时间。具体介绍可见 [Hierarchy Viewer](https://developer.android.com/studio/profile/hierarchy-viewer)

> Android Studio 3.1 或更高版本中，Hierarchy Viewer 已经被废弃，推荐在运行时使用 `Layout Inspector` 来检查应用程序的视图层次结构。

### Layout Inspector ###
`Layout Inspector` 是 Android Studio 替代 Hierarchy Viewer 的新方案。利用 Android Studio 中的布局检查器，可以在运行时从 Android Studio IDE 内检查自己应用的视图层次结构。如果布局在运行时(而不是完全在 XML 中)构建并且布局没有按预期显示，这种检查将非常有用。具体介绍可见 [Layout Inspector](https://developer.android.com/studio/debug/layout-inspector)

按以下步骤操作，打开布局检查器：
 - 在连接的设备或模拟器上运行您的应用。
 - 点击 Tools > Android > Layout Inspector。
 - 在出现的 Choose Process 对话框中，选择想要检查的应用进程，然后点击 OK。

> 注意：设备必须运行 Android 4.1 或更高版本。

### Lint ###
`Lint` 是 Android Studio 提供的一款代码扫描分析工具，可以扫描、发现代码结构和质量问题并提供解决方案。Lint 发现的每个问题都有描述信息和等级(和测试发现 bug 很相似)，可方便定位问题并按照严重程度进行解决。具体介绍可见 [Lint](https://developer.android.com/studio/write/lint)

### Systrace ###
`Systrace` 是 Android 4.1 中新增的性能数据采样和分析工具。它可帮助开发者收集 Android 关键子系统(如 Surfaceflinger、WindowManagerService 等 framework 部分关键模块、服务)的运行信息，从而帮助开发者更直观的分析系统瓶颈，改进性能。具体介绍可见 [Systrace](https://developer.android.google.cn/studio/command-line/systrace)

Systrace 的功能包括跟踪系统的 I/O 操作、内核工作队列、CPU 负载以及 Android 各个子系统的运行状况等。在 Android 平台中，它主要由以下三部分组成：
 - 内核部分：Systrace 利用了 Linux Kernel 中的 ftrace 功能。所以要使用 Systrace 的话，必须开启 kernel 中和 ftrace 相关的模块。
 - 数据采集部分：Android 定义了一个 Trace 类。应用程序可利用该类把统计信息输出给 ftrace。同时，Android 还有一个 atrace 程序，它可以从 ftrace 中读取统计信息然后交给数据分析工具来处理。
 - 数据分析工具：Android 提供了一个 systrace.py(Python 脚步文件，位于 Android SDK 目录/tools/systrace 中，其内部将调用 atrace 程序)用来配置数据采集的方式(如采集数据的标签、输出文件名等)和收集 ftrace 统计数据并生成一个结果网页文件供用户查看。

本质上，Systrace 是对 Linux Kernel 中 ftrace 的封装，应用程序需要利用 Android 提供的 Trace 类来使用 Systrace。

