---
title: Android Jetpack 简介
date: 2018-11-04 10:30:35
categories: Android
tags:
  - Android
  - Jetpack
---

Android Jetpack 深受支持库的启发，支持库包含的组件可以让开发者轻松利用 Android 新功能，同时保持向后兼容性；现在，应用商店中 99% 的应用都使用支持库。在支持库取得成功后，Google 推出了[架构组件](https://developer.android.com/jetpack/arch/)，让开发者在面对应用生命周期变化和复杂性时可以更轻松地处理数据。自从 Google 在去年的 I/O 大会上推出以来，相当数量的开发者已经采用这些组件。LinkedIn、Zillow 和 iHeartRadio 等公司取得了显著成效，他们应用的错误减少、可测试性提高，这让他们可以将更多时间放在精心打造自己的应用上。

## Jetpack 简介 ##
2018年的 Google I/O 大会上，谷歌推出了 Android Jetpack 架构组件。Android Jetpack 是一套组件、工具和指导，可以帮助您构建出色的 Android 应用。Android Jetpack 组件将现有的支持库与架构组件联系起来，并将它们分成四个类别：Architecture、Foundation、Behavior 以及 UI。
![Android Jetpack 分类](http://localhost:4000/medias/android/android_jetpack.png)

Android Jetpack 组件以“未捆绑的”库形式提供，这些库不是基础 Android 平台的一部分。这就意味着，您可以根据自己的需求采用每一个组件。在新的 Android Jetpack 功能发布后，您可以将其添加到自己的应用中，将您的应用部署到应用商店并向用户提供新功能，如果您的行动足够快，所有这些可以在一天内完成！
未捆绑的 Android Jetpack 库已经全部转移到新的 [`androidx.*`](https://developer.android.com/jetpack/androidx/) 命名空间中。这意味着它提供向后兼容性并且比 Android 平台更频繁地更新，确保开发者始终可以访问最新和最好的 Jetpack 组件版本。

Android Jetpack 是下一代的 Android 组件，该组件规模更庞大，并且具备支持库在向后兼容性和即时更新方面的优点，从而让开发者能够更轻松快捷地构建可靠的高品质应用。Android Jetpack 管理后台任务、导航和生命周期管理等活动，因此，您不必使用千篇一律的模板代码，而可以专注于如何让您的应用精彩绝伦。Android Jetpack 旨在与 Kotlin 搭配运行，在使用 Android KTX 时可以节省更多代码。

## Jetpack 的新组件 ##
Google 发布的全新 Android Jetpack 附带五个新组件：
 - [WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager/)
 - [Navigation](https://developer.android.com/topic/libraries/architecture/navigation/)
 - [Paging](https://developer.android.com/topic/libraries/architecture/paging/)
 - [Slices](https://developer.android.com/guide/slices/)
 - [Android KTX](https://developer.android.com/kotlin/ktx)

### WorkManager ###
`WorkMananager` 组件是一个功能强大的新库，可以为基于约束的后台作业（需要有保障的执行）提供一站式解决方案，消除了使用作业或 SyncAdapter 等框架的需求。WorkManager 提供了一个简化的现代化 API、在安装或未安装 Google Play 服务的设备上运行的功能、创建工作图的功能以及查询工作状态的功能。

WorkManager 根据设备 API 级别和应用程序状态等因素选择适当的方式来运行任务。如果 WorkManager 在应用程序运行时执行您的任务之一，WorkManager 可以在您应用程序进程的新线程中运行您的任务。如果您的应用程序未运行，WorkManager 会选择一种合适的方式来安排后台任务 - 具体取决于设备API级别和包含的依赖项，WorkManager 可能会使用 JobScheduler、Firebase JobDispatcher 或 AlarmManager。开发者无需编写设备逻辑来确定设备具有哪些功能并选择适当的 API; 相反，开发者可以将任务交给 WorkManager，让它选择最佳选项。

### Navigation ###
尽管 Activity 是系统提供的您的应用界面的入口点，但在相互分享数据以及转场方面，Activity 表现得不够灵活，这就让它不适合作为构建您的应用内导航的理想架构。于是，Google 宣布推出`导航组件`，作为构建 Android 应用内界面的框架，重点是让单 Activity 应用成为首选架构。利用导航组件对 Fragment 的原生支持，开发者可以获得架构组件的所有好处（例如生命周期和 ViewModel），同时让此组件为开发者处理 FragmentTransaction 的复杂性。此外，导航组件还可以自动构建正确的“向上”和“返回”行为，包含对深层链接的完整支持，并提供了帮助程序，用于将导航关联到合适的 UI 小部件，例如抽屉式导航栏和底部导航。但这些并不是全部！`Android Studio 3.2` 中的`导航编辑器`让开发者可以直观地查看和管理导航属性：
![导航编辑器](http://localhost:4000/medias/android/android_navigation.png)

### Paging ###
应用中呈现的数据可能非常大，这就导致加载的开销比较大，因此，避免一次下载、创建或呈现过多数据就显得非常重要。`分页组件`让开发者可以轻松加载和呈现大型数据集，同时在 `RecyclerView` 中进行快速、无限滚动。它可以从本地存储和/或网络加载分页数据，并让开发者能够定义内容的加载方式。此组件原生支持 `Room`、`LiveData` 和 `RxJava`。

### Slices ###
`切片`提供能在 App 之外展示(Google Search App 和 Googel Assistant）App 数据的 UI 元素，：
![Slices](http://localhost:4000/medias/android/android_slices.png)

Android Jetpack 内置了对切片的支持，可以一直延伸到 Android 4.4，大约 95% 的Android用户。

### Android KTX ###
Android KTX 是一组 Kotlin 扩展程序，属于 Android Jetpack 系列。它优化了供 Kotlin 使用的 Jetpack 和 Android 平台 API。Android KTX 旨在让开发者利用 Kotlin 语言功能（例如扩展函数/属性、lambda、命名参数和参数默认值），以更简洁、更愉悦、更惯用的方式使用 Kotlin 进行 Android 开发。Android KTX 不会向现有的 Android API 添加任何新功能。

Android Jetpack 利用 Kotlin 语言功能的一个目标是提高您的效率。Android KTX 可以让您将类似下面所示的 Kotlin 代码：
```kotlin
view.viewTreeObserver.addOnPreDrawListener(
    object : ViewTreeObserver.OnPreDrawListener {
        override fun onPreDraw(): Boolean {
            viewTreeObserver.removeOnPreDrawListener(this)
            actionToBeTriggered()
            return true
        }
    }
)
```
转换成如下所示的更精简的 Kotlin 代码：
```kotlin
view.doOnPreDraw {
     actionToBeTriggered()
}
```
