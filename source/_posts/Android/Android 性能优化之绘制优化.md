---
title: Android 性能优化之绘制优化
categories: Android
tags:
  - Android
  - 性能优化
abbrlink: 2e33cf5d
date: 2019-01-25 18:32:50
---

绘制性能的好坏 主要影响 ：Android应用中的页面显示速度
## 优化思路 ##
绘制影响 Android 性能的实质就是影响页面的`绘制`时间。
> 一个页面通过递归完成测量和绘制过程，即 `measure`、`layout` 过程。

因此可以从以下几个方面进行优化，从而提高 Android 应用中的页面显示速度：
 - 降低 View.onDraw() 的复杂度
 - 避免过度绘制(Overdraw)

## 优化方案 ##
### 降低 View.onDraw() 的复杂度 ###
#### 不要创建新的局部对象 ####
onDraw() 方法可能会被频繁调用。如果 onDraw() 方法内需创建局部对象，则会在瞬间产生大量的历史对象，这使得占用过多内存并导致系统频繁 GC，降低了程序的执行效率。因此，onDraw() 方法中不要创建新的局部对象。

#### 避免执行大量耗时操作 ####
Google 官方性能优化标准要求，View 的最佳绘制频率为 60 fps(即：要求每帧绘制时间≤16ms)。如果 onDraw() 方法内执行大量耗时操作，会抢占 CPU 的时间片，从而导致 View 的绘制过程不流畅。因此，onDraw() 方法中避免执行大量耗时操作。

### 避免过度绘制(Overdraw) ###
#### 过度绘制的简介 ####
过度绘制是指屏幕上的某一像素点在同一帧中被重复绘制多次。

过度绘制发生的主要原因是因为多层次或重叠的 UI 结构。
> 在多层次或重叠的 UI 结构里，若不可见的 UI 也在做绘制操作，则会导致某些像素区域被绘制多次。过度绘制则会导致屏幕显示的色块不同。

过度绘制的主要影响如下：
 - 界面显示时，浪费资源去渲染不必要、看不见的背景；
 - 多次绘制就会导致页面加载或滑动时的不流畅、掉帧(对用户体验来说就是 APP 卡顿)。

#### 过度绘制的检测 ####
 - 方法一：通过开发者选项开启 GPU 过度绘制调试
> Android 手机的开发者选项中有`调试 GPU 过度绘制`的选项，打开步骤为`设置 -> 开发者选项 -> 调试GPU过度绘制 -> 显示GPU过度绘制`。

 - 方法二：通过 adb 命令开启 GPU 过度绘制调试
开启调试 GPU 过度绘制的命令如下：
```shell
$ adb shell setprop debug.hwui.overdraw show
```
关闭调试 GPU 过度绘制的命令如下：
```shell
$ adb shell setprop debug.hwui.overdraw false
```
> 执行命令之后可能需要重新启动要调试的应用。

#### 过度绘制的表现形式 ####
过度绘制会导致屏幕显示的色块不同，具体如下图所示：
![过度绘制的表现形式](https://henleylee.github.io/medias/android/over_draw.png)

依据过度绘制的层度可以分成：
 - 无过度绘制：一个像素只被绘制了一次
 - 过度绘制x1：一个像素被绘制了两次
 - 过度绘制x2：一个像素被绘制了三次
 - 过度绘制x3：一个像素被绘制了四次
 - 过度绘制x4+：一个像素被绘制了五次及以上

#### 过度绘制的优化原则 ####
很多过度绘制是难以避免的(如背景+文字导致的过度绘制)，只能尽可能避免过度绘制：
 - 尽可能地控制过度绘制的次数为 2 次(绿色)以下，蓝色最理想；
 - 尽可能避免过度绘制的粉色和红色情况；
 - 不允许 3 次以上的过度绘制(淡红色)面积超过屏幕大小的 1/4。

#### 过度绘制的优化方案 ####
 - 优化方案1： 移除 Window 的默认背景
查看 Android 源码里的 `Theme` 主题，如下：
```xml
<style name="Theme">
    ...
    <!-- Window attributes -->
    <item name="windowBackground">@drawable/screen_background_selector_dark</item>
    ...
</style>
```
也就是说继承 `Theme` 这个 style 的风格，默认情况下，新建一个 Activity 都是有背景的。一般情况下，该默认的 Window 背景基本用不上：因背景都自定义设置；若不移除，则导致所有界面都多 1 次绘制。
使用时只要在自己的 AppTheme 里面去除该背景色即可：
```xml
<style name="AppTheme" parent="android:Theme.Light.NoTitleBar">
    <item name="android:windowBackground">@null</item>
</style>
```
或者在 Activity 的 onCreate() 方法中将背景置为 null 即可：
```java
getWindow().setBackgroundDrawable(null);
```

 - 优化方案2：移除控件中不必要的背景
   - 场景1：ListView 与 Item：列表页(ListView)与其内子控件(Item)的背景相同，则可移除子控件(Item)布局中的背景；
   - 场景2：ViewPager 与 Fragment：对于一个 ViewPager 和多个 Fragment 组成的首页界面，若每个 Fragment 都设有背景色，则可移除 ViewPager 的背景。

 - 优化方案3：减少布局文件的层级
> 原理：减少不必要的嵌套(UI层级少)，则过度绘制的可能性低。
> 优化方式：使用<merge>标签并合适选择布局类型。

 - 优化方案4：使用 Canvas 的 clipRect 和 clipPath 方法限制 View 的绘制区域
> 作用：使用 Canvas 的 clipRect 和 clipPath 方法给 Canvas 设置一个裁剪区域，只有在该区域内才会被绘制，区域之外的都不绘制(如：android.support.v4.widget.DrawerLayout)。

 - 优化方案5：ImageView 的 background 和 imageDrawable 重叠
 >  Android 中，所有的 View 均可以设置 background。ImageView 除了能够设置 background 之外，还能设置 ImageDrawable。开发中，很多时候需要显示图片，在图片加载出来之前通常是需要显示一张默认图片的，很多时候会使用 ImageView 的 background 属性来设置默认背景图，而 imageDrawable 来设置需要加载的图片。这样会导致一个问题，当图片加载到页面后，默认背景图被挡住了，但是却仍然然需要绘制，导致过度绘制情况的发生。解决方案是把背景图和真正加载的图片都通过 imageDrawable 方法进行设置。

