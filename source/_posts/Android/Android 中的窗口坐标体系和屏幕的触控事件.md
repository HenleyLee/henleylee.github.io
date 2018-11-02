---
title: Android 中的窗口坐标体系和屏幕的触控事件
date: 2018-07-24 13:41:50
categories: Android
tags:
  - Android
---

## Android坐标系 ##
在物理学中，要描述一个物体的运动，就必须选定一个参考系。所谓滑动，正是相对于参考系的运动。
在 Android 中，将屏幕最左上角的顶点作为 Android 坐标系的原点，从这个点向右是 X 轴正方向，从这个点向下是 Y 轴的正方向，如下图所示：
![Android坐标系](https://user-gold-cdn.xitu.io/2018/7/18/164ab1ab97865a55?w=439&h=447&f=png&s=3300)

系统提供了 `getLocationOnScreen(int location[])` 这样的方法来获取 Android 坐标系中点的位置，即该视图左上角在 Android 坐标系的坐标。
另外，在触控事件中使用 `getRawX()`、`getRawY()` 方法所获得的坐标同样是 Android 坐标系中的坐标。

## 视图坐标系 ##
Android 中除了上面所说的这种坐标系之外，还有一个视图坐标系，它描述了子视图在父视图中的位置关系。
这两种坐标系并不矛盾也不复杂，他们的作用是相辅相成的。与 Android 坐标系类似，视图坐标系同样是以原点向右为 X 轴正方向，以原点向下为 Y 轴正方向，
只不过在视图坐标系中，原点不再是 Android 坐标系中的屏幕最左上角，而是以父视图左上角为坐标原点，如下图所示：
![视图坐标系](https://user-gold-cdn.xitu.io/2018/7/18/164ab1c1d333de4b?w=426&h=461&f=png&s=3554)

在触控事件中，通过 `getX()`、`getY()` 所获得的坐标就是视图坐标系中的坐标。

## 触控事件——MotionEvent ##
触控事件 MotionEvent 在用户交互中，站着举足轻重的地位，学好触控事件是掌握后序内容的基础。
首先，来看看 MotionEvent 中封装的一些常用的事件常量，它定义了触控事件的不同类型。
```java
// 单点触摸按下动作
public static final int ACTION_DOWN             = 0;
// 单点触摸离开动作
public static final int ACTION_UP               = 1;
// 触摸点移动动作
public static final int ACTION_MOVE             = 2;
// 触摸动作取消
public static final int ACTION_CANCEL           = 3;
// 触摸动作超出边界
public static final int ACTION_OUTSIDE          = 4;
// 多点触摸按下动作
public static final int ACTION_POINTER_DOWN     = 5;
// 多点离开动作
public static final int ACTION_POINTER_UP       = 6; 
```

通常情况下，我们会在 `onTouchEvent(MotionEvent event)` 方法中通过 `event.getAction()` 方法来获取触控事件的类型，并使用 `switch-case` 方法来进行筛选，这个代码的模式基本固定，如下所示：
```java
@Override
public boolean onTouchEvent(MotionEvent event) {
    // 获取当前输入点的X、Y坐标(视图坐标)
    int x = (int) event.getX();
    int y = (int) event.getY();
    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            // 处理输入的按下事件
            break;
        case MotionEvent.ACTION_MOVE:
            // 处理输入的移动事件
            break;
        case MotionEvent.ACTION_UP:
            // 处理输入的离开事件
            break;
    }
    return true;
}
```

在不涉及多点操作的情况下，通常可以使用以上代码来完成触控事件的监听，不过这里只是一个代码模板，后面我们会在触控事件中完成具体的逻辑。

在 Android 中，系统提供了非常多的方法来获取坐标值、相对距离等。方法丰富固然好，但也给初学者带来了很多困惑，不知道在什么情况下使用什么方法，下面总结了一些 API，结合 Android 坐标系来看看该如何使用它们，如下图所示：
![获取坐标值、相对距离](https://user-gold-cdn.xitu.io/2018/7/18/164ab1f3899e3552?w=463&h=459&f=png&s=5353)

获取坐标值、相对距离的方法可以分成如下两个类别：
 - View提供的获取坐标方法
    * getTop()：获取到的是View自身的顶部到其父View顶部的距离。
    * getLeft()：获取到的是View自身的左侧到其父View左侧的距离。
    * getRight()：获取到的是View自身的右侧到其父View左侧的距离。
    * getBottom()：获取到的是View自身的底部到其父View顶部的距离。

 - MotionEvent提供的方法
    * getX()：获取触摸点距离View左侧的距离，即视图坐标。
    * getY()：获取触摸点距离View顶部的距离，即视图坐标。
    * getRawX()：获取触摸点距离整个屏幕左侧的距离，即绝对坐标。
    * getRawY()：获取触摸点距离整个屏幕顶部的距离，即绝对坐标。

> 注意：View的坐标系统是相对于父控件而言的。
