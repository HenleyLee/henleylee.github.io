---
title: Android Studio 提高 Gradle 构建速度的小技巧
categories: Android
tags:
  - Android
  - Android Studio
abbrlink: 3f20da69
date: 2019-05-28 12:20:15
---

长构建时间会减慢开发过程，因此本文将介绍一些可以帮助解决构建速度瓶颈的技巧。

提高构建速度的一般过程如下所示：
 - 采取一些可以立即为大多数 Android Studio 项目带来好处的措施，优化构建配置。
 - 分析构建配置，识别并诊断一些对项目或工作站来说比较棘手的瓶颈。

在开发应用程序时，应尽可能将其部署到运行 Android 7.0 (API级别24)或更高的设备上。Android 平台的新版本实现了更好的机制来推送更新到您的应用程序，比如 [Android Runtime (ART)](https://source.android.google.cn/devices/tech/dalvik/) 和对 [multiple DEX files](https://developer.android.google.cn/studio/build/multidex.html) 的本机支持。

> 注意：在第一次完全清理构建之后，您可能会注意到后续构建（清理和增量）的执行速度更快（即使不使用此页面中描述的任何优化）。 这是因为 Gradle 守护程序与其他 JVM 进程类似，具有提升性能的“预热”期。

在开启速度提升调优之前，先了解三个性能指标的说明：
 - 全量构建，也就是重新开始编译整个工程的 debug 版；
 - 代码增量构建，指的是我们修改了工程的 Java / Kotlin 代码；
 - 资源增量构建，指的是我们对资源文件的修改，增加减少了图片和字符串资源等。

## 优化构建配置 ##
请遵循以下提示以提高 Android Studio 项目的构建速度。

### 使用最新版本的开发工具 ###
Android 工具的每次更新都会修复大量的 bug 及提升性能等新特性，因此保持 Android 工具的最新版本有非常大的必要。要充分利用最新优化，请保持以下工具处于最新版本：
 - [Android Studio and SDK tools](https://developer.android.google.cn/studio/intro/update.html)
 - [The Android plugin for Gradle](https://developer.android.google.cn/studio/releases/gradle-plugin.html)

### 为开发创建构建变体 ###
准备发布应用时需要的许多配置在开发应用时都是不需要的。启用不必要的构建进程会减慢您的增量构建和清理构建速度，因此，在开发应用期间，配置一个 [build variant](https://developer.android.com/studio/build/build-variants.html)，使之仅包含需要的构建配置。下面的示例将为您的发布版本配置创建一个 *dev* 变体和一个 *prod* 变体：
```gradle
android {
  ...
  defaultConfig {...}
  buildTypes {...}
  productFlavors {
    // When building a variant that uses this flavor, the following configurations
    // override those in the defaultConfig block.
    dev {
      // To avoid using legacy multidex when building from the command line,
      // set minSdkVersion to 21 or higher. When using Android Studio 2.3 or higher,
      // the build automatically avoids legacy multidex when deploying to a device running
      // API level 21 or higher—regardless of what you set as your minSdkVersion.
      minSdkVersion 21
      versionNameSuffix "-dev"
      applicationIdSuffix '.dev'
    }

    prod {
      // If you've configured the defaultConfig block for the release version of
      // your app, you can leave this block empty and Gradle uses configurations in
      // the defaultConfig block instead. You still need to create this flavor.
      // Otherwise, all variants use the "dev" flavor configurations.
    }
  }
}
```

如果构建配置已使用产品风味来创建应用程序的不同版本，则可以使用 [flavor dimensions](https://developer.android.com/studio/build/build-variants.html#flavor-dimensions) 将 *dev* 和 *prod* 配置与这些风味组合。
例如，如果您已经配置了 *demo* 和 *full* 风味，则可以使用以下示例配置来创建组合风味，例如 *devDemo* 和 *prodFull*：
```gradle
android {
  ...
  defaultConfig {...}
  buildTypes {...}

  // Specifies the flavor dimensions you want to use. The order in which you
  // list each dimension determines its priority, from highest to lowest,
  // when Gradle merges variant sources and configurations. You must assign
  // each product flavor you configure to one of the flavor dimensions.

  flavorDimensions "stage", "mode"

  productFlavors {
    dev {
      dimension "stage"
      minSdkVersion 21
      versionNameSuffix "-dev"
      applicationIdSuffix '.dev'
      ...
    }

    prod {
      dimension "stage"
      ...
    }

    demo {
      dimension "mode"
      ...
    }

    full {
      dimension "mode"
      ...
    }
  }
}
```

### 启用单一变体同步项目 ###
将项目与构建配置同步是让 Android Studio 了解项目结构的重要一步。但是，对于大型项目而言，此过程可能非常耗时。如果项目中使用多个构建变体，现在可以通过将项目同步限制为仅当前选择的变体来优化项目同步。

需要使用 Android Studio 3.3 或更高版本和 Android Gradle Plugin 3.3.0 或更高版本来实现此优化。默认情况下，所有项目都启用了优化。

要手动启用此优化，请单击 **`File > Settings > Experimental > Gradle (Android Studio > Preferences > Experimental > Gradle on a Mac)`**，并选择 **`Only sync the active variant`** 复选框。

> 注意：此优化完全支持包含 `Java` 和 `C++` 语言的项目，并且对 `Kotlin` 有一些支持。 在为具有 `Kotlin` 内容的项目启用优化时，Gradle 同步将回退到内部使用完整变体。

### 避免编译不必要的资源 ###
避免编译和打包没有测试的资源（例如其他语言本地化和屏幕密度资源）。可以通过仅为 *dev* 风味指定一种语言资源和屏幕密度来实现，如下面的示例所示：
```gradle
android {
  ...
  productFlavors {
    dev {
      ...
      // The following configuration limits the "dev" flavor to using
      // English stringresources and xxhdpi screen-density resources.
      resConfigs "en", "xxhdpi"
    }
    ...
  }
}
```

### 为调试构建禁用 Crashlytics ###
如果不需要运行 [Crashlytics report](https://docs.fabric.io/android/crashlytics/overview.html)，请通过禁用插件来提高调试构建速度，如下所示：
```gradle
android {
  ...
  buildTypes {
    debug {
      ext.enableCrashlytics = false
    }
}
```

还需要在运行时为调试版本禁用 Crashlytics 工具包，方法是更改在应用程序中初始化 Fabric 支持的方式，如下所示：
```java
// Initializes Fabric for builds that don't use the debug build type.
Crashlytics crashlyticsKit = new Crashlytics.Builder()
    .core(new CrashlyticsCore.Builder().disabled(BuildConfig.DEBUG).build())
    .build();

Fabric.with(this, crashlyticsKit);
```

如果希望将 Crashlytics 与调试版本一起使用，仍然可以通过防止 Crashlytics 在每次构建期间使用自己的唯一构建 ID 更新应用程序资源来加速增量构建。要防止 Crashlytics 不断更新其构建 ID，请将以下内容添加到 `build.gradle` 文件中：
```gradle
android {
  ...
  buildTypes {
    debug {
      ext.alwaysUpdateBuildId = false
    }
}
```
有关在使用 [Crashlytics](https://docs.fabric.io/android/crashlytics/overview.html) 时优化构建的更多信息，请阅读[官方文档](https://docs.fabric.io/android/crashlytics/build-tools.html)。

### 在调试构建中使用静态构建配置 ###
始终对清单文件中的属性或调试构建类型的资源文件中的属性使用静态/硬编码值。如果清单文件或应用程序资源中的值需要在每次构建时更新，那么 `Instant Run` 就不能执行代码切换，它必须构建和安装新的 APK。

例如，使用动态版本号、版本名称、资源或任何其他更改清单文件的构建逻辑，每次要运行更改时都需要完整的 APK 构建，即使实际更改可能只需要热交换。如果构建配置需要这样的动态属性，那么将它们隔离到您的发布版本变体并保持调试版本的静态值，如下面的 `build.gradle` 文件所示：
```gradle
int MILLIS_IN_MINUTE = 1000 * 60
int minutesSinceEpoch = System.currentTimeMillis() / MILLIS_IN_MINUTE

android {
    ...
    defaultConfig {
        // Making either of these two values dynamic in the defaultConfig will
        // require a full APK build and reinstallation because the AndroidManifest.xml
        // must be updated (which is not supported by Instant Run).
        versionCode 1
        versionName "1.0"
        ...
    }

    // The defaultConfig values above are fixed, so your incremental builds don't
    // need to rebuild the manifest (and therefore the whole APK, slowing build times).
    // But for release builds, it's okay. So the following script iterates through
    // all the known variants, finds those that are "release" build types, and
    // changes those properties to something dynamic.
    applicationVariants.all { variant ->
        if (variant.buildType.name == "release") {
            variant.mergedFlavor.versionCode = minutesSinceEpoch;
            variant.mergedFlavor.versionName = minutesSinceEpoch + "-" + variant.flavorName;
        }
    }
}
```

### 使用静态依赖版本 ###
在 `build.gradle` 文件中声明依赖项时，您应当避免在结尾将版本号与加号一起使用，例如 `'com.android.tools.build:gradle:2.+'`。使用动态版本号可能会导致意外的版本更新，难以解决版本差异，并因 Gradle 检查有无更新而减慢构建速度，应改为使用静态/硬编码版本号。

### 启用离线模式 ###
如果网络连接速度比较慢，那么在 Gradle 尝试使用网络资源解析依赖项时，构建时间可能会受到影响。可以指定 Gradle 仅使用它已经缓存到本地的工件来避免使用网络资源。

要在使用 Android Studio 构建时离线使用 Gradle，请执行以下操作：
 - 点击 **File > Settings**（在 Mac 上，点击 **Android Studio > Preferences**），打开 **Preferences** 窗口。
 - 在左侧窗格中，点击 **Build, Execution, Deployment > Gradle**。
 - 勾选 **Offline work** 复选框。
 - 点击 **Apply** 或 **OK**。

如果正在从命令行构建，请传递 `--offline` 选项。

### 创建库模块 ###
在应用中查找可以转换为 [Android library module](https://developer.android.com/studio/projects/android-library.html) 的代码。通过这种方式将代码模块化可以让构建系统仅编译您修改的模块，并缓存这些输出以用于后面的构建。当启用[并行项目执行](https://docs.gradle.org/current/userguide/multi_project_builds.html#sec:parallel_execution)时，这种方式也会让并行项目执行更有效。要了解更多信息，请阅读 [Gradle 官方文档](https://docs.gradle.org/current/userguide/more_about_tasks.html)。

### 为自定义构建逻辑创建任务 ###
在创建构建分析后，如果分析显示构建时间中相当大的一部分用在了“配置项目”阶段，请检查 `build.gradle` 脚本并查找可以添加到自定义 Gradle 任务中的代码。将某个构建逻辑移动到任务中后，它仅会在需要时运行，可以为后续构建缓存结果，并且该构建逻辑将有资格并行运行（如果启用并行项目执行）。要了解更多信息，请阅读 [Gradle 官方文档](https://docs.gradle.org/current/userguide/more_about_tasks.html)。

> Tip：如果构建包含大量自定义任务，则可能需要通过[创建自定义任务类](https://docs.gradle.org/current/userguide/custom_tasks.html)来对 `build.gradle` 文件进行整理。 将类添加到 `project-root/buildSrc/src/main/groovy/` 目录中，Gradle 会自动将它们包含在项目中所有 `build.gradle` 文件的类路径中。

### 将图像转换成 WebP ###
[WebP](https://developers.google.cn/speed/webp/) 是一种既可以提供有损压缩(像 JPEG 一样)也可以提供透明度(像 PNG 一样)的图片文件格式，不过与 `JPEG` 或 `PNG` 相比，这种格式可以提供更好的压缩。降低图片文件大小可以加快构建的速度（无需执行构建时压缩），尤其是在应用使用大量图像资源时，更是如此。不过，在对 `WebP` 图像进行解压缩时，您可能会注意到设备的 `CPU` 使用率有小幅上升。使用 Android Studio 时，可以轻松地[将图像转换成 WebP](https://developer.android.google.cn/studio/write/convert-webp.html#convert_images_to_webp)。

### 禁用 PNG 处理 ###
如果不能(或不想)将 PNG 图像转换为 WebP，仍然可以通过在每次构建应用程序时禁用自动图像压缩来加速构建。如果使用的是 [Android plugin 3.0.0](https://developer.android.google.cn/studio/releases/gradle-plugin.html#3-0-0) 或更高版本，则默认情况下仅对 `debug` 构建类型禁用 PNG 处理。要为其他构建类型禁用此优化，请将以下内容添加到 `build.gradle` 文件中：
```gradle
android {
    buildTypes {
        release {
            // Disables PNG crunching for the release build type.
            crunchPngs false
        }
    }

// If you're using an older version of the plugin, use the
// following:
//  aaptOptions {
//      cruncherEnabled false
//  }
}
```

由于构建类型或产品风味不定义此属性，因此在构建应用程序的发行版本时，需要手动将此属性设置为 `true`。

### 启用 Instant Run ###
[Instant Run](https://developer.android.google.cn/studio/run/index.html#instant-run) 可以在不构建新 APK 的情况下（某些情况下，甚至不需要重启当前 Activity）推送特定代码和资源更改，从而显著缩短更新您的应用所需的时间。在对应用进行更改后，点击 **Apply Changes** 以使用 `Instant Run`，此功能会在执行以下操作时默认启用：
 - 使用 `debug` 构建变体构建应用。
 - 使用 [Android Plugin for Gradle](https://developer.android.google.cn/studio/releases/gradle-plugin.html) 2.3.0 或更高版本。
 - 在应用的模块级 `build.gradle` 文件中将 `minSdkVersion` 设置为 15 或更高。
 - 点击 `Run`，将应用部署到运行 Android 5.0(API 级别 21)及更高版本的设备上。

如果满足这些要求后在工具栏中没有看到 **Apply Changes** 按钮，请按以下步骤操作，确保未在 `IDE` 设置中停用 `Instant Run`：
 - 打开 **Settings** 或者 **Preferences** 对话框。
 - 导航至 **Build, Execution, Deployment > Instant Run**。
 - 确保勾选 **Enable Instant Run**。

### 启用构建缓存 ###
构建缓存可以存储 `Android Plugin for Gradle` 在构建项目时生成的特定输出（例如未打包的 AAR 和 pre-dexed 远程依赖项）。使用缓存时，清理构建将显著加快，因为构建系统在后续构建期间可以直接重用这些缓存文件，而不用重新创建它们。

使用 [Android plugin](https://developer.android.google.cn/studio/releases/gradle-plugin.html#revisions) 2.3.0 及更高版本的新项目在默认情况下会启用构建缓存（除非明确停用构建缓存）。要了解详情，请阅读 [Accelerate clean builds with the build cache](https://developer.android.google.cn/studio/build/build-cache.html)。

### 使用增量注解处理器 ###
Android Gradle plugin 3.3.0 及更高版本在使用注释处理器时改进了对[增量 Java 编译](https://docs.gradle.org/4.10.1/userguide/java_plugin.html#sec:incremental_annotation_processing)的支持。因此，为了提高增量构建速度，应该更新 Android Gradle 插件，并尽可能仅使用增量注释处理器。

> 注意：此功能与 Gradle 版本 4.10.1 及更高版本兼容，Gradle 5.1 除外（请参阅 [Gradle issue #8194](https://github.com/gradle/gradle/issues/8194)）。

要开始使用，请参阅以下支持增量 Java 编译的常用注释处理器列表。要获得更完整的列表，请参阅 [GitHub issue #5277](https://github.com/gradle/gradle/issues/5277)。某些插件可能需要其他步骤才能启用优化，因此请务必阅读每个插件的文档。
 - [Glide 4.9.0 and higher](https://github.com/bumptech/glide)：Android 的媒体管理和图像加载框架，将媒体解码、内存和磁盘缓存以及资源池封装到一个简单易用的界面中。
 - [Dagger 2.18 and higher](https://github.com/google/dagger)：为依赖项注入提供了一种编译时方法。
 - [Auto Service/Value 1.6.3 and higher](https://github.com/google/auto)：Java 代码生成器的集合。

请记住，如果使用一个或多个不支持增量构建的注释处理器，则不会启用增量 Java 编译。如果需要使用不支持此优化的注释处理器，请在 `gradle.properties` 文件中包含以下标志。执行此操作时，`Android Gradle plugin` 会在单独的任务中执行所有注释处理器，并允许 Java 编译任务以递增方式运行。
```gradle
android.enableSeparateAnnotationProcessing = true
```

## 分析构建配置 ##
较大的项目或者实现许多自定义构建逻辑的项目，可能需要更深入地研究构建过程才能找到瓶颈。可以通过分析 `Gradle` 执行构建生命周期的每个阶段和每个构建任务所需的时间来完成此操作。例如，如果构建分析显示 `Gradle` 在配置项目时花费了过多的时间，可能表明需要[将自定义构建逻辑移出配置阶段](https://developer.android.google.cn/studio/build/optimize-your-build#optimize_configuration)。此外，如果 `mergeDevDebugResources` 任务占用了大量构建时间，则表明还需要[将图像转换成 WebP](https://developer.android.google.cn/studio/build/optimize-your-build#use_webp) 或者[禁用 PNG 处理](https://developer.android.google.cn/studio/build/optimize-your-build#disable_crunching)。

使用分析来提高构建速度通常包括在启用分析的情况下运行构建，对构建配置进行一些调整，并分析更多内容以观察更改的结果。

要生成和查看构建分析，请执行以下步骤：
1. 在 Android Studio 中打开项目后，选择 **View > Tool Windows > Terminal** 以在项目的根目录下打开命令行。

2. 输入以下命令执行清理构建。在对构建进行概要分析时，应当在分析的每个构建之间执行清理构建，因为当任务的输入（例如源代码）未发生变化时，Gradle 会跳过任务。因此，没有输入更改的第二次构建总会以更快的速度运行，因为没有重新运行任务。因此，在不同构建之间运行 **`clean`** 任务可以确保分析完整的构建进程。
```gradle
// On Mac or Linux, run the Gradle wrapper using "./gradlew".
gradlew clean
```

3. 使用以下标志执行某种产品风味的调试版本，例如 *dev* 风味：
```gradle
gradlew --profile --offline --rerun-tasks assembleFlavorDebug
```
 - **`--profile：`**启用分析。
 - **`--offline：`**禁止 Gradle 获取在线依赖项。这样可以确保 Gradle 在尝试更新依赖项时引起的任何延迟都不会干扰分析数据。应当已将项目构建一次，以便确保 Gradle 已经下载和缓存项目的依赖项。
 - **`--rerun-tasks：`**强制 Gradle 重新运行所有任务并忽略任何任务优化。

4. 构建完成后，使用 **Project** 窗口导航到 `project-root/build/reports/profile/` 目录。如下图所示：
![指示分析报告位置的Project视图](https://developer.android.google.cn/studio/images/build/build-report_2X.png)

5. 右键点击 **profile-`timestamp`.html** 文件，然后选择 **Open in Browser > Default**。可以检查报告中的每个标签来了解构建，例如，**Task Execution** 标签显示了 Gradle 执行每个构建任务所花费的时间。报告如下图所示：
![在浏览器中查看报告](https://developer.android.google.cn/studio/images/build/build-report-before_2X.png)

6. 可选：在对项目或构建配置进行任何更改之前，请重复第 3 步中的命令，但请忽略 `--rerun-tasks` 标志。由于 Gradle 会尝试通过不重复执行输入未发生变化的任务来节省时间（这些任务在报告的 **Task Execution** 标签中标记为 `UP-TO-DATE`，如下图所示），可以确定哪些任务在不必要的时间执行了工作。例如，如果 `:app:processDevUniversalDebugManifest` 未标记为 `UP-TO-DATE`，则可能说明构建配置会随着每个构建动态更新 `manifest`。不过，有些任务（例如 `:app:checkDevDebugManifest`）需要在每个构建期间始终运行。
![查看任务执行结果](https://developer.android.google.cn/studio/images/build/build-report-tasks_2X.png)

现在，已经有了一个构建分析报告，可以通过检查报告每个标签中的信息来寻找优化机会。有些构建设置要求实验，因为它们的好处在项目与工作站之间可能有所不同。例如，包含大型代码库的项目可能会受益于使用 [ProGuard](https://developer.android.google.cn/studio/build/shrink-code.html) 移除未使用的代码和压缩 APK 大小。不过，较小的项目则可能从完全禁用 `ProGuard` 中受益。此外，增加 Gradle 堆大小（使用 `org.gradle.jvmargs`）可能会降低小内存机器的性能。

在对构建配置进行更改后，请重复上述步骤并生成新的构建分析来观察更改的结果。例如，下图所示为相同示例应用在进行本页介绍的一些基本优化后的报告。
![在优化构建速度后查看新报告](https://developer.android.google.cn/studio/images/build/build-report-after_2X.png)

> Tips：要获得更强大的分析工具，请考虑使用 [Gradle 开源分析器](https://github.com/gradle/gradle-profiler)。

## 致谢 ##
[Optimize your build speed](https://developer.android.com/studio/build/optimize-your-build)

