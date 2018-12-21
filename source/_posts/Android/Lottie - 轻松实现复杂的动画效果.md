---
title: Lottie - 轻松实现复杂的动画效果
categories: Android
tags:
  - Android
  - Lottie
  - 动画
abbrlink: 238b7a36
date: 2018-08-23 18:56:26
---

## 1. Lottie 介绍 ##
Lottie 是 [Airbnb](https://github.com/airbnb) 开源的一套跨平台的完整的动画效果解决方案，设计师可以使用 [Adobe After Effects](https://www.adobe.com/cn/products/aftereffects.html) 设计出漂亮的动画之后，使用 `Lottic` 提供的 [Bodymovin](https://github.com/airbnb/lottie-web) 插件将设计好的动画导出成 JSON 格式，就可以直接运用在 `iOS`、`Android`、`Web` 和 `React Native`之上，无需其他额外操作。

![](https://user-gold-cdn.xitu.io/2018/9/3/1659ed59b2e5b3f2?w=1308&h=358&f=png&s=42684)

Lottie 相关网站：
 - [Lottie 官网](http://airbnb.io/lottie/)
 - [Lottie on Github](https://github.com/airbnb)
 - [Lottie for Android](https://github.com/airbnb/lottie-android)
 - [Lottie for iOS](https://github.com/airbnb/lottie-ios)
 - [Lottie for Web](https://github.com/airbnb/lottie-web)
 - [Lottie for React Native](https://github.com/airbnb/lottie-react-native)

## 2. Lottie 使用 ##
![](https://user-gold-cdn.xitu.io/2018/8/3/164ff072f3b9c346?w=770&h=385&f=gif&s=347496)
![](https://user-gold-cdn.xitu.io/2018/8/3/164ff0758f7e8915?w=770&h=385&f=gif&s=2132669)

Lottie 支持 `Jellybean (API 16)` 及以上版本。最简单的使用方式是直接使用 `LottieAnimationView`， `LottieAnimationView` 直接继承自 `AppCompatImageView` 。

### 2.1 Lottie 依赖 ###
`Gradle` 是唯一支持的构建配置，所以只需要在项目的 `build.gradle` 文件中添加依赖即可:
```gradle
dependencies {
  implementation "com.airbnb.android:lottie:$lottieVersion"
}
```
<span>最新版本是： </span><a href="https://search.maven.org/search?q=a:lottie"><img src="https://user-gold-cdn.xitu.io/2018/8/20/165551befe8dc066" style="vertical-align:middle;"/></a>

### 2.2 Lottie 核心类 ###
 - **LottieAnimationView**：继承自 `AppCompatImageView`，是加载 `Lottie` 动画的默认和最简单的方式。
 - **LottieDrawable**：具有大多数与 `LottieAnimationView` 相同的 `API`，因此可以在任何视图上使用它。

### 2.3 加载动画 ###
Lottie 支持 `Jellybean (API 16)` 及以上版本。Lottie 动画支持从以下位置加载动画：
 - `src/main/res/raw` 中的 json 动画。
 - `src/main/assets` 中的 json 文件。
 - `src/main/assets` 中的 zip 文件。有关详细信息，请参阅 [images docs](http://airbnb.io/lottie/android/images.html)。
 - json 或 zip 文件的 `Url`。
 - `json 字符串`。源可以来自任何东西，包括自己的网络堆栈。
 - json 文件或 zip 文件的 `InputStream`。

#### 2.3.1 在 XML 中使用 ####
最简单的使用方法是使用 `LottieAnimationView`。Lottie 支持加载来自 `res/raw` 或 `assets/` 的动画资源。建议使用 `res/raw`，因为可以对动画通过 `R 文件`使用静态引用，而不只是使用字符串名称。这也可以帮助构建静态分析，因为它可以跟踪动画的使用。

`LottieAnimationView` 的常用属性及其功能如下：

| 属性                                     | 功能                               |
|------------------------------------------|------------------------------------|
| lottie_fileName                          | 设置播放动画的 json 文件名称       |
| lottie_rawRes                            | 设置播放动画的 json 文件资源       |
| lottie_autoPlay                          | 设置动画是否自动播放(默认为false)  |
| lottie_loop                              | 设置动画是否循环(默认为false)      |
| lottie_repeatMode                        | 设置动画的重复模式(默认为restart)  |
| lottie_repeatCount                       | 设置动画的重复次数(默认为-1)       |
| lottie_cacheStrategy                     | 设置动画的缓存策略(默认为weak)     |
| lottie_colorFilter                       | 设置动画的着色颜色(优先级最低)     |
| lottie_scale                             | 设置动画的比例(默认为1f)           |
| lottie_progress                          | 设置动画的播放进度                 |
| lottie_imageAssetsFolder                 | 设置动画依赖的图片资源文件地址     |

在 `res/raw (lottie_rawRes)` 或 `assets/ (lottie_fileName)` 中存放动画的 JSON 文件，然后就可以在 xml 中直接使用，如下：
```xml
<com.airbnb.lottie.LottieAnimationView
        android:id="@+id/animation_view"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"

        app:lottie_rawRes="@raw/hello_world"
        // or
        app:lottie_fileName="hello_world.json"

        app:lottie_loop="true"
        app:lottie_autoPlay="true" />
```

#### 2.3.2 在代码中使用 ####
`LottieAnimationView` 的常用方法及其功能如下：

| 方法                                     | 功能                                    |
|------------------------------------------|-----------------------------------------|
| setAnimation(String)                     | 设置播放动画的 json 文件名称            |
| setAnimation(String, CacheStrategy)      | 设置播放动画的 json 文件资源和缓存策略  |
| setAnimation(int)                        | 设置播放动画的 json 文件名称            |
| setAnimation(int, CacheStrategy)         | 设置播放动画的 json 文件资源和缓存策略  |
| loop(boolean)                            | 设置动画是否循环(默认为false)           |
| setRepeatMode(int)                       | 设置动画的重复模式(默认为restart)       |
| setRepeatCount(int)                      | 设置动画的重复次数(默认为-1)            |
| lottie_cacheStrategy                     | 设置动画的缓存策略(默认为weak)          |
| lottie_colorFilter                       | 设置动画的着色颜色(优先级最低)          |
| setScale(float)                          | 设置动画的比例(默认为1f)                |
| setProgress(float)                       | 设置动画的播放进度                      |
| setImageAssetsFolder(String)             | 设置动画依赖的图片资源文件地址          |
| playAnimation()                          | 从头开始播放动画                        |
| pauseAnimation()                         | 暂停播放动画                            |
| resumeAnimation()                        | 继续从当前位置播放动画                  |
| cancelAnimation()                        | 取消播放动画                            |

如果不想用 xml 实现，可以通过代码来实现，可以直接加载本地动画资源，也可以从网络请求加载动画。

 - 从 `res/raw` 或 `assets/` 加载动画资源：
```java
LottieAnimationView animationView = ...

animationView.setAnimation(R.raw.hello_world);
// or
animationView.setAnimation(R.raw.hello_world.json);

animationView.playAnimation();
```
该方法在后台加载文件并解析动画，并在完成后异步开始渲染。

 - 从网络请求加载动画：
Lottie 的一个优点是可以从网络请求加载动画。所以，应该将网络请求的响应内容转换为字符串格式。Lottie
使用一个流化的 json 反序列化器来提高性能和内存使用率，所以不要将它转换成您自己的 JSONObject，这只会损害性能。
```java
LottieAnimationView animationView = ...
// This allows lottie to use the streaming deserializer mentioned above.
JsonReader jsonReader = new JsonReader(new StringReader(json.toString()));
animationView.setAnimation(jsonReader);
animationView.playAnimation();
```

### 2.4 Lottie 的缓存策略 ###
你的应用程序中可能会有一些经常使用的动画，比如加载动画等等。为了避免每次加载文件和发序列化的开销，你可以在你的动画上设置一个缓存策略。上面所有的 `setAnimation` APIs都可以采用可选的第二个参数 `CacheStrategy`。在默认情况下，Lottie 将保存对动画的弱引用，这对于大多数情况来说应该足够了。但是，如果确定某个动画肯定会经常使用，那么请将其缓存策略更改为 `CacheStrategy.Strong`；或者如果确定某个动画很大而且不会经常使用，把缓存策略改成 `CacheStrategy.None`。

`CacheStrategy` 可以是`None`、`Weak` 和 `Strong` 三种形式来让 `LottieAnimationView` 对加载和解析动画的使用强或弱引用的方式。弱或强表示缓存中组合的 GC 引用强度。

### 2.5 直接使用 `LottieDrawable` ###
LottieAnimationView 是基于 `LottieDrawable` 的一个包装好的 `ImageView` 。LottieAnimationView 上的所有 API 都在 LottieDrawable 上进行镜像，因此可以创建自己的实例并在任何可以使用drawable的地方使用它，例如自定义 View 或菜单。

### 2.6 使用 `LottieComposition` 去预加载动画 ###
动画的支持模型是 `LottieComposition`。在大多数情况下，在 `LottieAnimationView` 或 `LottieDrawable` 上调用 `setAnimation(…)` 便足够了。但是，如果要预加载动画以使其立即可用，则可以使用 `LottieComposition.Factory` API返回可以直接在 `LottieAnimationView` 或 `LottieDrawable` 设置的 `LottieComposition` 对象。同样，通常不需要自己添加管理 compositions 的开销。`LottieAnimationView` 中的默认缓存足以满足大多数用例的需要。
```java
LottieAnimationView animationView = ...;
 ...
 Cancellable compositionLoader = LottieComposition.Factory.fromJsonString(getResources(), jsonString, (composition) -> {
     animationView.setComposition(composition);
     animationView.playAnimation();
 });

 // Cancel to stop asynchronous loading of composition
 // compositionCancellable.cancel();
```

### 2.7 动画监听 ###
```java
animationView.addAnimatorUpdateListener((animation) -> {
    // do something.
});
animationView.playAnimation();
...
if (animationView.isAnimating()) {
    // do something.
}
...
animationView.setProgress(0.5f);
...
```
在更新侦听器回调中： 

 - `animation.getAnimatedValue()` 将返回动画的播放进度，而不考虑当前设置的最小/最大帧[0,1]。
 - `animation.getAnimatedFraction()` 将返回动画的播放进度，同时考虑设置的最小/最大帧[minFrame，maxFrame]。

### 2.8 控制 Lottie 动画执行的速度和时长 ###
尽管 `playAnimation()` 对于绝大多数用例来说已足够，但可以在更新回调中调用 `setProgress(...)` 方法为自己的动画设置进度。
```java
// Custom animation speed or duration.
ValueAnimator animator = ValueAnimator.ofFloat(0f, 1f);
animator.addUpdateListener(animation -> {
    animationView.setProgress(animation.getAnimatedValue());
});
animator.start();
...
```

### 2.9 循环播放 ###
可以像使用 `ValueAnimator` 一样通过 `setRepeatMode(...)` 或 `setRepeatCount(...)` 方法控制动画的循环播放，或者你直接在 `xml` 中使用 `lottie_loop="true"` 开启循环播放。

### 2.10 动画尺寸（px vs dp） ###
`Lottie` 将 `After Effects` 中的所有 `px` 值转换为设备上的 `dps`，以便在设备上以相同的大小呈现所有内容。这意味着它不是在 After Effects 中制作1920x1080的动画，而是在After Effects中更像411x731px，它大致对应于当今大多数手机的dp屏幕尺寸。

但是，如果您的动画不是完美尺寸，则有两种选择：

#### 2.10.1 ImageView scaleType ####
`LottieAnimationView` 是一个包装好的 `ImageView`，它支持 `centerCrop` 和 `centerInside`，所以可以像使用其他 image 一样使用这两个工具方法。

#### 2.10.2 Lottie setScale(...) ####
`LottieAnimationView` 和 `LottieDrawable` 都有 `setScale(float)` API，可以使用它来手动放大或缩小动画。这很少有用，但可以在某些情况下使用。

如果动画执行缓慢，请务必查看有关[性能](http://airbnb.io/lottie/android/performance.html)的文档。但是，可以尝试使用 `scaleType` 缩放动画，这将减少 Lottie 每帧渲染的数量。如果你有大的 mask 和 mattes，这将特别有用。

### 2.11 播放动画片段 ###
播放/循环动画的单个部分通常很方便。例如，动画的前半部分可能处于“开启”状态，下半部分处于“关闭”状态。LottieAnimationView 和 LottieDrawable 有用于控制当前段的 API：
```java
setMinFrame(...)

setMaxFrame(...)

setMinProgress(...)

setMaxProgress(...)

setMinAndMaxFrame(...)

setMinAndMaxProgress(...)
```
循环API将遵循此处设置的最小/最大帧。

## 3. 原理介绍 ##
### 3.1 Lottie 动画 Json 文件结构 ###
![](https://user-gold-cdn.xitu.io/2018/9/3/1659ede1389454a3?w=1065&h=600&f=png&s=334910)
Lottie动画Json结构分为4层：
1. `结构层`：可以读取到动画画布的宽高，帧数，背景色，时间，起始关键帧，结束帧等信息。
2. `asset`：图片资源信息集合，这里放置的是 制作动画时引用的图片资源。
3. `layers`：图层集合，这里可以获取到多少图层，每个图层的开始帧 结束帧等。
4. `shapes`：元素集合，可以获取到每个图层都包含多个动画元素。

### 3.2 Lottie 分层渲染原理 ###
![](https://user-gold-cdn.xitu.io/2018/9/3/1659edfff49e0778?w=1111&h=500&f=png&s=98904)
LottieComposition 负责将 json 文件解析成数据对象， LottieDrawable 负责将 LottieComposition 解析的对象
绘制成 drawable 显示在 View 上。

### 3.3 Lottie 动画运行流程图 ###
![](https://user-gold-cdn.xitu.io/2018/9/3/1659ee0fe5632dd1?w=1286&h=500&f=png&s=163927)

## 4. 使用准备 ##
### 4.1 下载安装 After Effects ###
![After Effects](https://user-gold-cdn.xitu.io/2018/8/3/164fed05f09d92d4?w=642&h=231&f=png&s=8448)
下载并安装最新版本的 Adobe After Effects。
 > 下载地址：[https://www.adobe.com/cn/products/aftereffects.html](https://www.adobe.com/cn/products/aftereffects.html)

### 4.2 下载安装 Bodymovin ###
![Bodymovin](https://user-gold-cdn.xitu.io/2018/8/3/164fed1b660d93a9?w=642&h=231&f=png&s=8448)
下载并安装最新版本的 After Effects Bodymovin Extension。
1. 如果打开，关闭 After Effects；

2. 安装 ZXP Installer 安装程序；
 > 下载地址：[http://aescripts.com/learn/zxp-installer/](http://aescripts.com/learn/zxp-installer/)

3. 下载最新的 bodymovin 扩展；
 > 下载地址：[https://github.com/airbnb/lottie-web/blob/master/build/extension](https://github.com/airbnb/lottie-web/blob/master/build/extension/bodymovin.zxp)

![bodymovin](https://user-gold-cdn.xitu.io/2018/8/3/164feda80ec2fe7b?w=2012&h=620&f=png&s=96918)

4. 打开 ZXP Installer 并将 bodymovin 扩展名拖到窗口中；

5. 打开 After Effects。

  在“Window> Extensions”菜单下，您应该看到“Bodymovin”

### 4.3 Lottiefiles 示例应用程序 ###
![Lottiefiles](https://user-gold-cdn.xitu.io/2018/8/3/164feda5074d00ef?w=642&h=231&f=png&s=8448)
可以自己构建示例应用程序，也可以从 [Google Play](https://play.google.com/store/apps/details?id=com.airbnb.lottie) 或 [App Store](https://www.lottiefiles.com/ios) 上下载 Lottie 示例应用程序。示例应用程序包含一些内置动画，但也允许您从内部存储或从 url 加载动画。 

在完成这三件事之后，请在 [SVG / Sketch 到 Lottie 演练](http://airbnb.io/lottie/after-effects/artwork-to-lottie-walkthrough.html) 或 [Illustrator 到 Lottie 演练](http://airbnb.io/lottie/after-effects/illustrator-to-lottie-walkthrough.html) 中查看一些工作流程概述。

## 5 LottieFiles ##
 LottieFiles 提供了很多设计师上传的 Lottie 动画，并提供预览的效果，并且可以直接下载成 JSON ，或者生成二维码，可供 Lottie App 扫描看效果，非常的方便。
 > LottieFiles：[https://www.lottiefiles.com/](https://www.lottiefiles.com/)
 > 
 > Lottie-editor：[https://github.com/sonaye/lottie-editor](https://github.com/sonaye/lottie-editor)

![LottieFiles](https://user-gold-cdn.xitu.io/2018/8/3/164feed2344ab30f?w=1338&h=561&f=png&s=337889)

[LottieFiles](https://www.lottiefiles.com/) 本身已经支持使用 [Lottie-editor](https://editor.lottiefiles.com/) 去编辑动画。如果需要对某个动画进行修改，只需要在动画的预览界面，点击右上角的 **Customize with Bodymovin Editor**，即可直接对该动画进行编辑。

## 6. 致谢 ##
1. [airbnb.io/lottie](https://airbnb.io/lottie/)
2. [程序员也想改 Lottie 动画？是的！](https://juejin.im/post/5acc4162f265da23826e4dc0)
