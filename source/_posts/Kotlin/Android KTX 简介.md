---
title: Android KTX 简介
date: 2018-11-11 13:45:38
categories: Kotlin
tags:
  - Android
  - Kotlin
  - KTX
---

## KTX 简介 ##
Android KTX 是一组 Kotlin 扩展程序，属于 Android Jetpack 系列。它优化了供 Kotlin 使用的 Jetpack 和 Android 平台 API。Android KTX 旨在让您利用 Kotlin 语言功能（例如扩展函数/属性、lambda、命名参数和参数默认值），以更简洁、更愉悦、更惯用的方式使用 Kotlin 进行 Android 开发。Android KTX 不会向现有的 Android API 添加任何新功能。

## KTX 使用 ##
要开始使用 Android KTX，请将以下代码添加到项目的 `build.gradle` 文件中：
```gradle
repositories {
    google()
}
```

Android KTX 划分为不同的模块。每个模块都包含一个或多个软件包。

使用模块时，请在应用的 `build.gradle` 文件中为每个 Android KTX 软件工件添加一个依赖项。请记住在软件工件后面附上版本号。例如，如果您使用 `core-ktx` 模块，则完整的依赖项将如下所示：
```gradle
dependencies {
    implementation 'androidx.core:core-ktx:1.0.0-alpha1'
}
```

## KTX 模块 ##
Android KTX 由以下 Maven 软件工件组成。要获取 API 参考文档，请点击特定软件包名称并查看扩展函数摘要。

| KTX 模块                                         | 版本          | 软件包                                                                                                                                                  |
| ------------------------------------------------ | ------------  | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| androidx.core:core-ktx                           | 1.0.0-alpha1  | 查看下面的[所有核心软件包](#core-ktx)                                                                                                              |
| androidx.fragment:fragment-ktx                   | 1.0.0-alpha1  | [androidx.fragment.app](https://developer.android.com/reference/kotlin/androidx/fragment/app/package-summary#extension-functions-summary)               |
| androidx.palette:palette-ktx                     | 1.0.0-alpha1  | [androidx.palette.graphics](https://developer.android.com/reference/kotlin/androidx/palette/graphics/package-summary#extension-functions-summary)       |
| androidx.sqlite:sqlite-ktx                       | 1.0.0-alpha1  | [androidx.sqlite.db](https://developer.android.com/reference/kotlin/androidx/sqlite/db/package-summary#extension-functions-summary)                     |
| androidx.collection:collection-ktx               | 1.0.0-alpha1  | [androidx.collection](https://developer.android.com/reference/kotlin/androidx/collection/package-summary#extension-functions-summary)                   |
| androidx.lifecycle:lifecycle-viewmodel-ktx       | 2.0.0-alpha1  | [androidx.lifecycle](https://developer.android.com/reference/kotlin/androidx/lifecycle/package-summary#extension-functions-summary)                     |
| androidx.lifecycle:lifecycle-reactivestreams-ktx | 2.0.0-alpha1  | [androidx.lifecycle](https://developer.android.com/reference/kotlin/androidx/lifecycle/package-summary#extension-functions-summary)                     |
| android.arch.navigation:navigation-common-ktx    | 1.0.0-alpha01 | [androidx.navigation](https://developer.android.com/reference/kotlin/androidx/navigation/package-summary#extension-functions-summary)                   |
| android.arch.navigation:navigation-fragment-ktx  | 1.0.0-alpha01 | [androidx.navigation.fragment](https://developer.android.com/reference/kotlin/androidx/navigation/fragment/package-summary#extension-functions-summary) |
| android.arch.navigation:navigation-runtime-ktx   | 1.0.0-alpha01 | [androidx.navigation](https://developer.android.com/reference/kotlin/androidx/navigation/package-summary#extension-functions-summary)                   |
| android.arch.navigation:navigation-testing-ktx   | 1.0.0-alpha01 | [androidx.navigation.testing](https://developer.android.com/reference/kotlin/androidx/navigation/testing/package-summary#extension-functions-summary)   |
| android.arch.navigation:navigation-ui-ktx        | 1.0.0-alpha01 | [androidx.navigation.ui](https://developer.android.com/reference/kotlin/androidx/navigation/ui/package-summary#extension-functions-summary)             |
| android.arch.work:work-runtime-ktx               | 1.0.0-alpha01 | [androidx.work.ktx](https://developer.android.com/reference/kotlin/androidx/work/ktx/package-summary#extension-functions-summary)                       |

### core-ktx ###
核心模块包括以下软件包：
 - [androidx.core.animation](https://developer.android.com/reference/kotlin/androidx/core/animation/package-summary#extension-functions-summary)
 - [androidx.core.content](https://developer.android.com/reference/kotlin/androidx/core/content/package-summary#extension-functions-summary)
 - [androidx.core.graphics](https://developer.android.com/reference/kotlin/androidx/core/graphics/package-summary#extension-functions-summary)
 - [androidx.core.graphics.drawable](https://developer.android.com/reference/kotlin/androidx/core/graphics/drawable/package-summary#extension-functions-summary)
 - [androidx.core.net](https://developer.android.com/reference/kotlin/androidx/core/net/package-summary#extension-functions-summary)
 - [androidx.core.os](https://developer.android.com/reference/kotlin/androidx/core/os/package-summary#extension-functions-summary)
 - [androidx.core.preference](https://developer.android.com/reference/kotlin/androidx/core/preference/package-summary#extension-functions-summary)
 - [androidx.core.text](https://developer.android.com/reference/kotlin/androidx/core/text/package-summary#extension-functions-summary)
 - [androidx.core.transition](https://developer.android.com/reference/kotlin/androidx/core/transition/package-summary#extension-functions-summary)
 - [androidx.core.util](https://developer.android.com/reference/kotlin/androidx/core/util/package-summary#extension-functions-summary)
 - [androidx.core.view](https://developer.android.com/reference/kotlin/androidx/core/view/package-summary#extension-functions-summary)
 - [androidx.core.widget](https://developer.android.com/reference/kotlin/androidx/core/widget/package-summary#extension-functions-summary)

## KTX 示例 ##
Android KTX 是 Android Jetpack 基础组件。您可以在 [Sunflower](https://github.com/googlesamples/android-sunflower) 演示应用中查看它的使用情况。

以下示例演示了一些 Android KTX 扩展函数。它们按模块（软件工件）名称分组。有关扩展函数的完整列表，请查看完整的软件包参考文档。

### androidx.core:core-ktx ###
**Kotlin:**
```kotlin
sharedPreferences.edit()
    .putBoolean("key", value)
    .apply()
```

**Kotlin + Android KTX:**
```kotlin
sharedPreferences.edit {
    putBoolean("key", value)
}
```

### androidx.sqlite:sqlite-ktx ###
**Kotlin:**
```kotlin
db.beginTransaction()
try {
    // insert data
    db.setTransactionSuccessful()
} finally {
    db.endTransaction()
}
```

**Kotlin + Android KTX:**
```kotlin
db.transaction {
    // insert data
}
```

### androidx.fragment:fragment-ktx ###
**Kotlin:**
```kotlin
supportFragmentManager
    .beginTransaction()
    .replace(R.id.my_fragment_container, myFragment, FRAGMENT_TAG)
    .commitAllowingStateLoss()
```

**Kotlin + Android KTX:**
```kotlin
supportFragmentManager.transaction(allowStateLoss = true) {
    replace(R.id.my_fragment_container, myFragment, FRAGMENT_TAG)
}
```

## KTX 资源 ##
Android KTX 的相关网站：
 - [Android KTX](https://developer.android.com/kotlin/ktx)
 - [Android KTX on GitHub](https://github.com/android/android-ktx)
 - [Android KTX API Doc](https://android.github.io/android-ktx/core-ktx/)
