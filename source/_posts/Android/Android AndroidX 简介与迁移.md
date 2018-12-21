---
title: Android AndroidX 简介与迁移
categories: Android
tags:
  - Android
  - AndroidX
abbrlink: 3e59253e
date: 2018-11-06 18:32:35
---

## AndroidX 简介 ##
AndroidX 是 Android 团队用于在 [Jetpack](https://developer.android.com/jetpack) 中开发，测试，打包，版本和发布库的开源项目 。

AndroidX 是对原始 Android [Support Library](https://developer.android.com/topic/libraries/support-library/index)的重大改进 。与支持库一样，AndroidX 与 Android 操作系统分开提供，并提供跨 Android 版本的向后兼容性。AndroidX 通过提供功能奇偶校验和新库完全取代了支持库。此外，AndroidX 还包括以下功能：
 - AndroidX 中的所有软件包都以字符串开头，位于一致的命名空间中 `androidx`。支持库包已映射到相应的 `androidx.*` 包中。有关所有旧类和构建组件的完整映射到新构件。
 - 与支持库不同，AndroidX 软件包是单独维护和更新的。这些 `androidx` 包使用从版本 1.0.0 开始的严格语义版本控制。开发者可以单独更新项目中的 AndroidX 库。
 - 所有新的支持库开发都将在 AndroidX 库中进行。这包括维护原始支持库组件和引入新的 Jetpack 组件。
 
## 使用 AndroidX ##
如果要在新项目中使用 AndroidX，则需要一下条件：
 - 使用 `Android Studio 3.2 及更高版本`
 - 将 `compileSdkVersion` 设置为 `28(Android 9.0)或更高版本`
 - 将 `gradle-wrapper.properties` 文件中的 Gradle 版本改为 `4.6或更高版本`
 - 在 `gradle.properties` 文件中将以下两个 Android Gradle 插件标志设置 `true`：
    - `android.useAndroidX：`设置 true 为时，Android 插件使用相应的 AndroidX 库而不是支持库，默认为 false。
    - `android.enableJetifier：`设置 true 为时，Android 插件会自动迁移现有的第三方库，通过重写其二进制文件来使用 AndroidX，默认为 false。

## 迁移到 AndroidX ##
AndroidX 将原始支持库 API 包映射到 `androidx` 命名空间。只有包和 Maven 组件名称发生了变化; 类、方法和字段名称没有改变。

使用 Android Studio 3.2 及更高版本，您可以通过从菜单栏中选择 **Refactor> Migrate to AndroidX** 快速迁移现有项目以使用 AndroidX 。

如果您有任何尚未迁移到 AndroidX 名称空间的 Maven 依赖项，那么当您将 `gradle.properties` 文件中以下两个 Android Gradle 插件标志设置 `true`，Android Studio 构建系统也会为您迁移这些依赖项：
```gradle
android.useAndroidX=true
android.enableJetifier=true
```
要迁移不使用任何需要转换的依赖项的第三方库的现有项目，可以将 `android.useAndroidX` 标志设置为 `true`，将 `android.enableJetifier` 标志设置为 `false`。

## 组件映射 ##
下表列出了从旧组件到新组件的当前映射，也可以下载[CSV格式](https://developer.android.google.cn/topic/libraries/support-library/downloads/androidx-artifact-mapping.csv)映射文件：

| 旧构建组件                                                 | AndroidX 构建组件                                          | 
| ---------------------------------------------------------- | ---------------------------------------------------------- | 
| android.arch.core:common                                   | androidx.arch.core:core-common:2.0.0-rc01                  |
| android.arch.core:core                                     | androidx.arch.core:core:2.0.0-rc01                         |
| android.arch.core:core-testing                             | androidx.arch.core:core-testing:2.0.0-rc01                 |
| android.arch.core:runtime                                  | androidx.arch.core:core-runtime:2.0.0-rc01                 |
| android.arch.lifecycle:common                              | androidx.lifecycle:lifecycle-common:2.0.0-rc01             |
| android.arch.lifecycle:common-java8                        | androidx.lifecycle:lifecycle-common-java8:2.0.0-rc01       |
| android.arch.lifecycle:compiler                            | androidx.lifecycle:lifecycle-compiler:2.0.0-rc01           |
| android.arch.lifecycle:extensions                          | androidx.lifecycle:lifecycle-extensions:2.0.0-rc01         |
| android.arch.lifecycle:livedata                            | androidx.lifecycle:lifecycle-livedata:2.0.0-rc01           |
| android.arch.lifecycle:livedata-core                       | androidx.lifecycle:lifecycle-livedata-core:2.0.0-rc01      |
| android.arch.lifecycle:reactivestreams                     | androidx.lifecycle:lifecycle-reactivestreams:2.0.0-rc01    |
| android.arch.lifecycle:runtime                             | androidx.lifecycle:lifecycle-runtime:2.0.0-rc01            |
| android.arch.lifecycle:viewmodel                           | androidx.lifecycle:lifecycle-viewmodel:2.0.0-rc01          |
| android.arch.paging:common                                 | androidx.paging:paging-common:2.0.0-rc01                   |
| android.arch.paging:runtime                                | androidx.paging:paging-runtime:2.0.0-rc01                  |
| android.arch.paging:rxjava2                                | androidx.paging:paging-rxjava2:2.0.0-rc01                  |
| android.arch.persistence.room:common                       | androidx.room:room-common:2.0.0-rc01                       |
| android.arch.persistence.room:compiler                     | androidx.room:room-compiler:2.0.0-rc01                     |
| android.arch.persistence.room:guava                        | androidx.room:room-guava:2.0.0-rc01                        |
| android.arch.persistence.room:migration                    | androidx.room:room-migration:2.0.0-rc01                    |
| android.arch.persistence.room:runtime                      | androidx.room:room-runtime:2.0.0-rc01                      |
| android.arch.persistence.room:rxjava2                      | androidx.room:room-rxjava2:2.0.0-rc01                      |
| android.arch.persistence.room:testing                      | androidx.room:room-testing:2.0.0-rc01                      |
| android.arch.persistence:db                                | androidx.sqlite:sqlite:2.0.0-rc01                          |
| android.arch.persistence:db-framework                      | androidx.sqlite:sqlite-framework:2.0.0-rc01                |
| com.android.support.constraint:constraint-layout           | androidx.constraintlayout:constraintlayout:1.1.2           |
| com.android.support.constraint:constraint-layout-solver    | androidx.constraintlayout:constraintlayout-solver:1.1.2    |
| com.android.support.test.espresso.idling:idling-concurrent | androidx.test.espresso.idling:idling-concurrent:3.1.0      |
| com.android.support.test.espresso.idling:idling-net	     | androidx.test.espresso.idling:idling-net:3.1.0             |
| com.android.support.test.espresso:espresso-accessibility   | androidx.test.espresso:espresso-accessibility:3.1.0        |
| com.android.support.test.espresso:espresso-contrib	     | androidx.test.espresso:espresso-contrib:3.1.0              |
| com.android.support.test.espresso:espresso-core            | androidx.test.espresso:espresso-core:3.1.0                 |
| com.android.support.test.espresso:espresso-idling-resource | androidx.test.espresso:espresso-idling-resource:3.1.0      |
| com.android.support.test.espresso:espresso-intents	     | androidx.test.espresso:espresso-intents:3.1.0              |
| com.android.support.test.espresso:espresso-remote          | androidx.test.espresso:espresso-remote:3.1.0               |
| com.android.support.test.espresso:espresso-web             | androidx.test.espresso:espresso-web:3.1.0                  |
| com.android.support.test.janktesthelper:janktesthelper     | androidx.test.jank:janktesthelper:1.0.1                    |
| com.android.support.test.services:test-services            | androidx.test:test-services:1.1.0                          |
| com.android.support.test.uiautomator:uiautomator           | androidx.test.uiautomator:uiautomator:2.2.0                |
| com.android.support.test:monitor                           | androidx.test:monitor:1.1.0                                |
| com.android.support.test:orchestrator	                     | androidx.test:orchestrator:1.1.0                           |
| com.android.support.test:rules                             | androidx.test:rules:1.1.0                                  |
| com.android.support.test:runner                            | androidx.test:runner:1.1.0                                 |
| com.android.support:animated-vector-drawable	             | androidx.vectordrawable:vectordrawable-animated:1.0.0      |
| com.android.support:appcompat-v7	                     | androidx.appcompat:appcompat:1.0.0                         |
| com.android.support:asynclayoutinflater	             | androidx.asynclayoutinflater:asynclayoutinflater:1.0.0     |
| com.android.support:car	                             | androidx.car:car:1.0.0                                     |
| com.android.support:cardview-v7	                     | androidx.cardview:cardview:1.0.0                           |
| com.android.support:collections	                     | androidx.collection:collection:1.0.0                       |
| com.android.support:coordinatorlayout	                     | androidx.coordinatorlayout:coordinatorlayout:1.0.0         |
| com.android.support:cursoradapter	                     | androidx.cursoradapter:cursoradapter:1.0.0                 |
| com.android.support:customtabs	                     | androidx.browser:browser:1.0.0                             |
| com.android.support:customview	                     | androidx.customview:customview:1.0.0                       |
| com.android.support:design	                             | com.google.android.material:material:1.0.0-rc01            |
| com.android.support:documentfile	                     | androidx.documentfile:documentfile:1.0.0                   |
| com.android.support:drawerlayout	                     | androidx.drawerlayout:drawerlayout:1.0.0                   |
| com.android.support:exifinterface	                     | androidx.exifinterface:exifinterface:1.0.0                 |
| com.android.support:gridlayout-v7	                     | androidx.gridlayout:gridlayout:1.0.0                       |
| com.android.support:heifwriter	                     | androidx.heifwriter:heifwriter:1.0.0                       |
| com.android.support:interpolator	                     | androidx.interpolator:interpolator:1.0.0                   |
| com.android.support:leanback-v17	                     | androidx.leanback:leanback:1.0.0                           |
| com.android.support:loader	                             | androidx.loader:loader:1.0.0                               |
| com.android.support:localbroadcastmanager	             | androidx.localbroadcastmanager:localbroadcastmanager:1.0.0 |
| com.android.support:media2	                             | androidx.media2:media2:1.0.0-alpha03                       |
| com.android.support:media2-exoplayer	                     | androidx.media2:media2-exoplayer:1.0.0-alpha01             |
| com.android.support:mediarouter-v7	                     | androidx.mediarouter:mediarouter:1.0.0                     |
| com.android.support:multidex	                             | androidx.multidex:multidex:2.0.0                           |
| com.android.support:multidex-instrumentation	             | androidx.multidex:multidex-instrumentation:2.0.0           |
| com.android.support:palette-v7	                     | androidx.palette:palette:1.0.0                             |
| com.android.support:percent	                             | androidx.percentlayout:percentlayout:1.0.0                 |
| com.android.support:preference-leanback-v17                | androidx.leanback:leanback-preference:1.0.0                |
| com.android.support:preference-v14	                     | androidx.legacy:legacy-preference-v14:1.0.0                |
| com.android.support:preference-v7	                     | androidx.preference:preference:1.0.0                       |
| com.android.support:print	                             | androidx.print:print:1.0.0                                 |
| com.android.support:recommendation	                     | androidx.recommendation:recommendation:1.0.0               |
| com.android.support:recyclerview-selection                 | androidx.recyclerview:recyclerview-selection:1.0.0         |
| com.android.support:recyclerview-v7                        | androidx.recyclerview:recyclerview:1.0.0                   |
| com.android.support:slices-builders	                     | androidx.slice:slice-builders:1.0.0                        |
| com.android.support:slices-core	                     | androidx.slice:slice-core:1.0.0                            |
| com.android.support:slices-view	                     | androidx.slice:slice-view:1.0.0                            |
| com.android.support:slidingpanelayout	                     | androidx.slidingpanelayout:slidingpanelayout:1.0.0         |
| com.android.support:support-annotations	             | androidx.annotation:annotation:1.0.0                       |
| com.android.support:support-compat	                     | androidx.core:core:1.0.0                                   |
| com.android.support:support-content	                     | androidx.contentpager:contentpager:1.0.0                   |
| com.android.support:support-core-ui	                     | androidx.legacy:legacy-support-core-ui:1.0.0               |
| com.android.support:support-core-utils	             | androidx.legacy:legacy-support-core-utils:1.0.0            |
| com.android.support:support-dynamic-animation              | androidx.dynamicanimation:dynamicanimation:1.0.0           |
| com.android.support:support-emoji	                     | androidx.emoji:emoji:1.0.0                                 |
| com.android.support:support-emoji-appcompat	             | androidx.emoji:emoji-appcompat:1.0.0                       |
| com.android.support:support-emoji-bundled	             | androidx.emoji:emoji-bundled:1.0.0                         |
| com.android.support:support-fragment	                     | androidx.fragment:fragment:1.0.0                           |
| com.android.support:support-media-compat	             | androidx.media:media:1.0.0                                 |
| com.android.support:support-tv-provider	             | androidx.tvprovider:tvprovider:1.0.0                       |
| com.android.support:support-v13	                     | androidx.legacy:legacy-support-v13:1.0.0                   |
| com.android.support:support-v4	                     | androidx.legacy:legacy-support-v4:1.0.0                    |
| com.android.support:support-vector-drawable                | androidx.vectordrawable:vectordrawable:1.0.0               |
| com.android.support:swiperefreshlayout	             | androidx.swiperefreshlayout:swiperefreshlayout:1.0.0       |
| com.android.support:textclassifier                         | androidx.textclassifier:textclassifier:1.0.0               |
| com.android.support:transition	                     | androidx.transition:transition:1.0.0                       |
| com.android.support:versionedparcelable	             | androidx.versionedparcelable:versionedparcelable:1.0.0     |
| com.android.support:viewpager	                             | androidx.viewpager:viewpager:1.0.0                         |
| com.android.support:wear	                             | androidx.wear:wear:1.0.0                                   |
| com.android.support:webkit	                             | androidx.webkit:webkit:1.0.0                               |

## 类映射 ##
下表列出了部分常用的从旧命名空间到新 `androidx` 包的当前映射，也可以下载[CSV格式](https://developer.android.google.cn/topic/libraries/support-library/downloads/androidx-class-mapping.csv)映射文件：

| 支持库类	                                                    | AndroidX 类                                                  |
| ----------------------------------------------------------------- | ------------------------------------------------------------ | 
| android.support.multidex.MultiDex	                            | androidx.multidex.MultiDex                                   |
| android.support.v4.app.ActivityCompat                      	    | androidx.core.app.ActivityCompat                             |
| android.support.v4.app.ActivityManagerCompat	                    | androidx.core.app.ActivityManagerCompat                      |
| android.support.v4.app.ActivityOptionsCompat                      | androidx.core.app.ActivityOptionsCompat                      |
| android.support.v4.app.AlarmManagerCompat	                    | androidx.core.app.AlarmManagerCompat                         |
| android.support.v4.app.BundleCompat	                            | androidx.core.app.BundleCompat                               |
| android.support.v4.app.DialogFragment	                            | androidx.fragment.app.DialogFragment                         |
| android.support.v4.app.Fragment	                            | androidx.fragment.app.Fragment                               |
| android.support.v4.app.FragmentActivity	                    | androidx.fragment.app.FragmentActivity                       |
| android.support.v4.app.FragmentManager	                    | androidx.fragment.app.FragmentManager                        |
| android.support.v4.app.ListFragment	                            | androidx.fragment.app.ListFragment                           |
| android.support.v4.app.LoaderManager	                            | androidx.loader.app.LoaderManager                            |
| android.support.v4.app.NotificationCompat	                    | androidx.core.app.NotificationCompat                         |
| android.support.v4.app.NotificationCompatBuilder	            | androidx.core.app.NotificationCompatBuilder                  |
| android.support.v4.app.NotificationManagerCompat	            | androidx.core.app.NotificationManagerCompat                  |
| android.support.v4.content.ContextCompat	                    | androidx.core.content.ContextCompat                          |
| android.support.v4.content.CursorLoader	                    | androidx.loader.content.CursorLoader                         |
| android.support.v4.content.FileProvider	                    | androidx.core.content.FileProvider                           |
| android.support.v4.graphics.BitmapCompat	                    | androidx.core.graphics.BitmapCompat                          |
| android.support.v4.graphics.ColorUtils	                    | androidx.core.graphics.ColorUtils                            |
| android.support.v4.graphics.PaintCompat                           | androidx.core.graphics.PaintCompat                           |
| android.support.v4.graphics.PathParser	                    | androidx.core.graphics.PathParser                            |
| android.support.v4.math.MathUtils	                            | androidx.core.math.MathUtils                                 |
| android.support.v4.util.LruCache	                            | androidx.collection.LruCache                                 |
| android.support.v4.util.Pair	                                    | androidx.core.util.Pair                                      |
| android.support.v4.util.TimeUtils	                            | androidx.core.util.TimeUtils                                 |
| android.support.v4.widget.AutoScrollHelper	                    | androidx.core.widget.AutoScrollHelper                        |
| android.support.v4.widget.AutoSizeableTextView	            | androidx.core.widget.AutoSizeableTextView                    |
| android.support.v4.widget.CircleImageView	                    | androidx.swiperefreshlayout.widget.CircleImageView           |
| android.support.v4.widget.DrawerLayout	                    | androidx.drawerlayout.widget.DrawerLayout                    |
| android.support.v7.app.AlertDialog	                            | androidx.appcompat.app.AlertDialog                           |
| android.support.v7.app.AppCompatActivity	                    | androidx.appcompat.app.AppCompatActivity                     |
| android.support.v7.app.AppCompatDialog	                    | androidx.appcompat.app.AppCompatDialog                       |
| android.support.v7.widget.CardView	                            | androidx.cardview.widget.CardView                            |
| android.support.v7.widget.GridLayout	                            | androidx.gridlayout.widget.GridLayout                        |
| android.support.v7.widget.GridLayoutManager	                    | androidx.recyclerview.widget.GridLayoutManager               |

## AndroidX 影响 ##
官方博客中有说道，为了给开发者一定迁移的时间，所以 `28.0.0` 的稳定版本还是采用 `android.support`，但是所有后续的功能版本都将采用 `androidx`。

其实目前对于我们影响也不是很大，我们可以选择不使用，毕竟不是强制的。但长远看来还是有好处的。AndroidX 重新设计了包结构，旨在鼓励库的小型化，支持库和架构组件包的名字也都简化了；而且也是减轻 Android 生态系统碎片化的有效方式。

