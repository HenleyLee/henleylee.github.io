---
title: Android Window 类参数详解
categories: Android
tags:
  - Android
abbrlink: 7605f9c8
date: 2020-06-07 12:15:56
---

`Window` 是一个抽象类，它作为一个顶级视图添加到 `WindowManager` 中，`View` 是依附于 `Window` 而存在的，对 `View` 进行管理。`PhoneWindow：`Window 的唯一实现类，添加到 WindowManager 的根容器中。

`WindowManager` 是一个接口，继承自接口 `ViewManager`，对 `Window` 进行管理。`WindowManager` 是 `Window` 的容器，管理着 `Window`，对 `Window` 进行添加和删除，最终具体的工作都是由 `WindowManagerService` 来处理的，`WindowManager` 和 `WindowManagerService` 通过 `Binder` 来进行跨进程通信，`WindowManagerService` 才是 `Window` 的最终管理者。

## Window 的常用参数 ##
`Window` 的参数都被定义在 `WindowManager` 的静态内部类 `LayoutParams` 中，源码如下：
```java
public static class LayoutParams extends ViewGroup.LayoutParams implements Parcelable {

    /** Window左上角的X坐标 */
    public int x;

    /** Window左上角的Y坐标 */
    public int y;

    /** Window的水平方向权重 */
    public float horizontalWeight;

    /** Window的垂直方向权重 */
    public float verticalWeight;

    /** Window的类型 */
    public int type;

    /** Window的flag，用于控制Window的显示 */
    public int flags;

    /** Window软键盘输入区域的显示模式 */
    public int softInputMode;

    /** Window在屏幕中的显示位置 */
    public int gravity;

    /** Window和容器的水平间距 */
    public float horizontalMargin;

    /** Window和容器的垂直间距 */
    public float verticalMargin;

    /** Window的像素点格式，值定义在PixelFormat中 */
    public int format;

    /** Window的动画样式资源 */
    public int windowAnimations;

    /** Window的透明度，取值为0~1 */
    public float alpha = 1.0f;

    /** Window的屏幕模糊度，取值为0~1 */
    public float dimAmount = 1.0f;

    /** Window的屏幕亮度 */
    public float screenBrightness = BRIGHTNESS_OVERRIDE_NONE;

    /** Window关联的Binder */
    public IBinder token = null;

    /** Window所属应用程序包名 */
    public String packageName = null;

    /** Window的方向 */
    public int screenOrientation = ActivityInfo.SCREEN_ORIENTATION_UNSPECIFIED;

    /** Window在凹口屏幕的布局方式 */
    public int layoutInDisplayCutoutMode = LAYOUT_IN_DISPLAY_CUTOUT_MODE_DEFAULT;

}
```

## Window 的常用类型 ##
`Window` 的类型大概可以分为三类：`Application Window(应用程序窗口)`、`Sub Windwow(子窗口)`、`System Window(系统窗口)`， `Window` 的类型通过 `type` 值来表示，每个大类型又包含多个小类型，它们都定义在 `WindowManager` 的静态内部类 `LayoutParams`。源码如下：
```java
public static class LayoutParams extends ViewGroup.LayoutParams implements Parcelable {

    /** Window的类型 */
    public int type;

    /** 应用程序Window的开始值 */
    public static final int FIRST_APPLICATION_WINDOW = 1;
    /** 应用程序Window的结束值 */
    public static final int LAST_APPLICATION_WINDOW = 99;

    /** 子Window类型的开始值 */
    public static final int FIRST_SUB_WINDOW = 1000;
    /** 子Window类型的结束值 */
    public static final int LAST_SUB_WINDOW = 1999;

    /** 系统Window类型的开始值 */
    public static final int FIRST_SYSTEM_WINDOW = 2000;
    /** 系统Window类型的结束值 */
    public static final int LAST_SYSTEM_WINDOW = 2999;

}
```

| 类型                       | 值    | 备注                      | 
| :------------------------- | :---- | :------------------------ | 
| `FIRST_APPLICATION_WINDOW` | 1     | 应用程序 Window 的开始值  |
| `LAST_APPLICATION_WINDOW`  | 99    | 应用程序 Window 的结束值  |
| `FIRST_SUB_WINDOW`         | 1000  | 子 Window 类型的开始值    |
| `LAST_SUB_WINDOW`          | 1999  | 子 Window 类型的结束值    |
| `FIRST_APPLICATION_WINDOW` | 2000  | 系统 Window 类型的开始值  |
| `FIRST_APPLICATION_WINDOW` | 2999  | 系统 Window 类型的结束值  |

> 如果是层级在 **`2000(FIRST_SYSTEM_WINDOW)`** 以下的是不需要申请弹窗权限的。

### Application Window ###
`Application Window(应用程序窗口)` 的区间范围 `[1,99]`，例如：`Activity`。源码如下：
```java
public static class LayoutParams extends ViewGroup.LayoutParams implements Parcelable {

    /** 应用程序Window的开始值 */
    public static final int FIRST_APPLICATION_WINDOW = 1;

    /** 应用程序Window的基础值 */
    public static final int TYPE_BASE_APPLICATION   = 1;

    /** 普通的应用程序 */
    public static final int TYPE_APPLICATION        = 2;

    /** 特殊的应用程序窗口(在程序可以显示Window之前系统使用这个Window来显示一些东西) */
    public static final int TYPE_APPLICATION_STARTING = 3;

    /** TYPE_APPLICATION的变体(在应用程序显示之前，WindowManager会等待这个Window绘制完毕) */
    public static final int TYPE_DRAWN_APPLICATION = 4;

    /** 应用程序Window的结束值 */
    public static final int LAST_APPLICATION_WINDOW = 99;

}
```

| 类型                        | 备注                                                                                   | 
| :-------------------------- | :------------------------------------------------------------------------------------- | 
| `FIRST_APPLICATION_WINDOW`  | 应用程序 Window 的开始值                                                               |
| `TYPE_BASE_APPLICATION`     | 应用程序 Window 的基础值                                                               |
| `TYPE_APPLICATION`          | 普通的应用程序                                                                         |
| `TYPE_APPLICATION_STARTING` | 特殊的应用程序窗口(在程序可以显示 Window 之前系统使用这个 Window 来显示一些东西)       |
| `TYPE_DRAWN_APPLICATION`    | TYPE_APPLICATION 的变体(在应用程序显示之前，WindowManager 会等待这个 Window 绘制完毕)  |
| `LAST_APPLICATION_WINDOW`   | 应用程序 Window 的结束值                                                               |

### Sub Windwow ###
`Sub Window(子窗口)` 的区间范围 `[1000,1999]`，这些 `Window` 按照 `Z-order` 顺序依附于父 `Window` 上，并且他们的坐标空间相对于父 `Window` 的，例如：`PopupWindow`。源码如下：
```java
public static class LayoutParams extends ViewGroup.LayoutParams implements Parcelable {

    /** 子Window类型的开始值 */
    public static final int FIRST_SUB_WINDOW = 1000;

    /** 应用程序Window顶部的面板(这些Window出现在其附加Window的顶部) */
    public static final int TYPE_APPLICATION_PANEL = FIRST_SUB_WINDOW;

    /** 用于显示媒体(如视频)的Window(这些Window出现在其附加Window的后面) */
    public static final int TYPE_APPLICATION_MEDIA = FIRST_SUB_WINDOW + 1;

    /** 应用程序Window顶部的子面板(这些Window出现在其附加Window和任何Window的顶部) */
    public static final int TYPE_APPLICATION_SUB_PANEL = FIRST_SUB_WINDOW + 2;

    /** 类似于TYPE_APPLICATION_PANEL，当前Window的布局和顶级Window布局相同，而不是其容器的子级 */
    public static final int TYPE_APPLICATION_ATTACHED_DIALOG = FIRST_SUB_WINDOW + 3;

    /** 用显示媒体Window覆盖顶部的Window(系统隐藏的API) */
    public static final int TYPE_APPLICATION_MEDIA_OVERLAY  = FIRST_SUB_WINDOW + 4;

    /** 子面板在应用程序Window的顶部(这些Window显示在其附加Window的顶部，系统隐藏的API) */
    public static final int TYPE_APPLICATION_ABOVE_SUB_PANEL = FIRST_SUB_WINDOW + 5;

    /** 子Window类型的结束值 */
    public static final int LAST_SUB_WINDOW = 1999;

}
```

| 类型                               | 备注                                                                                   | 
| :--------------------------------- | :------------------------------------------------------------------------------------- | 
| `FIRST_SUB_WINDOW`                 | 子 Window 类型的开始值                                                                 |
| `TYPE_APPLICATION_PANEL`           | 应用程序 Window 顶部的面板(这些 Window 出现在其附加 Window 的顶部)                     |
| `TYPE_APPLICATION_MEDIA`           | 用于显示媒体(如视频)的 Window(这些 Window 出现在其附加 Window 的后面)                  |
| `TYPE_APPLICATION_SUB_PANEL`       | 应用程序 Window 顶部的子面板(这些 Window 出现在其附加 Window 和任何Window的顶部)       |
| `TYPE_APPLICATION_ATTACHED_DIALOG` | 类似于TYPE_APPLICATION_PANEL，当前Window的布局和顶级Window布局相同，而不是其容器的子级 |
| `TYPE_APPLICATION_MEDIA_OVERLAY`   | 用显示媒体 Window 覆盖顶部的 Window(系统隐藏的API)                                     |
| `TYPE_APPLICATION_ABOVE_SUB_PANEL` | 子面板在应用程序 Window 的顶部(这些 Window 显示在其附加 Window 的顶部，系统隐藏的 API) |
| `LAST_SUB_WINDOW`                  | 子 Window 类型的结束值                                                                 |


### System Window ###
`System Window(系统窗口)` 的区间范围 `[2000,2999]`，例如：`Toast`，`输入法窗口`，`系统音量条窗口`，`系统错误窗口`。源码如下：
```java
public static class LayoutParams extends ViewGroup.LayoutParams implements Parcelable {

    /** 系统Window类型的开始值 */
    public static final int FIRST_SYSTEM_WINDOW     = 2000;

    /** 系统状态栏，只能有一个状态栏，它被放置在屏幕的顶部，所有其他窗口都向下移动 */
    public static final int TYPE_STATUS_BAR         = FIRST_SYSTEM_WINDOW;

    /** 系统搜索窗口，只能有一个搜索栏，它被放置在屏幕的顶部 */
    public static final int TYPE_SEARCH_BAR         = FIRST_SYSTEM_WINDOW+1;

    /** 电话窗口，通常位于所有应用程序上方，但位于状态栏后面(对于非系统应用程序已弃用，用TYPE_APPLICATION_OVERLAY代替) */
    @Deprecated
    public static final int TYPE_PHONE              = FIRST_SYSTEM_WINDOW+2;

    /** 系统窗口，始终位于应用程序窗口的顶部(对于非系统应用程序已弃用，用TYPE_APPLICATION_OVERLAY代替) */
    @Deprecated
    public static final int TYPE_SYSTEM_ALERT       = FIRST_SYSTEM_WINDOW+3;

    /** 锁屏窗口(已经从系统中被移除，可以使用 TYPE_KEYGUARD_DIALOG 代替) */
    public static final int TYPE_KEYGUARD           = FIRST_SYSTEM_WINDOW+4;

    /** 临时通知(对于非系统应用程序已弃用，用TYPE_APPLICATION_OVERLAY代替) */
    @Deprecated
    public static final int TYPE_TOAST              = FIRST_SYSTEM_WINDOW+5;

    /** 系统叠加窗口，需要显示在其他所有窗口之上(对于非系统应用程序已弃用，用TYPE_APPLICATION_OVERLAY代替) */
    @Deprecated
    public static final int TYPE_SYSTEM_OVERLAY     = FIRST_SYSTEM_WINDOW+6;

    /** 优先电话UI，即使处于锁屏状态也需要显示(对于非系统应用程序已弃用，用TYPE_APPLICATION_OVERLAY代替) */
    @Deprecated
    public static final int TYPE_PRIORITY_PHONE     = FIRST_SYSTEM_WINDOW+7;

    /** 系统对话框窗口 */
    public static final int TYPE_SYSTEM_DIALOG      = FIRST_SYSTEM_WINDOW+8;

    /** 锁屏时显示的对话框 */
    public static final int TYPE_KEYGUARD_DIALOG    = FIRST_SYSTEM_WINDOW+9;

    /** 内部系统错误窗口，出现在所有可能的窗口顶部(对于非系统应用程序已弃用，用TYPE_APPLICATION_OVERLAY代替) */
    @Deprecated
    public static final int TYPE_SYSTEM_ERROR       = FIRST_SYSTEM_WINDOW+10;

    /** 输入法窗口，显示在普通UI上方，应用程序可重新布局以免输入焦点被此窗口覆盖 */
    public static final int TYPE_INPUT_METHOD       = FIRST_SYSTEM_WINDOW+11;

    /** 输入法对话框，显示于当前输入法窗口之上 */
    public static final int TYPE_INPUT_METHOD_DIALOG= FIRST_SYSTEM_WINDOW+12;

    /** 墙纸窗口，放置在要位于墙纸顶部的任何窗口后面 */
    public static final int TYPE_WALLPAPER          = FIRST_SYSTEM_WINDOW+13;

    /** 状态栏的滑动面板 */
    public static final int TYPE_STATUS_BAR_PANEL   = FIRST_SYSTEM_WINDOW+14;

    /** 系统叠加窗口，需要显示在其他所有窗口之上(系统隐藏的API) */
    public static final int TYPE_SECURE_SYSTEM_OVERLAY = FIRST_SYSTEM_WINDOW+15;

    /** 拖放式伪窗口(系统隐藏的API) */
    public static final int TYPE_DRAG               = FIRST_SYSTEM_WINDOW+16;

    /** 状态栏的滑动面板(系统隐藏的API) */
    public static final int TYPE_STATUS_BAR_SUB_PANEL = FIRST_SYSTEM_WINDOW+17;

    /** (鼠标)指针(系统隐藏的API) */
    public static final int TYPE_POINTER = FIRST_SYSTEM_WINDOW+18;

    /** 导航栏(系统隐藏的API) */
    public static final int TYPE_NAVIGATION_BAR = FIRST_SYSTEM_WINDOW+19;

    /** 系统音量条窗口(系统隐藏的API) */
    public static final int TYPE_VOLUME_OVERLAY = FIRST_SYSTEM_WINDOW+20;

    /** 引导进度对话框，位于所有窗口只上(系统隐藏的API) */
    public static final int TYPE_BOOT_PROGRESS = FIRST_SYSTEM_WINDOW+21;

    /** 应用程序叠加窗口，显示在所有活动窗口上方，但在关键系统窗口(如状态栏或IME)下方 */
    public static final intTYPE_APPLICATION_OVERLAY= FIRST_SYSTEM_WINDOW + 38;

    /** 系统 Window 类型的结束值 */
    public static final int LAST_SYSTEM_WINDOW      = 2999;

}
```

| 类型                         | 备注                                                                                                             | 
| :--------------------------- | :--------------------------------------------------------------------------------------------------------------- | 
| `FIRST_APPLICATION_WINDOW`   | 系统 Window 类型的开始值                                                                                         |
| `TYPE_STATUS_BAR`            | 系统状态栏，只能有一个状态栏，它被放置在屏幕的顶部，所有其他窗口都向下移动                                       |
| `TYPE_SEARCH_BAR`            | 系统搜索窗口，只能有一个搜索栏，它被放置在屏幕的顶部                                                             |
| `TYPE_PHONE`                 | 电话窗口，通常位于所有应用程序上方，但位于状态栏后面(对于非系统应用程序已弃用，用 TYPE_APPLICATION_OVERLAY 代替) |
| `TYPE_SYSTEM_ALERT`          | 系统窗口，始终位于应用程序窗口的顶部(对于非系统应用程序已弃用，用 TYPE_APPLICATION_OVERLAY 代替)                 |
| `TYPE_KEYGUARD`              | 锁屏窗口(已经从系统中被移除，可以使用 TYPE_KEYGUARD_DIALOG 代替)                                                 |
| `TYPE_TOAST`                 | 临时通知(对于非系统应用程序已弃用，用 TYPE_APPLICATION_OVERLAY 代替)                                             |
| `TYPE_SYSTEM_OVERLAY`        | 系统叠加窗口，需要显示在其他所有窗口之上(对于非系统应用程序已弃用，用 TYPE_APPLICATION_OVERLAY 代替)             |
| `TYPE_PRIORITY_PHONE`        | 优先电话UI，即使处于锁屏状态也需要显示(对于非系统应用程序已弃用，用 TYPE_APPLICATION_OVERLAY 代替)               |
| `TYPE_SYSTEM_DIALOG`         | 系统对话框窗口                                                                                                   |
| `TYPE_KEYGUARD_DIALOG`       | 锁屏时显示的对话框                                                                                               |
| `TYPE_SYSTEM_ERROR`          | 内部系统错误窗口，出现在所有可能的窗口顶部(对于非系统应用程序已弃用，用 TYPE_APPLICATION_OVERLAY 代替)           |
| `TYPE_INPUT_METHOD`          | 输入法窗口，显示在普通UI上方，应用程序可重新布局以免输入焦点被此窗口覆盖                                         |
| `TYPE_INPUT_METHOD_DIALOG`   | 输入法对话框，显示于当前输入法窗口之上                                                                           |
| `TYPE_WALLPAPER`             | 墙纸窗口，放置在要位于墙纸顶部的任何窗口后面                                                                     |
| `TYPE_STATUS_BAR_PANEL`      | 状态栏的滑动面板                                                                                                 |
| `TYPE_SECURE_SYSTEM_OVERLAY` | 系统叠加窗口，需要显示在其他所有窗口之上(系统隐藏的API)                                                          |
| `TYPE_DRAG`                  | 拖放式伪窗口(系统隐藏的API)                                                                                      |
| `TYPE_STATUS_BAR_SUB_PANEL`  | 状态栏的滑动面板(系统隐藏的API)                                                                                  |
| `TYPE_POINTER`               | (鼠标)指针(系统隐藏的API)                                                                                        |
| `TYPE_NAVIGATION_BAR`        | 导航栏(系统隐藏的API)                                                                                            |
| `TYPE_VOLUME_OVERLAY`        | 系统音量条窗口(系统隐藏的API)                                                                                    |
| `TYPE_BOOT_PROGRESS`         | 引导进度对话框，位于所有窗口只上(系统隐藏的API)                                                                  |
| `TYPE_APPLICATION_OVERLAY`   | 应用程序叠加窗口，显示在所有活动窗口上方，但在关键系统窗口(如状态栏或IME)下方                                    |
| `FIRST_APPLICATION_WINDOW`   | 系统Window类型的结束值                                                                                           |

> **注意：**
> - `TYPE_PHONE`、`TYPE_SYSTEM_ALERT`、`TYPE_TOAST`、`TYPE_SYSTEM_OVERLAY`、`TYPE_PRIORITY_PHONE`、`TYPE_SYSTEM_ERROR` 这些 `type` 在 `API 26` 中均已经过时，使用 `TYPE_APPLICATION_OVERLAY` 代替，需要申请 `Manifest.permission.SYSTEM_ALERT_WINDOW` 权限。
> - `TYPE_KEYGUARD` 已经被从系统中移除，可以使用 `TYPE_KEYGUARD_DIALOG` 来代替。

## Window 的标志位 ##
`Window` 的 `flag` 可以通过 `Window#addFlags(int)` 方法和  `Window#setFlags(int, int)` 方法设置，通过 `Window#clearFlags(int)` 方法清除。

`Window` 的 `flag` 用于控制 `Window` 的显示，它们的值也是定义在 `WindowManager` 的静态内部类 `LayoutParams` 中，源码如下：
```java
public static class LayoutParams extends ViewGroup.LayoutParams implements Parcelable {

    /** Window的flag，用于控制Window的显示 */
    public int flags;

    /** Window可见时允许锁屏 */
    public static final int FLAG_ALLOW_LOCK_WHILE_SCREEN_ON     = 0x00000001;

    /** Window后面的所有内容都会变暗 */
    public static final int FLAG_DIM_BEHIND        = 0x00000002;

    /** Window后面的内容都变模糊(API已过时) */
    @Deprecated
    public static final int FLAG_BLUR_BEHIND        = 0x00000004;

    /** Window不能获得输入焦点，即不接受任何按键或按钮事件，例如该Window上 有EditView，点击EditView 是不会弹出软键盘的；
        Window范围外的事件依旧为原窗口处理；例如点击该窗口外的View，依然会有响应。另外只要设置了此Flag，都将会启用FLAG_NOT_TOUCH_MODAL */
    public static final int FLAG_NOT_FOCUSABLE      = 0x00000008;

    /** Window将不会接受任何Touch事件 */
    public static final int FLAG_NOT_TOUCHABLE      = 0x00000010;

    /** 将Window之外的按键事件发送给后面的Window处理，而自己只会处理Window区域内的触摸事件；
        Window之外的View 也是可以响应Touch事件 */
    public static final int FLAG_NOT_TOUCH_MODAL    = 0x00000020;

    /** Window可见时设备的屏幕处于打开状态并保持明亮 */
    public static final int FLAG_KEEP_SCREEN_ON     = 0x00000080;

    /** Window占满整个屏幕，忽略边框周围的装饰(例如状态栏) */
    public static final int FLAG_LAYOUT_IN_SCREEN   = 0x00000100;

    /** 允许Window扩展到屏幕之外 */
    public static final int FLAG_LAYOUT_NO_LIMITS   = 0x00000200;

    /** 全屏显示，隐藏所有的Window装饰(例如状态栏) */
    public static final int FLAG_FULLSCREEN      = 0x00000400;

    /** 覆盖FLAG_FULLSCREEN并强制显示屏幕装饰(例如状态栏) */
    public static final int FLAG_FORCE_NOT_FULLSCREEN   = 0x00000800;

    /** 将窗口的内容视为安全的，防止其出现在屏幕截图中或在非安全的显示器上查看 */
    public static final int FLAG_SECURE             = 0x00002000;

    /** 一种特殊模式，在该模式下，布局参数用于在将表面合成到屏幕时执行缩放 */
    public static final int FLAG_SCALED             = 0x00004000;

    /** 当用户的脸贴近屏幕时(比如打电话)，不会去响应此事件 */
    public static final int FLAG_IGNORE_CHEEK_PRESSES    = 0x00008000;

    /** 仅与FLAG_LAYOUT_IN_SCREEN结合使用的特殊选项。在屏幕上请求布局时，窗口可能出现在屏幕装饰(例如状态栏)的上方或下方 */
    public static final int FLAG_LAYOUT_INSET_DECOR = 0x00010000;

    /** Window与当前输入法的交互方式 */
    public static final int FLAG_ALT_FOCUSABLE_IM = 0x00020000;

    /** 当按键动作发生在Window之外时，将接收到一个MotionEvent.ACTION_OUTSIDE事件 */
    public static final int FLAG_WATCH_OUTSIDE_TOUCH = 0x00040000;

    /** 窗口可以在锁屏的 Window 之上显示, 使用Activity#setShowWhenLocked(boolean)方法代替 */
    @Deprecated
    public static final int FLAG_SHOW_WHEN_LOCKED = 0x00080000;

    /** 要求系统壁纸显示在该Window后面，Window表面必须是半透明的，才能真正看到它背后的壁纸 */
    public static final int FLAG_SHOW_WALLPAPER = 0x00100000;

    /** Window被添加或可见时，系统便会打开屏幕，使用Activity#setTurnScreenOn(boolean)方法代替 */
    @Deprecated
    public static final int FLAG_TURN_SCREEN_ON = 0x00200000;

    /** 设置状态栏为透明并且为全屏模式 */
    public static final int FLAG_TRANSLUCENT_STATUS = 0x04000000;

    /** 设置导航栏为透明并且为全屏模式 */
    public static final int FLAG_TRANSLUCENT_NAVIGATION = 0x08000000;

    /** 表示此窗口负责绘制系统栏背景。如果设置，系统栏将以透明背景绘制，
        此Window中的相应区域将填充Window#getStatusBarColor()和Window#getNavigationBarColor()中指定的颜色。 */
    public static final int FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS = 0x80000000;

}
```

| flag                                | 备注                                                                                                                                                                                                                                                 | 
| :---------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | 
| `FLAG_ALLOW_LOCK_WHILE_SCREEN_ON`   | Window 可见时允许锁屏                                                                                                                                                                                                                                |
| `FLAG_DIM_BEHIND`                   | Window 后面的所有内容都会变暗                                                                                                                                                                                                                        |
| `FLAG_BLUR_BEHIND`                  | Window 后面的内容都变模糊(API 已过时)                                                                                                                                                                                                                |
| `FLAG_NOT_FOCUSABLE`                | Window 不能获得输入焦点，即不接受任何按键或按钮事件，例如该 Window 上 有 EditView，点击 EditView 是不会弹出软键盘的；Window 范围外的事件依旧为原窗口处理；例如点击该窗口外的View，依然会有响应。另外只要设置了此Flag，都将会启用FLAG_NOT_TOUCH_MODAL |
| `FLAG_NOT_TOUCHABLE`                | Window 将不会接受任何 Touch 事件                                                                                                                                                                                                                     |
| `FLAG_NOT_TOUCH_MODAL`              | 将 Window 之外的按键事件发送给后面的 Window 处理，而自己只会处理 Window 区域内的触摸事件；Window 之外的 View 也是可以响应 Touch 事件                                                                                                                 |
| `FLAG_KEEP_SCREEN_ON`               | Window 可见时设备的屏幕处于打开状态并保持明亮                                                                                                                                                                                                        |
| `FLAG_LAYOUT_IN_SCREEN`             | Window 占满整个屏幕，忽略边框周围的装饰(例如状态栏)                                                                                                                                                                                                  |
| `FLAG_LAYOUT_NO_LIMITS`             | 允许 Window 扩展到屏幕之外                                                                                                                                                                                                                           |
| `FLAG_FULLSCREEN`                   | 全屏显示，隐藏所有的 Window 装饰(例如状态栏)                                                                                                                                                                                                         |
| `FLAG_FORCE_NOT_FULLSCREEN`         | 覆盖 FLAG_FULLSCREEN 并强制显示屏幕装饰(例如状态栏)                                                                                                                                                                                                  |
| `FLAG_SECURE`                       | 将窗口的内容视为安全的，防止其出现在屏幕截图中或在非安全的显示器上查看                                                                                                                                                                               |
| `FLAG_SCALED`                       | 一种特殊模式，在该模式下，布局参数用于在将表面合成到屏幕时执行缩放                                                                                                                                                                                   |
| `FLAG_IGNORE_CHEEK_PRESSES`         | 当用户的脸贴近屏幕时(比如打电话)，不会去响应此事件                                                                                                                                                                                                   |
| `FLAG_LAYOUT_INSET_DECOR`           | 仅与 FLAG_LAYOUT_IN_SCREEN 结合使用的特殊选项。在屏幕上请求布局时，窗口可能出现在屏幕装饰(例如状态栏)的上方或下方                                                                                                                                    |
| `FLAG_ALT_FOCUSABLE_IM`             | Window 与当前输入法的交互方式                                                                                                                                                                                                                        |
| `FLAG_WATCH_OUTSIDE_TOUCH`          | 当按键动作发生在 Window 之外时，将接收到一个 MotionEvent.ACTION_OUTSIDE 事件                                                                                                                                                                         |
| `FLAG_SHOW_WHEN_LOCKED`             | 窗口可以在锁屏的 Window 之上显示, 使用 Activity#setShowWhenLocked(boolean) 方法代替                                                                                                                                                                  |
| `FLAG_SHOW_WALLPAPER`               | 要求系统壁纸显示在该 Window 后面，Window 表面必须是半透明的，才能真正看到它背后的壁纸                                                                                                                                                                |
| `FLAG_TURN_SCREEN_ON`               | Window 被添加或可见时，系统便会打开屏幕，使用 Activity#setTurnScreenOn(boolean) 方法代替                                                                                                                                                             |
| `FLAG_TRANSLUCENT_STATUS`           | 设置状态栏为透明并且为全屏模式                                                                                                                                                                                                                       |
| `FLAG_TRANSLUCENT_NAVIGATION`       | 设置导航栏为透明并且为全屏模式                                                                                                                                                                                                                       |
| `FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS` | 表示此窗口负责绘制系统栏背景。如果设置，系统栏将以透明背景绘制，此 Window 中的相应区域将填充 Window#getStatusBarColor() 和 Window#getNavigationBarColor() 中指定的颜色                                                                               |

## Window 的软键盘模式 ##
`Window` 的软键盘模式表示 `Window` 软键盘输入区域的显示模式，常见的情况 `Window` 的软键盘打开会占据整个屏幕。

软键盘模式(`softInputMode`) 值，与 `AndroidManifest` 中 `Activity` 的属性 `android:windowSoftInputMode` 是对应的，因此可以在 `AndroidManifest` 文件中为 `Activity` 设置 `android:windowSoftInputMode`，示例代码如下：
```xml
<activity android:windowSoftInputMode="adjustNothing" />
```

也可以在 Java 代码中通过 `Window#setSoftInputMode(int)` 方法为 `Window` 设置 `softInputMode`，示例代码如下：
```java
getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_NOTHING);
```

`Window` 的 `softInputMode` 用于控制 `Window` 软键盘输入区域的显示模式，它们的值也是定义在 `WindowManager` 的静态内部类 `LayoutParams` 中，源码如下：
```java
public static class LayoutParams extends ViewGroup.LayoutParams implements Parcelable {

    /** Window软键盘输入区域的显示模式 */
    public int softInputMode;

    /** 未指定任何状态。当窗口获得焦点时，系统可以显示或隐藏软键盘 */
    public static final int SOFT_INPUT_STATE_UNSPECIFIED = 0;

    /** 不改变软键盘的状态 */
    public static final int SOFT_INPUT_STATE_UNCHANGED = 1;

    /** 当用户进入该窗口时，隐藏软键盘 */
    public static final int SOFT_INPUT_STATE_HIDDEN = 2;

    /** 当窗口获取焦点时，隐藏软键盘 */
    public static final int SOFT_INPUT_STATE_ALWAYS_HIDDEN = 3;

    /** 当用户进入窗口时，显示软键盘 */
    public static final int SOFT_INPUT_STATE_VISIBLE = 4;

    /** 当窗口获取焦点时，显示软键盘 */
    public static final int SOFT_INPUT_STATE_ALWAYS_VISIBLE = 5;

    /** 窗口会调整大小以适应软键盘窗口 */
    public static final int SOFT_INPUT_MASK_ADJUST = 0xf0;

    /** 没有指定状态，系统会选择一个合适的状态或依赖于主题的设置 */
    public static final int SOFT_INPUT_ADJUST_UNSPECIFIED = 0x00;

    /** 当软键盘弹出时，窗口会调整大小，该模式不能与SOFT_INPUT_ADJUST_PAN结合使用；
        如果窗口的布局参数标志包含FLAG_FULLSCREEN，则将忽略这个值，窗口不会调整大小，但会保持全屏。 */
    public static final int SOFT_INPUT_ADJUST_RESIZE = 0x10;


    /** 当软键盘弹出时，窗口不需要调整大小，只需通过进行输入框平移以确保当前输入焦点可见；
        该模式不能与SOFT_INPUT_ADJUST_RESIZE结合使用 */
    public static final int SOFT_INPUT_ADJUST_PAN = 0x20;

    /** 将不会调整大小，直接覆盖在窗口上 */
    public static final int SOFT_INPUT_ADJUST_NOTHING = 0x30;

    /** 当用户转至此窗口时，由系统自动设置，当窗口显示之后该标志将自动清除 */
    public static final int SOFT_INPUT_IS_FORWARD_NAVIGATION = 0x100;

}
```

| 软键盘模式                         | xml属性            | 备注                                                                                                                                                                   | 
| :--------------------------------- | :----------------- | :-----------------------------------------------------------------------------------------------------------------------------------------------------| 
| `SOFT_INPUT_STATE_UNSPECIFIED`     | stateUnspecified   | 未指定任何状态，当窗口获得焦点时，系统可以显示或隐藏软键盘                                                                                                             |
| `SOFT_INPUT_STATE_UNCHANGED`       | stateUnchanged     | 不改变软键盘的状态                                                                                                                                                     |
| `SOFT_INPUT_STATE_HIDDEN`          | stateHidden        | 当用户进入该窗口时，隐藏软键盘                                                                                                                                         |
| `SOFT_INPUT_STATE_ALWAYS_HIDDEN`   | stateAlwaysHidden  | 当窗口获取焦点时，隐藏软键盘                                                                                                                                           |
| `SOFT_INPUT_STATE_VISIBLE`         | stateVisible       | 当用户进入窗口时，显示软键盘                                                                                                                                           |
| `SOFT_INPUT_STATE_ALWAYS_VISIBLE`  | stateAlwaysVisible | 当窗口获取焦点时，显示软键盘                                                                                                                                           |
| `SOFT_INPUT_MASK_ADJUST`           | --                 | 窗口会调整大小以适应软键盘窗口                                                                                                                                         |
| `SOFT_INPUT_ADJUST_UNSPECIFIED`    | adjustUnspecified  | 没有指定状态，系统会选择一个合适的状态或依赖于主题的设置                                                                                                               |
| `SOFT_INPUT_ADJUST_RESIZE`         | adjustResize       | 当软键盘弹出时，窗口会调整大小，该模式不能与SOFT_INPUT_ADJUST_PAN结合使用；如果窗口的布局参数标志包含FLAG_FULLSCREEN，则将忽略这个值，窗口不会调整大小，但会保持全屏。 |
| `SOFT_INPUT_ADJUST_PAN`            | adjustPan          | 当软键盘弹出时，窗口不需要调整大小，只需通过进行输入框平移以确保当前输入焦点可见；该模式不能与SOFT_INPUT_ADJUST_RESIZE结合使用。                                       |
| `SOFT_INPUT_ADJUST_NOTHING`        | adjustNothing      | 将不会调整大小，直接覆盖在窗口上                                                                                                                                       |
| `SOFT_INPUT_IS_FORWARD_NAVIGATION` | --                 | 当用户转至此窗口时，由系统自动设置，当窗口显示之后该标志将自动清除                                                                                                     |

## Window 的凹口屏幕布局方式 ##
`Window` 的 `layoutInDisplayCutoutMode` 用于控制 `Window` 在凹口屏幕上的布局方式，它们的值也是定义在 `WindowManager` 的静态内部类 `LayoutParams` 中，源码如下：
```java
public static class LayoutParams extends ViewGroup.LayoutParams implements Parcelable {

    /** Window在凹口屏幕的布局方式 */
    public int layoutInDisplayCutoutMode = LAYOUT_IN_DISPLAY_CUTOUT_MODE_DEFAULT;

    /** 默认情况下，全屏窗口不会使用到刘海区域，非全屏窗口可正常使用刘海区域 */
    public static final int LAYOUT_IN_DISPLAY_CUTOUT_MODE_DEFAULT = 0;

    /** 窗口声明使用刘海区域(已弃用，使用LAYOUT_IN_DISPLAY_CUTOUT_MODE_SHORT_EDGES代替) */
    @Deprecated
    public static final int LAYOUT_IN_DISPLAY_CUTOUT_MODE_ALWAYS = 1;

    /** 窗口声明使用刘海区域 */
    public static final int LAYOUT_IN_DISPLAY_CUTOUT_MODE_SHORT_EDGES = 1;

    /** 窗口声明不使用刘海区域 */
    public static final int LAYOUT_IN_DISPLAY_CUTOUT_MODE_NEVER = 2;

}
```

| 布局方式                                    | 备注                                                                              | 
| :------------------------------------------ | :-------------------------------------------------------------------------------- | 
| `LAYOUT_IN_DISPLAY_CUTOUT_MODE_DEFAULT`     | 默认情况下，全屏窗口不会使用到刘海区域，非全屏窗口可正常使用刘海区域              |
| `LAYOUT_IN_DISPLAY_CUTOUT_MODE_ALWAYS`      | 窗口声明使用刘海区域(已弃用，使用 LAYOUT_IN_DISPLAY_CUTOUT_MODE_SHORT_EDGES代替 ) |
| `LAYOUT_IN_DISPLAY_CUTOUT_MODE_SHORT_EDGES` | 窗口声明使用刘海区域                                                              |
| `LAYOUT_IN_DISPLAY_CUTOUT_MODE_NEVER`       | 窗口声明不使用刘海区域                                                            |

## Window 的屏幕方向 ##
`Window` 的 `screenOrientation` 用于控制 `Window` 的屏幕方向，它们的值也是定义在 `ActivityInfo` 中，源码如下：
```java
public class ActivityInfo extends ComponentInfo implements Parcelable {

    /** 未设置特定的方向值(系统隐藏API) */
    public static final int SCREEN_ORIENTATION_UNSET = -2;

    /** 未设置特定的方向值，与 unspecified 对应 */
    public static final int SCREEN_ORIENTATION_UNSPECIFIED = -1;

    /** 横屏(90度)，与 landscape 对应 */
    public static final int SCREEN_ORIENTATION_LANDSCAPE = 0;

    /** 竖屏(0度)，与 behind 对应 */
    public static final int SCREEN_ORIENTATION_PORTRAIT = 1;

    /** 屏幕方向为用户当前的首选方向，与 user 对应 */
    public static final int SCREEN_ORIENTATION_USER = 2;

    /** 屏幕方向与活动堆栈中紧接其下的活动的方向相同，与 behind 对应 */
    public static final int SCREEN_ORIENTATION_BEHIND = 3;

    /** 无论系统是否设置为自动转屏，屏幕方向跟随方向传感器，与 sensor 对应 */
    public static final int SCREEN_ORIENTATION_SENSOR = 4;

    /** 无论系统是否设置为自动转屏，屏幕方向不跟随方向传感器，与 nosensor 对应 */
    public static final int SCREEN_ORIENTATION_NOSENSOR = 5;

    /** 根据方向传感器横屏切换(90度和270度)，与 sensorLandscape 对应 */
    public static final int SCREEN_ORIENTATION_SENSOR_LANDSCAPE = 6;

    /** 根据方向传感器竖屏切换(0度和180度)，与 sensorPortrait 对应 */
    public static final int SCREEN_ORIENTATION_SENSOR_PORTRAIT = 7;

    /** 反向横屏(270度)，与 reverseLandscape 对应 */
    public static final int SCREEN_ORIENTATION_REVERSE_LANDSCAPE = 8;

    /** 反向竖屏(180度)，与 reversePortrait 对应 */
    public static final int SCREEN_ORIENTATION_REVERSE_PORTRAIT = 9;

    /** 显示的方向(4个方向)是由设备的方向传感器来决定的，除了它允许屏幕有4个显示方向之外，其他与设置为 sensor 时情况类似，与 fullSensor 对应 */
    public static final int SCREEN_ORIENTATION_FULL_SENSOR = 10;

    /** 屏幕方向为横向，根据设备传感器和用户的喜好可以为正向或反向横向，与 userLandscape 对应 */
    public static final int SCREEN_ORIENTATION_USER_LANDSCAPE = 11;

    /** 屏幕方向为纵向，根据设备传感器和用户的喜好可以为正向或反向纵向，与 userPortrait 对应 */
    public static final int SCREEN_ORIENTATION_USER_PORTRAIT = 12;

    /** 如果用户已锁定基于传感器的旋转，则其行为与用户相同，否则与 fullSensor 相同，并允许4种可能的屏幕方向中的任何一种，与 fullUser 对应 */
    public static final int SCREEN_ORIENTATION_FULL_USER = 13;

    /** 不管方向如何，都将方向锁定为其当前旋转，与 locked 对应 */
    public static final int SCREEN_ORIENTATION_LOCKED = 14;

}
```

| 屏幕方向                                | xml属性          | 备注                                                                                                              | 
| :-------------------------------------- | :--------------- | :---------------------------------------------------------------------------------------------------------------- | 
| `SCREEN_ORIENTATION_UNSET`              | --               | 未设置特定的方向值(系统隐藏API)                                                                                   |
| `SCREEN_ORIENTATION_UNSPECIFIED`        | unspecified      | 未设置特定的方向值                                                                                                |
| `SCREEN_ORIENTATION_LANDSCAPE`          | landscape        | 横屏(90度)                                                                                                        |
| `SCREEN_ORIENTATION_PORTRAIT`           | behind           | 竖屏(0度)                                                                                                         |
| `SCREEN_ORIENTATION_USER`               | user             | 屏幕方向为用户当前的首选方向                                                                                      |
| `SCREEN_ORIENTATION_BEHIND`             | behind           | 屏幕方向与活动堆栈中紧接其下的活动的方向相同                                                                      |
| `SCREEN_ORIENTATION_SENSOR`             | sensor           | 无论系统是否设置为自动转屏，屏幕方向跟随方向传感器                                                                |
| `SCREEN_ORIENTATION_NOSENSOR`           | nosensor         | 无论系统是否设置为自动转屏，屏幕方向不跟随方向传感器                                                              |
| `SCREEN_ORIENTATION_SENSOR_LANDSCAPE`   | sensorLandscape  | 根据方向传感器横屏切换(90度和270度)                                                                               |
| `SCREEN_ORIENTATION_SENSOR_PORTRAIT`    | sensorPortrait   | 根据方向传感器竖屏切换(0度和180度)                                                                                |
| `SCREEN_ORIENTATION_REVERSE_LANDSCAPE`  | reverseLandscape | 反向横屏(270度)                                                                                                   |
| `SCREEN_ORIENTATION_REVERSE_PORTRAIT`   | reversePortrait  | 反向竖屏(180度)                                                                                                   |
| `SCREEN_ORIENTATION_FULL_SENSOR`        | fullSensor       | 显示的方向(4个方向)是由设备的方向传感器来决定的，除了它允许屏幕有4个显示方向之外，其他与设置为 sensor 时情况类似  |
| `SCREEN_ORIENTATION_USER_LANDSCAPE`     | userLandscape    | 屏幕方向为横向，根据设备传感器和用户的喜好可以为正向或反向横向                                                    |
| `SCREEN_ORIENTATION_USER_PORTRAIT`      | userPortrait     | 屏幕方向为纵向，根据设备传感器和用户的喜好可以为正向或反向纵向                                                    |
| `SCREEN_ORIENTATION_FULL_USER`          | fullUser         | 如果用户已锁定基于传感器的旋转，则其行为与用户相同，否则与 fullSensor 相同，并允许4种可能的屏幕方向中的任何一种   |
| `SCREEN_ORIENTATION_LOCKED`             | locked           | 不管方向如何，都将方向锁定为其当前旋转                                                                            |

## Window 的视图层级顺序 ##
我们在手机上看的是二维的，但是实际上是三维的显示，`WindowManager` 的静态内部类 `LayoutParams` 中 包含了 `Window` 的 `x` 轴坐标和 `y` 轴坐标。

`Window` 视图层级顺序 用 `Z-order` 来表示，`Z-order` 对应着 `WindowManager.LayoutParams` 的 `type` 值，`Z-order` 可以理解为 Android 视图的层级概念，值越大越靠前，就越靠近用户。

`Z-order` 的值的计算逻辑在 `WindowState` 类中，`WindowState` 构造的时候初始化当前的 `mBaseLayer` 和 `mSubLayer`，这两个参数应该是决定 `Z-order` 的两个因素：
```java
WindowState(WindowManagerService service, Session s, IWindow c, WindowToken token,
            WindowState parentWindow, int appOp, int seq, WindowManager.LayoutParams a,
            int viewVisibility, int ownerId, boolean ownerCanAddInternalSystemWindow,
            PowerManagerWrapper powerManagerWrapper) {

    // ...

    // 判断该是否在子Window的类型范围内[1000,1999]
    if (mAttrs.type >= FIRST_SUB_WINDOW && mAttrs.type <= LAST_SUB_WINDOW) {
        // 调用getWindowLayerLw()方法返回值在[1,33]之间，根据不同类型的Window在屏幕上进行排序
        mBaseLayer = mPolicy.getWindowLayerLw(parentWindow)
                * TYPE_LAYER_MULTIPLIER + TYPE_LAYER_OFFSET;
        // mSubLayer子窗口的顺序
        // 调用getSubWindowLayerFromTypeLw())方法返回值在[-2.3]之间 ，返回子Window相对于父Window的位置
        mSubLayer = mPolicy.getSubWindowLayerFromTypeLw(a.type);

        // ...
    } else {
        mBaseLayer = mPolicy.getWindowLayerLw(this)
                * TYPE_LAYER_MULTIPLIER + TYPE_LAYER_OFFSET;
        mSubLayer = 0;

        // ...
    }

    // ...
}
```

 - `mBaseLayer` 是基础序，对应的区间范围 [1,33]；
 - `mSubLayer` 相同分组下的子 `Window` 的序，对应的区间范围 [-2.3]；
 - 判断该是否在子 `Window` 的类型范围内[1000,1999]；
 - 如果是子 `Window`，调用 `getWindowLayerLw()` 方法，计算 `mBaseLayer` 的值，返回一个用来对 `Window` 进行排序的任意整数，调用 `getSubWindowLayerFromTypeLw()` 方法，计算 `mSubLayer` 的值，返回子 `Window` 相对于父 `Window` 的位置；
 - 如果不是子 `Window`，调用 `getWindowLayerLw()` 方法，计算 `mBaseLayer` 的值，返回一个用来对 `Window` 进行排序的任意整数，`mSubLayer` 值为 0。

### 窗口主序的确定 ###
`mBaseLayer` 也被称为窗口主序，调用 `WindowManagerPolicy` 的 `getWindowLayerLw()` 方法，传入 `WindowState` 的实例 `parentWindow`，计算 `mBaseLayer` 的值：
```java
int APPLICATION_LAYER = 2;
int APPLICATION_MEDIA_SUBLAYER = -2;
int APPLICATION_MEDIA_OVERLAY_SUBLAYER = -1;
int APPLICATION_PANEL_SUBLAYER = 1;
int APPLICATION_SUB_PANEL_SUBLAYER = 2;
int APPLICATION_ABOVE_SUB_PANEL_SUBLAYER = 3;

default int getWindowLayerLw(WindowState win) {
    return getWindowLayerFromTypeLw(win.getBaseType(), win.canAddInternalSystemWindow());
}

/**
 * 根据不同类型的 Window 在屏幕上进行排序
 * 返回一个用来对窗口进行排序的任意整数，数字越小，表示的值越小
 */   
default int getWindowLayerFromTypeLw(int type, boolean canAddInternalSystemWindow) {
    // 判断是否在应用程序 Window 类型的取值范围内 [1,99]
    if (type >= FIRST_APPLICATION_WINDOW && type <= LAST_APPLICATION_WINDOW) {
        return APPLICATION_LAYER;
    }

    switch (type) {
        case TYPE_WALLPAPER:        // 壁纸，通过 window manager 删除它
            return 1;
        case TYPE_PHONE:            // 电话
            return 3;
        case TYPE_SEARCH_BAR:       // 搜索栏
            return 4;
        case TYPE_SYSTEM_DIALOG:    // 系统对话框
            return 7;
        case TYPE_TOAST:            // Toast
            return 8;
        case TYPE_INPUT_METHOD:     // 输入法
            return 15;
        case TYPE_STATUS_BAR:       // 状态栏
            return 17;
        case TYPE_KEYGUARD_DIALOG:  // 锁屏
            return 20;
        case TYPE_NAVIGATION_BAR:   // 导航栏
            return 23;
        case TYPE_POINTER:
            // the (mouse) pointer layer
            return 33;
        default:
            return APPLICATION_LAYER;
    }
}
```

根据不同类型的 `Window` 在屏幕上进行排序，返回一个用来对 `Window` 进行排序的任意整数，数字越小，表示的值越小，通过以下公式来计算它的基础序，基础序越大，`Z-order` 值越大越靠前，就越靠近用户。

### 窗口子序的确定 ###
`mSubLayer` 也被称为窗口子序，调用 `getSubWindowLayerFromTypeLw()` 方法，传入 `WindowManager.LayoutParams` 的实例 `a` 的 `type` 值，计算 `mSubLayer` 的值：
```java
int APPLICATION_LAYER = 2;
int APPLICATION_MEDIA_SUBLAYER = -2;
int APPLICATION_MEDIA_OVERLAY_SUBLAYER = -1;
int APPLICATION_PANEL_SUBLAYER = 1;
int APPLICATION_SUB_PANEL_SUBLAYER = 2;
int APPLICATION_ABOVE_SUB_PANEL_SUBLAYER = 3;

/**
 * 计算 Window 相对于父 Window 的位置
 * 返回 一个整数，正值在前面，表示在父 Window 上面，负值在后面，表示在父 Window 的下面
 */
 default int getSubWindowLayerFromTypeLw(int type) {
    switch (type) {
        case TYPE_APPLICATION_PANEL:                      // 1000
        case TYPE_APPLICATION_ATTACHED_DIALOG:            // 1003
            return APPLICATION_PANEL_SUBLAYER;            // 返回值是1
        case TYPE_APPLICATION_MEDIA:                      // 1001
            return APPLICATION_MEDIA_SUBLAYER;            // 返回值是-2
        case TYPE_APPLICATION_MEDIA_OVERLAY:              // 1004
            return APPLICATION_MEDIA_OVERLAY_SUBLAYER;    // 返回值是-1
        case TYPE_APPLICATION_SUB_PANEL:                  // 1002
            return APPLICATION_SUB_PANEL_SUBLAYER;        // 返回值是2
        case TYPE_APPLICATION_ABOVE_SUB_PANEL:            // 1005
            return APPLICATION_ABOVE_SUB_PANEL_SUBLAYER;  // 返回值是3
    }
    return 0;
}
```

计算子 `Window` 相对于父 `Window` 的位置，返回一个整数，用来描述一个窗口是否属于另外一个窗口的子窗口，或者是用来确定子窗口和父窗口之间的相对位置的，正值表示在父 `Window` 上面，负值表示在父 `Window` 的下面。




