---
title: Activity 的生命周期和启动模式
date: 2018-11-03 11:35:35
categories: Android
tags:
  - Android
  - Activity
---

## Activity的生命周期 ##
> 生命周期和启动模式以及 IntentFilter 的匹配规则分析。

Activity 的生命周期分为两个部分：
 - 典型情况下的生命周期
 - 异常情况下的生命周期

### 典型情况下的生命周期分析 ###
 - **`onCreate：`**首次创建 Activity 时调用。 您应该在此方法中执行所有正常的静态设置 — 创建视图、将数据绑定到列表等等。始终后接 **onStart()**。
 - **`onRestart：`**在 Activity 已停止并即将再次启动前调用。始终后接 **onStart()**。
 - **`onStart：`**在 Activity 即将对用户可见之前调用。如果 Activity 转入前台，则后接 **onResume()**，如果 Activity 转入隐藏状态，则后接 **onStop()**。
 - **`onResume：`**在 Activity 即将开始与用户进行交互之前调用。此时，Activity 处于 Activity 堆栈的顶层，并具有用户输入焦点。始终后接 **onPause()**。
 - **`onPause：`**当系统即将开始继续另一个 Activity 时调用。此方法通常用于确认对持久性数据的未保存更改、停止动画以及其他可能消耗 CPU 的内容，诸如此类。它应该非常迅速地执行所需操作，因为它返回后，下一个 Activity 才能继续执行。如果 Activity 返回前台，则后接 **onResume()**，如果 Activity 转入对用户不可见状态，则后接 **onStop()**。
 - **`onStop：`**在 Activity 对用户不再可见时调用。如果 Activity 被销毁，或另一个 Activity（一个现有 Activity 或新 Activity）继续执行并将其覆盖，就可能发生这种情况。如果 Activity 恢复与用户的交互，则后接 **onRestart()**，如果 Activity 被销毁，则后接 **onDestroy()**。
 - **`onDestroy：`**在 Activity 被销毁前调用。这是 Activity 将收到的最后调用。当 Activity 结束（有人对 Activity 调用了 finish()），或系统为节省空间而暂时销毁该 Activity 实例时，可能会调用它。可以通过 **isFinishing()** 方法区分这两种情形。

![Activity 的生命周期](http://localhost:4000/medias/android/activity_lifecycle.png)

> 注意：
>  - onStart和onStop是从Activity是否可见这个角度来回调的
>  - onResum和onPause是从Activity是否在前台这个角度来回调的

### 异常情况下的生命周期分析 ###
#### 情况 1：资源相关的系统配置发生改变导致Activity被杀死并重新创建 ####
> 比如说横屏手机和竖屏手机会拿到两张不同的图片（设定了 landscape 或者 portrait 状态下的图片）。本来手机在竖屏状态，突然旋转屏幕，由于系统配置发生了变化，在默认情况下，Activity 会被销毁并且重新创建，当然我们也可以阻止系统重新创建我们的 Activity。

当系统配置发生改变后，Activity 会调用 **onPause -> onStop -> onDestroy**。

由于是异常情况终止，系统会在 `onStop` 之前调用 `onSaveInstanceState` 来保存当前 `Activity` 的状态。(与 `onPause` 没有时序关系)

当 `Activity` 被系统重新创建后，系统会调用 `onRestoreInstanceState`，把之前 `onSaveInstanceState` 方法所保存的 `Bundle` 对象作为参数同时传给 `onRestoreInstanceState` 和 `onCreate` 方法。(从时序来说，`onRestoreInstanceState` 的调用时机在 `onStart` 之后)
![异常情况下 Activity 的重建过程](http://localhost:4000/medias/android/activity_recreate.png)

而在视图方面，当 Activity 在异常情况下需要重新创建时，系统会默认为我们保存当前 Activity 的视图结构，并且在 Activity 重启后为我们恢复这些数据。

其实每个 View 都有 onSaveInstanceState 和 onRestoreInstanceState，关于保存和恢复 View 层级结构，系统的工作流程如下：
![关于保存与恢复 View 层级结构](http://localhost:4000/medias/android/activity_view.png)

> onSaveInstanceState 方法，系统只会在 Activity 即将被销毁并且有机会重新显示的情况下才会去调用它。

#### 情况 2：资源内存不足导致低优先级的Activity被杀死 ####
其实这种情况的数据存储与恢复过程与`情况 1`完全一致。

Activity的优先级情况：
 - **前台的 Activity** —— 正在和用户交互的 Activity，优先级最高
 - **可见但非前台的 Activity** —— 比如 Activity 中弹出了一个对话框，导致 Activity 可见但是位于后台，无法和用户进行直接交互
 - **后台的 Activity** —— 已经被暂停的 Activity，比如执行了 onStop，优先级最低

当系统内存不足时，系统就会按照上述优先级去杀死目标 `Activity` 所在的进程，并在后续通过 `onSaveInstanceState` 和 `onRestoreInstanceState` 来存储和恢复数据。而将后台工作放入 `Service` 中是一个比较好的方法。

### 当系统配置改变后 Activity 如何不被重新创建 ###
由于系统配置中有很多内容，如果当某项内容发生改变后，不想系统重新创建 `Activity`，可以给 `Activity` 指定 `configChanges` 属性：
```xml
   android:configChanges="orientation|keyboardHidden"
```

| 参数                 | 含义                                               |
|----------------------|----------------------------------------------------|
| `mcc`                | SIM卡唯一标识IMSI(国际移动用户识别码)中的国家代码，由3位数组成，中国为460.此项 标识mcc代码发生了改变 |
| `mnc`                | SIM卡唯一标识IMSI(国际移动用户识别码)中的运营商代码，由两位数字组成，中国移动TD系统为00，中国联通为01，中国电信为03。此项标识mnc发生改变 |
| `locale`             | 设备的本地位置发生了改变们一般指切换了系统语言 |
| `touchscreen`        | 触摸屏发生了改变，正常情况下无法发生，可以忽略它 |
| `keyboard`           | 键盘类型发生了改变，比如用户使用了外插键盘 |
| `keyboardHidden`     | 键盘的可访问性发生了改变，比如用户调出了键盘 |
| `navigation`         | 系统导航方式发生了改变，比如采用了轨迹球导航，很难发生，可以忽略 |
| `screenLayout`       | 屏幕布局发生了改变，很可能是用户激活了另一个显示设备 |
| `fontScale`          | 系统字体缩放比如发生了改变，比如用户选择了一个新字号 |
| `uiMode`             | 用户界面模式发生了改变，比如是否开启了夜间模式（API8新添加） |
| `orientation`        | 屏幕方向发生了改变，这个是最常用的，比如旋转了手机屏幕 |
| `screenSize`         | 当屏幕的尺寸信息发生了改变，当旋转设备屏幕时，屏幕尺寸会发生变化，这个选项比较特殊，它和编译选项有关，当编译选项中的minSdkVersion和targetSdkVersion 均低于13时，此选项不会导致Activity重启，否则会导致Activity重启（API13新添加） |
| `smallestScreenSize` | 设备的物理屏幕尺寸发生了改变，这个项目和屏幕的方向没有关系，仅仅表示在实际的物理屏幕的尺寸改变的时候发生，比如用户切换到了外部的显示设备，这个选项和screenSize一样，当编译选项中的minSdkVersion和targetSdkVersion均低于13时，此选项不会导致Activity重启，否则会导致Activity重启（API13新添加） |
| `layoutDirection`    | 当布局方向发生变化，这个属性用的比较少，正常情况下无须修改布局的layoutDirection属性（API17新添加） |

如果我们没有在 `Activity` 的 `configChanges` 属性中指定该选项的话，当配置发生改变后就会导致 Activity 重新创建。

最常用的只有 `locale`、`orientation` 和 `keyboardHidden`。需要修改的代码很简单，只需要在 `AndroidMenifest.xml` 中加入 `Activity` 的声明即可：
```xml
<activity
    android:name="com.dimon.MainActivity"
    android:configChanges="orientation|screenSize"
    android:label="@string/app_name">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
```

```java
@Override
public void onConfigurationChanged(Configuration newConfig){
    super.onConfigurationChanged(newConfig);
  Log.d(TAG,"onConfigurationChanged,newOrientation:" + newConfig.orientation);
}
```

`Activity` 没有重新创建，并且没有调用 `onSaveInstanceState` 和 `onRestoreInstanceState` 来存储和恢复数据，而是系统调用了 `Activity` 的 `onConfigurationChanged` 方法，这个时候我们可以加入一些自己的特殊处理了。

## Activity的启动模式 ##
### Activity 的 LaunchMode ###
> 复习一点：启动 Activity 时，系统会创造实例并把他们放入任务栈里，而任务栈是一种“后进先出”的栈结构。

Activity的四种启动模式：
 - **`standard：`**标准模式、默认模式。每次启动一个Activity都会重新创建一个新的实例，不管这个实例是否已经存在。在这种模式下，某个Activity启动了一号Activity，那么一号Activity就运行在启动它的那个Activity所在的栈中。
 - **`singleTop：`**栈顶复用模式。如果新的Activity已经位于任务栈的栈顶，那么此Activity就不会被重新创建，同时它的onNewIntent方法会被回调，并且可以根据此方法的参数获得当前请求的信息。
 - **`singleTask：`**栈内复用模式。在这种单实例模式下，只要Activity在一个栈中存在，那么多次启动此Activity都不会重新创建实例，系统也会调用其onNewIntent。
 - **`singleInstance：`**单实例模式。这是一种加强的singleTask模式，除了具有singleTask模式的所有特性外，还加强了一点，那就是具体此种模式的Activity只能单独地位于一个任务栈中。

> 注：在任何跳转的时候，首先调用本 Activity 的 `onPause`，然后跳转。如果被跳转的 Activity 由于启动方式而没创建新的实例，则会先调用 `onNewIntent` ，然后按照正常的生命周期调用。

如：
 - A→B，A：onPause；B：onCreate，onStart，onResume。
 - A(singleTop)→A，A：onPause；A：onSaveInstanceState；A：onResume。

### 一些具体问题与情况 ###
1. 首先要说明：任务栈分为前台任务栈和后台任务栈，后台任务栈中的 Activity 位于暂停状态。singleTask 模式的 Activity 切换到栈顶会导致在它之上的栈内的 Activity 出栈。
2. TaskAffinity：任务相关性。标识一个 Activity 所需要的任务栈的名字。
```xml
adnroid:taskAffinity="com.dimon.task1"
```
默认情况下 Activity 所需要的任务栈的名字为应用的包名。
![TaskAffinity](http://localhost:4000/medias/android/activity_taskaffinity.png)

### 给 Activity 指定启动模式 ###
 - 方法一：通过 AndroidMenifest 为 Activity 指定启动模式
```xml
<activity
    android:name="com.dimon.SecondActivity"
    android:configChanges="screenLayout"
    adnroid:taskAffinity="com.dimon.task1"
    android:launchMode="singleTask"
    android:label="@string/app_name"/>
```

 - 方法二：通过 Intent 中设置标志位为 Activity 指定启动模式
```java
Intent intent = new Intent();
intent.setClass(MainActivity.this, SecondActivity.class);
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
startActivity(intent);
```

> 区别：
> 优先级：第二种方法比第一种优先级高，两种都存在时，以第二种为准。
> 限定范围：第一种方法比无法设置 `FLAG_ACTIVITY_CLEAR_TOP` 标识，而第二种方法比无法指定 `singleInstance` 模式。

### Acticity 中常用的 Flags ###
 - **`FLAG_ACTIVITY_NEW_TASK：`**这个标记位的作用是为 Activity 指定 `singleTask` 启动模式，其效果和在 XML 中指定该启动模式相同。
 - **`FLAG_ACTIVITY_SINGLE_TOP：`**这个标记位的作用是为 Activity 指定 `singleTop` 启动模式，其效果和在 XML 中指定该启动模式相同。
 - **`FLAG_ACTIVITY_CLEAR_TOP：`**具有次标记位的 Activity，当它启动时，在同一个任务栈中所有位于它上面的 Activity 都要出栈，这个标记位一般会和 `singleTask` 启动模式一起出现。如果被启动的 Activity 的实例已经存在，那么系统就会调用它的 `onNewIntent`。如果被启动的Activity采用了 `standard` 启动模式，那么它以及它之上的 Activity 都要出栈，系统会创建新的 Activity 实例并放入栈顶。
 - **`FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS：`**具有这个标记的 Activity 不会出现在历史 Activity 的列表中，当某种情况下我们不希望用户通过历史列表回到我们的 Activity 的时候这个标记比较有用。它等同于在 XML 中指定 Activity 的属性 `android:excludeFromRecents="true"`。

### IntentFilter 的匹配规则 ###
Intent 解析机制主要是通过查找已注册在 `AndroidManifest.xml` 中的所有 `IntentFilter` 及其中定义的 `Intent`，最终找到匹配的 `Intent`。在这个解析过程中，Android 是通过 Intent 的 `action`、`type`、`category` 这三个属性来进行匹配判断的。一个过滤列表中的 `action`、`type`、`category` 可以有多个，所有的 `action`、`type`、`category` 分别构成不同类别，同一类别信息共同约束当前类别的匹配过程。只有一个 Intent **同时匹配 action、type、category** 这三个类别才算完全匹配，**只有完全匹配才能启动 Activity**。另外一个组件若声明了多个 Intent Filter，只需要匹配任意一个即可启动该组件。 

#### action 的匹配规则 ####
action 是一个字符串，如果 Intent 指明定了 action，则目标组件的 IntentFilter 的 action 列表中就必须包含有这个 action，否则不能匹配。一个 Intent Filter 中可声明多个 action，Intent 中的 action 与其中的任一个 action 在字符串形式上完全相同（`注意，区分大小写，大小写不同但字符串内容相同也会造成匹配失败`），action 方面就匹配成功。可通过 Intent 的 setAction 方法为 Intent 设置 action，也可在构造 Intent 时传入 action。需要注意的是，隐式 Intent 必须指定 action。

Android 系统预定义了许多 action，这些 action 代表了一些常见的操作。常见action如下（Intent类中的常量）：
```java
Intent.ACTION_VIEW
Intent.ACTION_DIAL
Intent.ACTION_SENDTO
Intent.ACTION_SEND
Intent.ACTION_WEB_SEARCH
```

#### category 的匹配规则 ####
category 也是一个字符串，但是它与 action 的过滤规则不同，它要求 Intent 中如果含有 category，那么所有的 category 都必须和过滤规则中的其中一个 category 相同。也就是说，Intent 中如果出现了 category，不管有几个 category，对于每个 category 来说，它必须是过滤规则中的定义了的 category。当然，Intent 中也可以没有 category（`若Intent中未指定category，系统会自动为它带上“android.intent.category.DEFAULT”`），如果没有，仍然可以匹配成功。category 和 action 的区别在于，action 要求 Intent 中必须有一个 action 且必须和过滤规则中的某几个 action 相同，而 category 要求 Intent 可以没有 category，但是一旦发现存在 category，不论你有多少，每个都要能够和过滤规则中的任何一个 category 相同。我们可以通过 addCategory 方法为 Intent 添加 category。
特别说明：
```xml
<intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
```
这二者共同出现，标明该 Activity 是一个入口 Activity，并且会出现在系统应用列表中，二者缺一不可。

#### data 的匹配规则 ####
如果 Intent 没有提供 type，系统将从 data 中得到数据类型。同 action 类似，只要 Intent 的 data 只要与 Intent Filter 中的任一个 data 声明完全相同，data 方面就完全匹配成功。

data由两部分组成：`mimeType` 和 `URI`。
 - mimeType：媒体类型，例如 imgage/jpeg、auto/mpeg4 和 viedo/* 等，可以表示图片、文本、视频等不同的媒体格式。
 - uri：由 scheme、host、port、path | pathPattern | pathPrefix 这 4 部分组成。

`URI` 的结构如下：
```
<scheme>://<host>:<port>/[<path>|<pathPrefix>|<pathPattern >]
```
 - scheme：URI 的模式，比如 http、https、file、content 等。
 - host：URI 的主机名，比如 www.baidu.com 。
 - port：URI 的端口号。
 - path、pathPattern 和 pathPrefix：这三个表示路径信息。path：用来匹配完整的路径；pathPrefix：用来匹配路径的前缀部分；pathPattern：用表达式来匹配整个路径。

### IntentFilter 常见问题汇总 ###
#### path、pathPattern 和 pathPrefix 的区别 ####
 - path：用来匹配完整的路径，如：http://example.com/blog/abc.html ，这里将 path 设置为 /blog/abc.html 才能够进行匹配；
 - pathPrefix：用来匹配路径的前缀部分，拿上来的 Uri 来说，这里将 pathPrefix 设置为 /blog 就能进行匹配了；
 - pathPattern：用表达式来匹配整个路径，但是它里面可以包括通配符 `*`，注意正则表达式。

> 匹配符号：
    `*` 用来匹配0次或更多，如：`a*` 可以匹配“a”、“aa”、“aaa”...
    `.` 用来匹配任意字符，如：`.` 可以匹配“a”、“b”，“c”...
    `.*` 用来匹配任意字符0次或更多，如：`.*html` 可以匹配 “abchtml”、“chtml”，“html”，“sdf.html”...

> 转义：因为当读取 Xml 的时候，“\” 是被当作转义字符的（当它被用作 pathPattern 转义之前），因此这里需要两次转义，读取 Xml 是一次，在 pathPattern 中使用又是一次。如：“*” 这个字符就应该写成 “\\*”，“\” 这个字符就应该写成 “\\\\”。

#### 查询是否有 Activity 可以匹配指定 Intent 的组件 ####
 - 采用 `PackageManager` 的 `resolveActivity` 或者 `Intent` 的 `resolveActivity` 方法会获得`最适合 Intent 的一个 Activity`。
 - 调用 `PackageManager` 的 `queryIntentActivities` 会返回`所有成功匹配 Intent 的 Activity`。

#### android.intent.action.MAIN 与 android.intent.category.LAUNCHER 的区别 ####
 - android.intent.action.MAIN：决定应用程序的入口 Activity，也就是决定一个应用程序最先启动那个组件。 
 - android.intent.category.LAUNCHER：决定应用程序是否被列入系统的启动器，也就是说是否在桌面上显示一个图标。Launcher 是安卓系统中的桌面启动器，是桌面UI的统称。

这两个属性组合情况：
 - 第一种情况：有MAIN,无LAUNCHER，程序列表中无图标。原因：android.intent.category.LAUNCHER 决定应用程序是否显示在程序列表里。
 - 第二种情况：无MAIN,有LAUNCHER，程序列表中无图标。原因：android.intent.action.MAIN 决定应用程序最先启动的 Activity，如果没有 Main，则不知启动哪个 Activity，所以也不会有图标出现。

所以这两个属性一般成对出现。

> 如果一个应用中有两个组件 intent-filter 都添加了 android.intent.action.MAIN 和 android.intent.category.LAUNCHER 这两个属性，则这个应用将会显示两个图标，写在前面的组件先运行。 

#### intent-filter 匹配优先级 ####
首先查看 Intent 的过滤器(intent-filter)，按照以下优先关系查找：**`action->data->category`**。

