---
title: Android 刘海屏适配指南
categories: Android
tags:
  - Android
abbrlink: 430c6e50
date: 2019-04-15 18:46:56
---

Apple 一直在引领设计的潮流，自从 iPhone X 发布之后，“刘海屏”就一直存在争议，本以为是一个美丽的错误，却造就了一时间“刘海屏”的模仿潮。目前，国内已经推出的刘海屏”手机有 OPPO R15 和 华为 P20，并且 Google 也在 I/O 大会上提高了相应的适配方案。

刘海屏指的是手机屏幕正上方由于追求极致边框而采用的一种手机解决方案，因形似刘海儿而得名。也有一些其他叫法：挖孔屏、凹口屏等，这里统一按刘海屏命名。就现在市场上的情况来说，“刘海屏”主要分成两类，一类是标准的 Android P API，另外一类就是厂商在 Android P 以下的系统，做的特殊适配。

## 适配方案 ##
对于增加了刘海屏的手机屏幕，大部分都是“切割”的区域都位于状态栏，所以就面临了三种情况。
1. 对于有状态栏的页面，不会受到刘海屏特性的影响，因为刘海屏包含在状态栏中了；
2. 全屏显示的页面，系统刘海屏方案会对应用界面做下移处理，避开刘海区显示，这时会看到刘海区域变成一条黑边，完全看不到刘海了；
3. 已经适配 Android P 应用的全屏页面可以通过谷歌提供的适配方案使用刘海区，真正做到全屏显示。

## 标准 API ##
Android P 支持最新的全面屏以及为摄像头和扬声器预留空间的凹口屏幕。 通过全新的 DisplayCutout 类，可以确定非功能区域的位置和形状，这些区域不应显示内容。 要确定这些凹口屏幕区域是否存在及其位置，请使用 getDisplayCutout() 函数。

### DisplayCutout 类 ###
`android.view.DisplayCutout` 类主要用于获取凹口位置和安全区域的位置等。主要方法如下所示：

| 方法                   | 功能描述                                                       |
|------------------------|----------------------------------------------------------------|
| `getBoundingRects()`   | 返回 Rects 的列表，每个 Rects 都是显示屏上非功能区域的边界矩形 |
| `getSafeInsetLeft()`   | 返回安全区域距离屏幕左边的距离(单位：px)                       |
| `getSafeInsetRight()`  | 返回安全区域距离屏幕右边的距离(单位：px)                       |
| `getSafeInsetTop()`    | 返回安全区域距离屏幕顶部的距离(单位：px)                       |
| `getSafeInsetBottom()` | 返回安全区域距离屏幕底部的距离(单位：px)                       |

### 凹口屏幕显示模式 ###
Android P 中新增了一个布局参数属性 layoutInDisplayCutoutMode，包含了三种不同的模式，如下所示：

| 模式                                        | 模式说明                                                                                                                    |
|---------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------|
| `LAYOUT_IN_DISPLAY_CUTOUT_MODE_DEFAULT`     | 只有当 DisplayCutout 完全包含在系统栏中时，才允许窗口延伸到 DisplayCutout 区域。否则，窗口布局不与 DisplayCutout 区域重叠。 |
| `LAYOUT_IN_DISPLAY_CUTOUT_MODE_NEVER`       | 该窗口决不允许与 DisplayCutout 区域重叠。                                                                                   |
| `LAYOUT_IN_DISPLAY_CUTOUT_MODE_SHORT_EDGES` | 该窗口始终允许延伸到屏幕短边上的 DisplayCutout 区域。                                                                       |

可以通过以下代码设置凹口屏幕显示模式：
```java
WindowManager.LayoutParams layoutParams = getWindow().getAttributes();
layoutParams.layoutInDisplayCutoutMode = WindowManager.LayoutParams.LAYOUT_IN_DISPLAY_CUTOUT_MODE_NEVER;
getWindow().setAttributes(layoutParams);
```

## 非标准 API ##
在 Android P 之前，没有标准 API。然而国产各大厂商在 Android P 之前(基本都是Android O)就用上了高档大气上档次的刘海屏，所以，这也造就了各大厂商在 Android P 之前的解决方案百花齐放。下面，我们来看下主流厂商：华为、vivo、OPPO、小米等所提供的方案。

### 华为 ###
#### 使用刘海区显示 ####
使用新增的 `meta-data` 属性 `android.notch_support`，在应用的 `AndroidManifest.xml` 中增加 `meta-data` 属性，此属性不仅可以针对 `Application` 生效，也可以对 `Activity` 配置生效。具体方式如下所示：
```xml
<meta-data android:name="android.notch_support" android:value="true" />
```

`android.notch_support` 属性不仅可以针对 `Application` 生效，也可以对 `Activity` 配置生效：
 - **`对 Application 生效`**意味着该应用的所有页面，系统都不会做竖屏场景的特殊下移或者是横屏场景的右移特殊处理。
 - **`对 Activity 生效`**意味着可以针对单个页面进行刘海屏适配，设置了该属性的 `Activity` 系统将不会做特殊处理。

#### 判断是否刘海屏 ####
通过以下代码即可知道华为手机是否是刘海屏手机，返回 true 表示是刘海屏，返回 false 表示非刘海屏：
```java
public static boolean hasNotchInScreenForEMUI(Context context) {
    boolean ret = false;
    try {
        ClassLoader cl = context.getClassLoader();
        Class HwNotchSizeUtil = cl.loadClass("com.huawei.android.util.HwNotchSizeUtil");
        Method get = HwNotchSizeUtil.getMethod("hasNotchInScreen");
        ret = (boolean) get.invoke(HwNotchSizeUtil);
    } catch (ClassNotFoundException e) {
        Log.e("test", "hasNotchInScreen ClassNotFoundException");
    } catch (NoSuchMethodException e) {
        Log.e("test", "hasNotchInScreen NoSuchMethodException");
    } catch (Exception e) {
        Log.e("test", "hasNotchInScreen Exception");
    } finally {
        return ret;
    }
}
```

#### 获取刘海尺寸 ####
通过以下代码即可获取华为手机的刘海尺寸：
```java
public static int[] getNotchSizeForEMUI(Context context) {
    int[] ret = new int[]{0, 0};
    try {
        ClassLoader cl = context.getClassLoader();
        Class HwNotchSizeUtil = cl.loadClass("com.huawei.android.util.HwNotchSizeUtil");
        Method get = HwNotchSizeUtil.getMethod("getNotchSize");
        ret = (int[]) get.invoke(HwNotchSizeUtil);
    } catch (ClassNotFoundException e) {
        Log.e("test", "getNotchSize ClassNotFoundException");
    } catch (NoSuchMethodException e) {
        Log.e("test", "getNotchSize NoSuchMethodException");
    } catch (Exception e) {
        Log.e("test", "getNotchSize Exception");
    } finally {
        return ret;
    }
}
```

### 小米 ###
#### 使用刘海区显示 ####
使用新增的 `meta-data` 属性 `notch.config`，在应用的 `AndroidManifest.xml` 中增加 `meta-data` 属性，此属性可以针对 `Application` 生效。具体方式如下所示：
```xml
<meta-data android:name="notch.config" android:value="portrait|landscape" />
```
value 的取值可以是以下4种：
 - `none：`横竖屏都不绘制耳朵区
 - `portrait：`竖屏绘制到耳朵区
 - `landscape：`横屏绘制到耳朵区
 - `portrait|landscape：`横竖屏都绘制到耳朵区

注：一旦开发者声明了 `meta-data`，系统就会优先遵从开发者的声明。

#### 判断是否刘海屏 ####
小米手机系统增加了 property `ro.miui.notch`，值为 `1` 时则是 Notch 屏手机。
```java
SystemProperties.getInt("ro.miui.notch", 0) == 1;
```

`SystemProperties` 是系统隐藏 API，所以可以通过以下代码即可知道小米手机是否是刘海屏手机，返回 true 表示是刘海屏，返回 false 表示非刘海屏：
```java
public static boolean hasNotchInScreenForMIUI() {
    int notch = 0;
    try {
        @SuppressLint("PrivateApi")
        Class<?> clz = Class.forName("android.os.SystemProperties");
        Method get = clz.getMethod("getInt", String.class, int.class);
        notch = (int) get.invoke(clz, "ro.miui.notch", 0);
    } catch (ClassNotFoundException e) {
        Log.e("test", "hasNotchInScreen ClassNotFoundException");
    } catch (NoSuchMethodException e) {
        Log.e("test", "hasNotchInScreen NoSuchMethodException");
    } catch (IllegalAccessException e) {
        Log.e("test", "hasNotchInScreen IllegalAccessException");
    } catch (InvocationTargetException e) {
        Log.e("test", "hasNotchInScreen InvocationTargetException");
    } finally {
        return notch == 1;
    }
}
```

#### 获取刘海尺寸 ####
MIUI 10 新增了获取刘海宽和高的方法，需升级至 8.6.26 开发版及以上版本。

以下是获取当前设备刘海高度的方法:
```java
public static int getNotchHeightForMIUI(Context context) {
    int resourceId = context.getResources().getIdentifier("notch_height", "dimen", "android");
    if (resourceId > 0) {
        return context.getResources().getDimensionPixelSize(resourceId);
    }
    return 0;
}
```

以下是获取当前设备刘海宽度的方法:
```java
public static int getNotchWidthForMIUI(Context context) {
    int resourceId = context.getResources().getIdentifier("notch_width", "dimen", "android");
    if (resourceId > 0) {
        return context.getResources().getDimensionPixelSize(resourceId);
    }
    return 0;
}
```

### OPPO ###
#### 判断是否刘海屏 ####
通过以下代码即可知道 OPPO 手机是否是刘海屏手机，返回 true 表示是刘海屏，返回 false 表示非刘海屏：
```java
public static boolean hasNotchInScreenForOPPO(Context context) {
    return context.getPackageManager().hasSystemFeature("com.oppo.feature.screen.heteromorphism");
}
```

#### 获取刘海尺寸 ####
OPPO 手机目前不提供接口获取刘海尺寸，目前其有刘海屏的机型尺寸规格都是统一的。不排除以后机型会有变化。其显示屏宽度为 1080px，高度为 2280px。刘海区域则都是宽度为 324px, 高度为 80px。



### VIVO ###
#### 判断是否刘海屏 ####
通过以下代码即可知道华为手机是否是刘海屏手机，返回 true 表示是刘海屏，返回 false 表示非刘海屏：
```java
public static final int VIVO_NOTCH  = 0x00000020;  // 是否有刘海
public static final int VIVO_FILLET = 0x00000008; // 是否有圆角

public static boolean hasNotchForVOIO(Context context) {
    boolean ret = false;
    try {
        ClassLoader classLoader = context.getClassLoader();
        Class FtFeature = classLoader.loadClass("android.util.FtFeature");
        Method method = FtFeature.getMethod("isFeatureSupport", int.class);
        ret = (boolean) method.invoke(FtFeature, VIVO_NOTCH);
    } catch (ClassNotFoundException e) {
        Log.e("Notch", "hasNotchAtVoio ClassNotFoundException");
    } catch (NoSuchMethodException e) {
        Log.e("Notch", "hasNotchAtVoio NoSuchMethodException");
    } catch (Exception e) {
        Log.e("Notch", "hasNotchAtVoio Exception");
    } finally {
        return ret;
    }
}
```

#### 获取刘海尺寸 ####
VIVO 手机目前不提供接口获取刘海尺寸，目前其有刘海屏的机型尺寸规格都是统一的。不排除以后机型会有变化。刘海区域则都是宽度为 100dp, 高度为 27dp。

## 参考文档 ##
[Google 官方文档](https://developer.android.com/guide/topics/display-cutout/)
[华为刘海屏手机安卓O版本适配指导](https://devcenter.huawei.com/consumer/cn/devservice/doc/50114)
[小米刘海屏水滴屏 Android O 适配](https://dev.mi.com/console/doc/detail?pId=1293)
[OPPO 凹形屏适配说明](https://open.oppomobile.com/wiki/doc#id=10159)
[VIVO 异形屏应用适配指南](https://dev.vivo.com.cn/documentCenter/doc/103)

