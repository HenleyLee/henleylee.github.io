---
title: Android 状态栏、导航栏相关
date: 2018-07-20 15:55:35
categories: Android
tags:
  - Android
---

## 1.状态栏 ##
#### 1.获取状态栏高度 ####
```java
    /**
     * 获得状态栏的高度(单位：px)
     */
    public static int getStatusBarHeight(Context context) {
        Resources resources = context.getResources();
        int resourceId = resources.getIdentifier("status_bar_height", "dimen", "android");
        return resources.getDimensionPixelSize(resourceId);
    }
```

## 2.导航栏 ##
#### 2.1.判断是否存在导航栏 ####
```java
    /**
     * 检测设备是否有底部导航栏
     */
    @TargetApi(Build.VERSION_CODES.ICE_CREAM_SANDWICH)
    public static boolean hasNavigationBar(Context context) {
        //通过判断设备是否有返回键、菜单键(不是虚拟键,是手机屏幕外的按键)来确定是否有navigation bar
        boolean hasMenuKey = ViewConfiguration.get(context).hasPermanentMenuKey();
        boolean hasBackKey = KeyCharacterMap.deviceHasKey(KeyEvent.KEYCODE_BACK);
        if (!hasMenuKey && !hasBackKey) {
            return true;// 做任何你需要做的,这个设备有一个导航栏
        }
        return false;
    }
```
#### 2.2.获取导航栏高度 ####
```java
    /**
     * 获得导航栏的高度(单位：px)
     */
    public static int getNavigationBarHeight(Context context) {
        Resources resources = context.getResources();
        int resourceId = resources.getIdentifier("navigation_bar_height", "dimen", "android");
        return resources.getDimensionPixelSize(resourceId);
    }
```
