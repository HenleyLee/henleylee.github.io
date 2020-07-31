---
title: Android build.gradle 配置详解
categories: Android
tags:
  - Android
  - Gradle
abbrlink: 9986453e
date: 2018-12-20 18:25:36
---

## Gradle 简介 ##
Android Studio 是采用 `Gradle` 来构建项目的。`Gradle` 是一个非常先进的项目构建工具，若想用 Gradle 构建 Android 项目，需要创建一个脚本，此脚本被称为 `build.gradle`。

Gradle 构建脚本并非基于传统的 `XML` 文件(如 Ant 和 Maven)，而是 `Groovy` 的领域专业语言(`DSL`)。`Groovy` 是一种基于 `JVM` 的动态语言，优势更加显著。

> 若只是用它构建普通的工程，可以不去学 Groovy 语言；若想深入的研究自定义的构建插件，可以考虑学 Groovy，因为 Groovy 语言是基于 JVM 的动态语言，所以有 Java 基础的同学学习 Groovy 语言不会很难。

## build.gradle 文件 ##
在一个 `Android` 项目中一般会出现至少 2 个 `build.gradle` 文件，一个是 `Project` 的 `gradle` 文件，其他的都是 `Module` 的 `gradle` 文件。

> 如果项目目录结构切换到 Android 模式下，则所有的 `gradle` 文件都在 `Gradle Scripts` 分组下。

### Project 的 build.gradle 文件 ###
**`Project`** 的 `build.gradle` 文件对应的默认配置如下：
```gradle
// Top-level build file where you can add configuration options common to all sub-projects/modules.

// 配置 Gradle 脚本执行所需依赖分别是对应的 Maven 库和插件
buildscript {
    // 配置 Gradle 脚本依赖项上所需要的存储库
    repositories {
        google()    // 从 Android Studio 3.0 后新增了 google() 配置，可以引用 google 上的开源项目
        jcenter()   // 是一个代码托管仓库，声明了 jcenter() 配置，可以引用 jcenter 上的开源项目
    }
    // 配置 Gradle 脚本所需要的依赖项
    dependencies {
        // Android 项目的 Gradle 插件，Gradle 是一个强大的项目构建工具
        classpath 'com.android.tools.build:gradle:3.2.1'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

// 配置项目本身及其每个子项目所需要的依赖
allprojects {
    // 配置此项目所需依赖的存储库
    repositories {
        google()
        jcenter()
    }
}

// 运行 gradle clean 时，执行此处定义的 task 任务。该任务继承自 Delete，删除根目录中的 build 目录。
// 相当于执行Delete.delete(rootProject.buildDir)。
task clean(type: Delete) {
    delete rootProject.buildDir
}
```

可以看到 `Project` 的 `build.gradle` 配置包含以下内容：
 - **`buildscript{}：`**配置 Gradle 脚本执行所需依赖分别是对应的 Maven 库和插件。
   - **`repositories{}：`**配置 Gradle 脚本依赖项上所需要的存储库。
   - **`dependencies{}：`**配置 Gradle 脚本所需要的依赖项。
 - **`allprojects{}：`**配置项目本身及其每个子项目所需要的依赖。
 - **`task clean(type: Delete){}：`**运行 `gradle clean` 时，执行此处定义的 task 任务。该任务继承自 Delete，删除根目录中的 build 目录。

### Module 的 build.gradle 文件 ###
**`Module`** 的 `build.gradle` 文件对应的默认配置如下：
```gradle
apply plugin: 'com.android.application'

android {
    compileSdkVersion 28
    buildToolsVersion "28.0.0"
    defaultConfig {
        applicationId "com.android.application"
        minSdkVersion 14
        targetSdkVersion 28
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:appcompat-v7:28.0.0'
    implementation 'com.android.support:design:28.0.0'
    implementation 'com.android.support.constraint:constraint-layout:1.1.3'
    implementation 'com.android.support:support-vector-drawable:28.0.0'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.2'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
}
```

可以看到 `Module` 的 `build.gradle` 配置包含以下内容：
 - **`apply plugin：`**配置 Module 应用的插件。
 - **`android：`**配置项目构建的各种属性。
   - **`compileSdkVersion：`**配置编译版本。
   - **`buildToolsVersion：`**配置构建工具的版本。
   - **`defaultConfig{}：`**配置 Android 插件应用于所有构建变体的变体属性。
   - **`buildTypes{}：`**配置项目的所有构建类型。
 - **`dependencies{}：`**配置项目的依赖项。

#### apply plugin 文件 ####
**`apply plugin：`**配置 Module 应用的插件。一般位于文件中第一行，该插件一般有两种值可选：
 - **`com.android.application：`**表示该模块为应用程序模块，可以直接运行，打包得到的是 `.apk` 文件。
 - **`com.android.library：`**表示该模块为库模块，只能作为代码库依附于别的应用程序模块来运行，打包得到的是 `.aar` 文件。

配置代码如下：
```gradle
apply plugin: 'com.android.application'
// apply plugin: 'com.android.library'
```

> `apply plugin: 'com.android.application'` 等同于 `project.plugins.apply("com.android.application")`。

`apply plugin` 除了可以指定上面两种插件之外，还可以指定其他的插件，代码如下：
```gradle
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
apply plugin: 'kotlin-kapt'

apply plugin: 'com.jakewharton.butterknife'
```

#### android{} 闭包 ####
**`android：`**闭包主要用于配置项目构建的各种属性，代码中由 `com.android.build.gradle.AppExtension` 类表示，该类继承自 `com.android.build.gradle.BaseExtension`。`BaseExtension` 是所有 Android 插件的基本扩展，使用过程中不直接使用此扩展，而是使用以下之一：
> `AppExtension` 用于构建 `Android app module` 的 `com.android.application` 插件扩展。
> `LibraryExtension` 用于构建 `Android library module` 的 `com.android.library` 插件扩展。
> `TestExtension` 用于构建 `Android test module` 的 `com.android.test` 插件扩展。
> `FeatureExtension` 用于构建 `Android Instant Apps` 的 `com.android.feature` 插件扩展。

##### compileSdkVersion #####
**`compileSdkVersion：`**用于配置编译时的 Android SDK 版本。

##### buildToolsVersion #####
**`buildToolsVersion：`**用于配置编译时使用的构建工具的版本。Android Studio 3.0 后去除此项配置。

##### defaultConfig{} 闭包 #####
**`defaultConfig{}：`**闭包用于配置 Android 插件应用于所有构建变体的变体属性。`defaultConfig{}` 主要包含以下配置：
 - **`applicationId：`**指定项目的包名，在创建项目的时候已经指定了包名，当要修改整个项目的包名时可以在此更改。
 - **`minSdkVersion：`**指定项目最低兼容的版本，如果设备低于这个版本将无法安装这个应用。
 - **`targetSdkVersion：`**指定项目的目标版本，表示在该目标版本上已经做过充分测试，系统会为该应用启动一些对应该目标系统的最新功能特性，Android 系统平台的行为变更，只有 `targetSdkVersion` 的属性值被设置为大于或等于该系统平台的 API 版本时，才会生效。
 - **`versionCode：`**指定项目的版本号，一般每次打包上线时该值只能增加，打包后看不见。
 - **`versionName：`**指定项目的版本名称，展示在应用市场上。
 - **`testInstrumentationRunner：`**指定项目的测试运行程序类名。

##### buildTypes{} 闭包 #####
**`buildTypes{}：`**闭包主要用于配置项目的所有构建类型。`buildTypes{}` 一般包含两个子闭包，一个是 `debug` 闭包，用于指定生成测试版安装文件的配置，可以忽略不写；另一个是 `release` 闭包，用于指定生成正式版安装文件的配置。两者能配置的参数相同，最大的区别默认属性配置不一样。
详细的配置内容如下：
```gradle
buildTypes {
    release {
        buildConfigField("boolean", "LOG_DEBUG", "false")
        buildConfigField("String", "URL_PERFIX", "\"https://release.cn/\"")
        minifyEnabled false
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        signingConfig signingConfigs.release
        pseudoLocalesEnabled false
        zipAlignEnabled true
        applicationIdSuffix 'release'
        versionNameSuffix 'release'
    }
    debug {
        buildConfigField("boolean", "LOG_DEBUG", "true")
        buildConfigField("String", "URL_PERFIX", "\"https://test.com/\"")
        minifyEnabled false
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        signingConfig signingConfigs.debug
        debuggable false
        jniDebuggable false
        renderscriptDebuggable false
        zipAlignEnabled true
        pseudoLocalesEnabled false
        applicationIdSuffix 'debug'
        versionNameSuffix 'debug'
    }
}
```
**`buildTypes{}：`**主要包含以下配置：
 - **`minifyEnabled：`**指定是否对代码进行混淆，true 表示对代码进行混淆，false表示对代码不进行混淆，默认的是 false。
 - **`proguardFiles：`**指定混淆的规则文件，这里指定了 proguard-android.txt 文件和 proguard-rules.pro 文件两个文件，proguard-android.txt 文件为默认的混淆文件，里面定义了一些通用的混淆规则。proguard-rules.pro 文件位于当前项目的根目录下，可以在该文件中定义一些项目特有的混淆规则。
 - **`buildConfigField：`**用于解决 Beta 版本服务和 Release 版本服务地址不同或者一些 Log 打印需求控制的。例如：配置buildConfigField("boolean", "LOG_DEBUG", "true")，这个方法接收三个非空的参数，第一个：确定值的类型，第二个：指定key的名字，第三个：传值，调用的时候BuildConfig.LOG_DEBUG即可调用。
 - **`debuggable：`**指定是否支持断点调试，release 默认为 false，debug 默认为 true。
 - **`jniDebuggable：`**指定是否可以调试 NDK 代码，使用 lldb 进行 C 和 C++ 代码调试，release 默认为 false
 - **`signingConfig：`**指定签名信息，通过 signingConfigs.release 或者 signingConfigs.debug，配置相应的签名，但是添加此配置前必须先添加 signingConfigs 闭包，添加相应的签名信息。
 - **`renderscriptDebuggable：`**指定是否开启渲染脚本就是一些 C 写的渲染方法，默认为 false。
 - **`renderscriptOptimLevel：`**指定渲染等级，默认是 3。
 - **`pseudoLocalesEnabled：`**指定是否在 APK 中生成伪语言环境，帮助国际化的东西，一般使用的不多。
 - **`applicationIdSuffix：`**指定添加 applicationId 的后缀，一般使用的不多。
 - **`versionNameSuffix：`**指定添加版本名称的后缀，一般使用的不多。
 - **`zipAlignEnabled：`**指定是否对 APK 包执行 ZIP 对齐优化，减小 ZIP 体积，增加运行效率，release 和 debug 默认都为true。

#### dependencies{} 闭包 ####
**`dependencies{}：`**闭包主要用于配置项目的依赖项。一般 Android 项目中一共有以下三种依赖方式：
 - **`库依赖：`**可以对项目中的库模块添加依赖关系。
 - **`本地依赖：`**可以对本地的 Jar 包或目录添加依赖关系
 - **`远程依赖：`**则可以对远程仓库上的开源项目添加依赖关系。

各种依赖方式的代码如下：
```gradle
// 本地依赖声明
implementation fileTree(dir: 'libs', include: ['*.jar'])
// 库依赖声明
implementation project(':library')
// 远程依赖声明
implementation 'com.android.support:appcompat-v7:28.0.0'
```

