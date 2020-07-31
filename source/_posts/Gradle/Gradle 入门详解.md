---
title: Gradle 入门详解
categories: Gradle
tags:
  - Gradle
abbrlink: 8317ffc8
date: 2019-06-10 18:30:55
---

Java 作为一门世界级主流编程语言，有一款高效易用的项目管理工具是 Java 开发者想要的。先是 2000 年的 Ant，后有 2004 年 Maven 的诞生，都在 Java 市场上取得了巨大的成功。但二者都有一定的不足和局限性。

2012年基于 Ant 和 Maven 产生的 Gradle，弥补了 Ant 和 Maven 的不足，带来了一些更高效的特点。它使用了一种更高效的特点。它使用一种基于 Groovy 的特定领域语言（DSL）来声明项目设置，抛弃了基于 XML 的各种繁琐配置。

## Gradle是什么？ ##
**`Gradle`**是一个基于 `Apache Ant` 和 `Apache Maven` 概念的项目自动化**`构建工具`**。它使用一种基于 `Groovy` 的特定领域语言来声明项目设置，而不是传统的 `XML`。当前其支持的语言限于 `Java`、`Groovy` 和 `Scala`，计划未来将支持更多的语言。

可能刚接触 `Gradle` 的同学都不是很了解 `Gradle` 的这个定义。可能就只会跟着网上的教程 copy 一点配置，但是不理解这些配置背后的原理。那么怎么来理解这句话呢，我们可以把握到三个要点：
 - 首先，Gradle 是一种 **`构建工具`**；
 - 其次，Gradle 是 **`基于 maven 概念`**的；
 - 最后，Gradle 使用 **`Groovy`** 语言来声明。

> Gradle 官网：[https://gradle.org/](https://gradle.org/)

## 为什么使用Gradle？ ##
`Gradle` 的核心是在基于 `Groovy` 对 `Domain Specific Language(DSL)` 语言进行一个丰富的扩展。`Gradle` 具有以下优点：
 - 更容易重用资源和代码；
 - 可以更容易创建不同版本的程序；
 - 更容易配置、扩展；
 - 更好的 IDE 集成。

## Gradle基础配置 ##
首先明确 `Gradle` 跟 `Maven` 一样，也有一个配置文件，`Maven` 里面是叫 `pom.xml`，而在 `Gradle` 中是叫 `build.gradle`。`Android Studio` 中的 Android 项目通常至少包含两个 `build.gradle` 文件，一个是 `Project` 范围的，另一个是 `Module` 范围的，由于一个 `Project` 可以有多个 `Module`，所以每个 `Module` 下都会对应一个 `build.gradle`。

`Project` 下的 `build.gradle` 是基于整个 `Project` 的配置，而 `Module` 下的 `build.gradle` 是每个模块自己的配置。

### project#build.gradle ###
```gradle
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {    // 构建过程依赖的仓库
        google()
        jcenter()
    }
    dependencies {    // 构建过程需要依赖的库
        classpath 'com.android.tools.build:gradle:3.5.3'
        
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {         // 整个项目依赖的仓库
    repositories {
        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```
> 注：大家可能很奇怪，为什么仓库 `repositories` 需要声明两次，这其实是由于它们作用不同，`buildscript` 中的仓库是 `gradle` 脚本自身需要的资源，而 `allprojects` 下的仓库是项目所有模块需要的资源。所以大家千万不要配错了。

### module#build.gradle ###
```gradle
// 声明插件，这是一个android程序，如果是android库，应该是com.android.library
apply plugin: 'com.android.application'

android {
    compileSdkVersion 29                 // 编译版本
    buildToolsVersion "29.0.2"           // buildtool版本
    defaultConfig {                      // 默认配置
        applicationId "com.gradle.test"  // 包名
        minSdkVersion 14
        targetSdkVersion 29
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
    buildTypes {                         // 构建配置(混淆、签名等配置)         
        release {
            minifyEnabled true
            zipAlignEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
        debug {
            debuggable true
            minifyEnabled false
        }
    }
}

// 模块依赖
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])  // 依赖libs目录下所有jar包
    implementation 'androidx.appcompat:appcompat:1.0.2'       // 依赖appcompat库
}
```

## Gradle相关配置 ##
现在大家对 `build.gradle` 已经初步了解了，我们再看下其他一些与 `gradle` 相关的文件。

### gradle.properties ###
在使用 Android Studio 新建 Android 项目之后，在项目根目录下会默认生成一个 `gradle.properties` 文件，我们可以在里面做一些 `Gradle` 文件的全局性的配置，也可以将比较私密的信息放在里面，防止泄露。

`gradle.properties` 里面定义的属性是全局的，可以在各个模块的 `build.gradle` 里面直接引用，因此可以在这里面可以定义一些常量供 `build.gradle` 使用。

还可以将一些构建环境的配置信息放在 `gradle.properties` 文件中，如果想了解更多如何配置编译环境，可以访问[Build Environment](https://docs.gradle.org/current/userguide/build_environment.html)

### settings.gradle ###
`settings.gradle` 文件是用来配置多模块的，比如你的项目有两个模块app,library,那么你就需要在这个文件中进行配置，格式如下：
```gradle
include ':app',':library'
rootProject.name='My Application'
```

### gradle目录 ###
`gradle 目录`下有个 `wrapper` 目录，里面有两个文件，`gradle-wrapper.jar` 和 `gradle-wrapper.properties`，它们就是 `gradle wrapper`。`Gradle` 项目都会有，你可以通过命令 `gradle init` 来创建它们(前提是本地安装了 gradle 并且配置到了环境变量中)。

`gradle-wrapper.properties` 配置本地路径，配置如下：
```gradle
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-5.4.1-all.zip
```
各配置项作用如下：
 - distributionBase：下载的 Gradle压缩包解压后存储的主目录
 - distributionPath：相对于 distributionBase 的解压后的 Gradle 压缩包的路径
 - zipStoreBase：同 distributionBase，只不过是存放 zip 压缩包的
 - zipStorePath：同 distributionPath，只不过是存放 zip 压缩包的
 - distributionUrl：Gradle 发行版压缩包的下载地址

### gradlew和gradlew.bat ###
`gradlew` 和 `gradlew.bat` 分别是 `linux` 下的 shell 脚本和 `windows` 下的批处理文件，它们的作用是根据 `gradle-wrapper.properties` 文件中的 `distributionUrl` 下载对应的 `gradle` 版本。这样就可以保证在不同的环境下构建时都是使用的统一版本的 `gradle`，即使该环境没有安装 `gradle` 也可以，因为 `gradle wrapper` 会自动下载对应的 `gradle` 版本。

> `gradlew` 的用法跟 `gradle` 一模一样，比如执行构建 `gradle build` 命令，你可以用 `gradlew build`。`gradlew` 即 `gradle wrapper` 的缩写。

### gradle仓库 ###
`gradle` 有三种仓库：`maven`仓库，`ivy` 仓库以及 `flat` 本地仓库。声明方式如下：
```gradle
maven { url 'xxx' }
ivy { url 'xxx' }
flatDir { dirs 'xxx' }
```

有一些仓库提供了别名，可直接使用：
```gradle
google()
jcenter()
mavenLocal()
mavenCentral()
```

### gradle任务 ###
`gradle` 中有一个核心概念叫任务，跟 `maven` 中的插件目标类似。

`gradle` 的 `android`插件提供了四个顶级任务：
 - check：运行检测和测试任务
 - build：运行 assemble 和 check
 - clean：清理输出任务
 - assemble：构建项目输出

执行任务可以通过 `gradle/gradlew` + 任务名称的方式执，执行一个顶级任务会同时执行与其依赖的任务，比如执行下面的任务：
```gradle
gradlew assemble
```
它通常会执行：
```gradle
gradlew assembleDebug
gradlew assembleRelease
```
这时会在你项目的 `build/outputs/apk` 或者 `build/outputs/aar` 目录生成输出文件。

可以通过下面的命令列出所有可用的任务：
```gradle
gradlew tasks
```
在 Android Studio 中可以打开右侧 gradle 视图查看所有任务。

