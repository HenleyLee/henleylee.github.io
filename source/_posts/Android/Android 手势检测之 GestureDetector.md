---
title: Android 手势检测之 GestureDetector
categories: Android
tags:
  - Android
abbrlink: da8b8b2f
date: 2019-01-13 10:36:35
---

当用户触摸屏幕的时候，会产生许多手势，例如 down、up、scroll，filing 等等。

一般情况下，我们知道 `View` 类有个 `View.OnTouchListener` 内部接口，通过重写它的 `onTouch(View v, MotionEvent event)` 方法，可以处理一些触摸事件，但是这个方法太过简单，如果需要处理一些复杂的手势，用这个接口就会很麻烦(比如需要根据用户触摸的轨迹去判断是什么手势)。

Android SDK 提供了 `GestureDetector` 类来帮助开发者识别一些基本的触摸手势，主要是通过它的 `onTouchEvent(event)` 方法完成了不同手势的识别。虽然它能识别手势，但是不同的手势要怎么处理，应该是提供给程序员实现的。

## GestureDetector 介绍 ##

Detector 的意思就是探测者，所以 `GestureDetector` 就是用来监听手势的发生。`GestureDetector` 类对外提供了三个接口：`OnGestureListener`、`OnDoubleTapListener`、`OnContextClickListener`，用来回调不同类型的触摸事件。`GestureDetector` 的类图如下如所示：
![GestureDetector类图](https://henleylee.github.io/medias/view/view_gesture_detector.png)

`GestureDetector` 类里这些接口的方法，就是相应触摸事件的回调，实现了这些方法，就能实现传入触摸事件之后做出相应的回调。`GestureDetector` 还有一个内部类 `SimpleOnGestureListener`，实现了这三个接口。

### OnGestureListener ###
**`OnGestureListener`** 接口主要用于手势检测，有以下类型事件：按下(Down)、触摸反馈(ShowPress)、长按(LongPress)、单击抬起(SingleTapUp)、滚动(Scroll)、抛(Fling)。`OnGestureListener` 接口包含以下方法：
 - **`boolean onDown(MotionEvent e)：`**用户按下屏幕就会触发。
 - **`void onShowPress(MotionEvent e)：`**用户按下屏幕后100ms(Android 源码)还没有松开或者移动就会触发，官方在源码的解释是说一般用于告诉用户已经识别按下事件的回调。
 - **`void onLongPress(MotionEvent e)：`**用户按下屏幕一定时间后(源码里默认是100ms+500ms)触发，触发之后不会触发其他回调，直至松开(UP事件)。 
 - **`boolean onSingleTapUp(MotionEvent e)：`**用户手指松开(UP事件)的时候如果没有执行 onScroll() 和 onLongPress() 这两个回调的话，就会触发，说明这是一个点击抬起事件，但是不能区分是否双击事件的抬起。
 - **`boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY)：`**手指滑动时就会触发(接收到 MOVE 事件，且位移大于一定距离)，e1、e2 分别是之前 DOWN 事件和当前的 MOVE 事件，distanceX 和 distanceY 分别表示自上次调用 onScroll() 以来，沿 X、Y 轴滑动的距离。 
 - **`boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY)：`**用户按下屏幕执行抛操作之后的回调，MOVE 事件之后手松开(UP 事件)瞬间的 X 或者 Y 方向速度，如果达到一定数值(源码默认是每秒50px)，就是抛操作(也就是快速滑动的时候松手会有这个回调，因此基本上有 onFling() 必然有 onScroll())。

### OnDoubleTapListener ###
**`OnDoubleTapListener`** 接口主要用于监听监听单击和双击事件，有以下类型事件：双击(DoubleTap)、单击确认(SingleTapConfirmed) 和 双击事件回调(DoubleTapEvent)。`OnDoubleTapListener` 接口包含以下方法：
 - **`boolean onSingleTapConfirmed(MotionEvent e)：`**当单击事件发生时(确认为单击事件而非双击事件)触发。
 - **`boolean onDoubleTap(MotionEvent e)：`**当双击事件发生时触发。
 - **`boolean onDoubleTapEvent(MotionEvent e)：`**onDoubleTap() 回调之后的输入事件(DOWN、MOVE、UP)都会触发这个方法(该方法可以实现一些双击后的控制，如让 View 双击后变得可拖动等)。

### OnContextClickListener ###
**`OnContextClickListener`** 接口主要用于检测外部设备上的按钮是否按下，它是在 Android 6.0(API 23)才添加的一个接口。`OnContextClickListener` 接口包含以下方法：
 - **`boolean onContextClick(MotionEvent e)：`**外部设备上的按钮被按下就会触发。

### SimpleOnGestureListener ###
**`SimpleOnGestureListener`** 类是上述三个接口的空实现，一般情况下使用这个比较多，也比较方便。

## GestureDetector 使用 ##
**`GestureDetector`** 可以使用 `MotionEvent` 检测各种手势和事件。`GestureDetector.OnGestureListener` 回调将在特定的事件发生时通知用户。这个类只能用于检测触摸事件的 MotionEvent，不能用于轨迹球事件。 

### 使用方法 ###
使用 `GestureDetector` 需要以下几个步骤：
 - 为 `View` 创建一个 `GestureDetector` 实例。
 - 在  `View` 的 `onTouchEvent(MotionEvent ev)` 方法中，确保调用 `GestureDetector` 的 `onTouchEvent(MotionEvent ev)` 方法。回调中定义的方法将在事件发生时执行。
 - 如果侦听 `onContextClick(MotionEvent ev)`，则必须在 `View` 的 `onGenericMotionEvent(MotionEvent ev)` 方法中调用 `GestureDetector` 的 `onGenericMotionEvent(MotionEvent ev)` 方法。

### 构造方法 ###
`GestureDetector` 一共有 5 种构造函数，但有 2 种被废弃了，1 种是重复的，所以只需要关注其中的 2 种构造函数即可，如下：
```java
public GestureDetector(Context context, OnGestureListener listener)

public GestureDetector(Context context, OnGestureListener listener, Handler handler)
```

第 1 种构造函数里面需要传递两个参数，`Context`(上下文)和 `OnGestureListener`(手势监听器)，这个很容易理解，也是最经常使用的一种。

第 2 种构造函数则需要多传递一个 `Handler` 作为参数，这个有什么作用呢？其实作用也非常简单，这个 `Handler` 主要是为了给 `GestureDetector` 提供一个 `Looper`。
> 在通常情况下是不需这个 `Handler` 的，因为它会在内部自动创建一个 `Handler` 用于处理数据，如果在主线程中创建 `GestureDetector`，那么它内部创建的 `Handler` 会自动获得主线程的 `Looper`，然而如果在一个没有创建 `Looper` 的子线程中创建 `GestureDetector` 则需要传递一个带有 `Looper` 的 `Handler` 给它，否则就会因为无法获取到 `Looper` 导致创建失败。**`重点是传递的 Handler 一定要有 Looper，重点是 Looper，而非 Handler`**。

在没有 `Looper` 的地方使用 `GestureDetector` 时可以通过以下方法：
```java
// 方法一：在主线程创建 Handler，使用第 2 种构造方法进行创建
final Handler handler = new Handler();
new Thread(new Runnable() {
    @Override
    public void run() {
        final GestureDetector detector = new GestureDetector(getContext(), new GestureDetector.SimpleOnGestureListener(), handler);
        // ... 省略其它代码 ...
    }
}).start();

// 方法二：在子线程创建 Handler，并且指定 Looper，使用第 2 种构造方法进行创建
new Thread(new Runnable() {
    @Override
    public void run() {
        final Handler handler = new Handler(Looper.getMainLooper());
        final GestureDetector detector = new GestureDetector(getContext(), new GestureDetector.SimpleOnGestureListener(), handler);
        // ... 省略其它代码 ...
    }
}).start();

// 方法三：子线程准备了 Looper，那么可以直接使用第 1 种构造方法进行创建
new Thread(new Runnable() {
    @Override
    public void run() {
        Looper.prepare(); // 初始化Looper(重点)
        final GestureDetector detector = new GestureDetector(getContext(), new GestureDetector.SimpleOnGestureListener());
        // ... 省略其它代码 ...
    }
}).start();
```

### 相关方法 ###
`GestureDetector` 中除了各类监听器之外，与 `GestureDetector` 相关的方法其实并不多，只有以下几个：

| 方法                                                          | 描述                                                                                   |
|---------------------------------------------------------------|----------------------------------------------------------------------------------------|
| void setIsLongpressEnabled(boolean isLongpressEnabled)        | 通过布尔值设置是否允许触发长按事件，true 表示允许，false 表示不允许                    |
| boolean isLongpressEnabled()                                  | 判断当前是否允许触发长按事件，true 表示允许，false 表示不允许                          |
| boolean onTouchEvent(MotionEvent ev)                          | 这个是其中一个重要的方法，在最开始已经演示过使用方式了                                 |
| boolean onGenericMotionEvent(MotionEvent ev)                  | 这个是在 API 23 之后才添加的内容，主要是为 OnContextClickListener 服务的，暂时不用关注 |
| void setOnDoubleTapListener(OnDoubleTapListener listener)     | 设置 OnDoubleTapListener                                                               |
| void setContextClickListener(OnContextClickListener listener) | 设置 ContextClickListener                                                              |


