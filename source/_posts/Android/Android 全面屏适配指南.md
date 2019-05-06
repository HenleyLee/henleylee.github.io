---
title: Android 全面屏适配指南
categories: Android
tags:
  - Android
abbrlink: 62291f67
date: 2019-04-13 10:52:36
---

全面屏是手机业界对于超高屏占比手机设计的一个宽泛的定义。从字面上解释就是，手机的正面全部都是屏幕，四个边框位置都是采用无边框设计，追求接近 100% 的屏占比。但受限于目前的技术，还不能做到手机正面屏占比 100% 的手机。现在业内所说的全面屏手机是指真实屏占比可以达到 80% 以上，拥有超窄边框设计的手机。

全面屏手机屏幕的宽高比例比较特殊，不再是以前的 `16:9` 了。比如三星的 Galaxy S8 屏幕分辨率是：2960×1440，对应的屏幕比例为 18.5:9。VIVO X20 手机屏幕分辨率是 2160x1080，对应的屏幕比例为 18:9。对于这种奇葩的屏幕比例，Android 开发者该如何去优化自己的应用，才能在这些手机上显示的更加完美呢？下面，从以下方面来探究 APP 完美适配全面屏手机的方法。

## 声明最大屏幕高宽比 ##
由于全面屏手机的高宽比比之前大，如果不适配的话，Android 默认为最大的宽高比是 1.86，小于全面屏手机的宽高比，因此，在全面屏手机上打开部分 APP 时，上下就会留有空间，显示为黑条。这样非常影响视觉体验，另外全面屏提供的额外空间也没有得以利用，因此，这样的应用需要做相关适配。

在 `Android 7.0(API level 24)`及更高版本中 Google 默认支持了多窗口模式，即 `Manifest` 文件中配置 `Activity` 的 `android:resizeableActivity` 默认属性为 `true`，在这种情况下并不需要配置 android:MaxAspectRatio 即可自动适配全面屏，还可以为整个应用或特定 Activity 明确设置 `android:resizeableActivity="true"` 属性。

如果不希望自己的应用或 `Activity` 在多窗口模式下运行，请设置 `android:resizeableActivity="false"`。在这种情况下，应用会始终全屏显示。系统会根据 Android 操作系统级别控制完成此操作的方式：
 - 如果您的应用定位到 Android 8.0(API level 26)及更高版本，它会根据其布局填充整个屏幕。
 - 如果您的应用定位到 Android 7.1(API level 25)及更低版本，则系统会将应用界面的大小限制为长宽比为 16:9(约为 1.86)的窗口。 如果应用在具有较大屏幕长宽比的设备上运行，则该应用会在带黑边的 16:9 窗口中显示，从而使部分屏幕处于未占用状态。

对于这种情况就需要考虑适配了，目前有以下两种解决方案：
 - 设置最大长宽比：
   - Android 8.0(API level 26)及更高版本设置最大长宽比，可以在 `<activity>` 标签中使用 `android:maxAspectRatio` 声明最大比例。声明 `2.4` 的最大长宽比的示例代码如下所示：
```xml
<!-- Render on full screen up to screen aspect ratio of 2.4 -->
<!-- Use a letterbox on screens larger than 2.4 -->
<activity android:maxAspectRatio="2.4">
    ...
</activity>
```
   - Android 7.1(API level 25)及更低版本设置最大长宽比，可以在 请在 `<application>` 标签中添加一个名为 `android.max_aspect` 的 `<meta-data>` 元素，如下所示：
```xml
<!-- Render on full screen up to screen aspect ratio of 2.4 -->
<!-- Use a letterbox on screens larger than 2.4 -->
<meta-data android:name="android.max_aspect" android:value="2.4" />
```

 > 如果设置了最大长宽比，请勿忘记同时设置 `android:resizeableActivity="false"`。否则，最大长宽比没有任何作用。

 - 设置支持多窗口模式：
在 `Android 7.0(API level 24)`及更高版本中 Google 默认支持了多窗口模式，即 `Manifest` 文件中配置 `Activity` 的 `android:resizeableActivity` 默认属性为 `true`。
```xml
<application
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:supportsRtl="true"
    android:theme="@style/AppTheme"
    android:resizeableActivity="true">
    <activity android:name=".MainActivity">
        <intent-filter>
            <action android:name="android.intent.action.MAIN"/>
            <category android:name="android.intent.category.LAUNCHER"/>
        </intent-filter>
    </activity>
</application>
```

## UI 适配 ##
通过增加上面适配方案提到的配置，应用在全面屏手机上就能够默认全屏显示了，但是为了避免出现 UI 异常的问题，还是需要应用自己做一些额外的 UI 适配工作：
1. 对于列表形式的应用，如：微信、网易新闻等，只是显示的内容变多，基本无影响。
2. 对于整屏的应用，应用为了保证多种屏幕的适配，需遵循 Google 的适配建议，可以参考 [Google 官网](https://developer.android.com/guide/practices/screens_support.html)中的最佳做法章节进行修改适配。
3. 对于使用整幅图片作为背景时需注意图片的填充方式，否则可能会无法填充整个屏幕。如：使用背景是用 ImageView 建议将其 scaleType 设置为 CENTER_CROP 或者使用 .9.png 图片。

## 虚拟导航键优化 ##
为了实现更高的屏占比，屏幕内的虚拟导航键就成了标准功能，如何让其应用界面在视觉上统一，同样需要开发者的积极适配。Android 已经有相关接口允许开发者自定义虚拟键的样式。关于使用哪种样式，有以下建议:
1. 如果页面含有复杂背景/纹理，建议设置为透明。
2. 含底部 Tab 栏的页面，建议将虚拟键设置为底部 Tab 栏的颜色。
3. 不含底部 Tab 栏的页面，建议使用背景颜色。

> 由于一个应用内含有多种不同的页面，希望开发者能当前页面的情况，来选择合适的虚拟键样式，以保证视觉的统一美观。

Android 有标准的实现方式，调用以下接口即可 [window.setNavigationBarColor(int color)][1]。在调用该接口时，还需要设置一些 flag，详见该接口的注释说明：
```java
/**
 * Sets the color of the navigation bar to {@param color}.
 *
 * For this to take effect,
 * the window must be drawing the system bar backgrounds with
 * {@link android.view.WindowManager.LayoutParams#FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS} and
 * {@link android.view.WindowManager.LayoutParams#FLAG_TRANSLUCENT_NAVIGATION} must not be set.
 *
 * If {@param color} is not opaque, consider setting
 * {@link android.view.View#SYSTEM_UI_FLAG_LAYOUT_STABLE} and
 * {@link android.view.View#SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION}.
 * <p>
 * The transitionName for the view background will be "android:navigation:background".
 * </p>
 * @attr ref android.R.styleable#Window_navigationBarColor
 */
public abstract void setNavigationBarColor(@ColorInt int color);
```

除了调用该方法外，还可以通过在主题中添加 [android:navigationBarColor][2] 属性来实现：
```xml
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
    <item name="colorPrimary">@color/colorPrimary</item>
    <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
    <item name="colorAccent">@color/colorAccent</item>
    <item name="android:navigationBarColor">@color/navigationBarColor</item>
</style>
```

## 参考文档 ##
[Android 官方文档](https://developer.android.com/guide/practices/screens-distribution)
[小米全面屏适配说明](https://dev.mi.com/console/doc/detail?pId=1160)
[华为全面屏适配技术指导](https://developer.huawei.com/consumer/cn/devservice/doc/50111)

[1]: https://developer.android.com/reference/android/view/Window.html#setNavigationBarColor(int)
[2]: https://developer.android.com/reference/android/view/Window.html#attr_android:navigationBarColor

