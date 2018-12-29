---
title: Android Plugin for Gradle 3.0.0
categories: Android
tags:
  - Android
  - Gradle
abbrlink: 2f48e782
date: 2018-12-24 18:26:45
---

Android Studio 升级到 3.0 之后，Gradle Plugin 和 Gradle 不升级也是可以继续使用的，但很多新的特性如：Java 8 支持、新的依赖匹配机制、AAPT2 等新功能都无法正常使用~ 所以长期看来，最后还是得升级的。

## 升级 Gradle 配置 ##
Gradle plugin 3.0.0 要求 Gradle 4.1 或者更高的版本。

### 更新 Gradle 版本 ###
如果正在使用 Android Studio 3.0 或更高版本打开现有项目，请按照提示操作，将现有项目自动更新到兼容版本的 Gradle。要手动更新 Gradle，请更新 `/gradle/wrapper` 目录下的 `gradle-wrapper.properties` 文件中的网址，如下所示：
```gradle
distributionUrl=https\://services.gradle.org/distributions/gradle-4.1-all.zip
```

### 配置 Gradle Plugin ###
如果正在使用 Android Studio 3.0 或更高版本打开现有项目，请按照提示操作，将现有项目自动更新到最新版本的 Android Gradle Plugin。要手动更新 Gradle Plugin，请在项目根目录下的 `build.gradle` 文件中更改插件版本并添加 Maven 仓库，如下所示：
```gradle
buildscript {
    repositories {
        ...
        // You need to add the following repository to download the new plugin.
        google()
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.0-beta4'
    }
}
```

## flavor dimensions ##
Android Studio 3.0 之前，进行多模块依赖开发的情况下，项目是正常运行的，然而升级到 3.0 之后，原本的项目就出现了问题，具体问题如下：
```
All flavors must now belong to a named flavor dimension. Learn more at https://d.android.com/r/tools/flavorDimensions-missing-error-message.html
```

要解决此错误，请将每个风味分配至一个给定维度，如下面的示例中所示。由于依赖项匹配现在由插件处理，应当谨慎地为风味维度命名。
```gradle
// Specifies a flavor dimension.
flavorDimensions "color"

productFlavors {
     red {
      // Assigns this product flavor to the 'color' flavor dimension.
      // This step is optional if you are using only one dimension.
      dimension "color"
      ...
    }

    blue {
      dimension "color"
      ...
    }
}
```

> 在 `android{}` 里面加入 `flavorDimensions` 定义风味维度，然后在 `productFlavors` 中中的每个 `flavor` 指定所属的风味维度。

## 使用新依赖项配置 ##
Gradle 3.4 引入了新的 [Java 库插件配置](https://docs.gradle.org/current/userguide/java_library_plugin.html#sec:java_library_configurations_graph)，允许您控制到编译和运行时类路径的发布（适用于模块间依赖项）。Android Plugin 3.0.0 正在迁移到这些新依赖项配置。要迁移项目，只需更新依赖项以使用新配置，而非已弃用配置，如下表中所列：

| 新配置         | 已弃用配置 | 行为                                                                                                                                                                                                                              |
|----------------|------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| implementation | compile    | 依赖项在编译时对模块可用，并且仅在运行时对模块的消费者可用。对于大型多项目构建，使用 `implementation` 而不是 `api/compile` 可以显著缩短构建时间，因为它可以减少构建系统需要重新编译的项目量。大多数应用和测试模块都应使用此配置。 |
| api            | compile    | 依赖项在编译时对模块可用，并且在编译时和运行时还对模块的消费者可用。此配置的行为类似于 `compile`（现在已弃用），一般情况下，您应当仅在库模块中使用它。应用模块应使用 `implementation`，除非想要将其 API 公开给单独的测试模块。    |
| compileOnly    | provided   | 依赖项仅在编译时对模块可用，并且在编译或运行时对其消费者不可用。此配置的行为类似于 `provided`（现在已弃用）。                                                                                                                     |
| runtimeOnly    | apk        | 依赖项仅在运行时对模块及其消费者可用。此配置的行为类似于 `apk`（现在已弃用）。                                                                                                                                                    |

Gralde 插件 3.0 之前的 `build.gradle` 文件中依赖项默认都是 `compile`，3.0 之后的依赖项默认是 `implementation`。

> **注：** `compile`、`provided` 和 `apk` 目前仍然可用。不过，它们将在下一个主要版本的 Android 插件中消失。

### Gradle 依赖配置指令 ###
 - **`implementation(compile)：`**只在内部使用此 Module，不会向外部暴露其依赖的 Module 内容。
 - **`api(compile)：`**和 compile 的作用一样，当前 Module 会暴露其依赖的其他 Module 内容。使用该方式依赖的库将会参与编译和打包。
 - **`compileOnly(provided)：`**使用该方式依赖的库只在编译时有效，不会参与打包 。
 - **`runtimeOnly(apk)：`**使用该方式依赖的库只在生成 APK 的时候参与打包，编译时不会参与。
 - **`testImplementation(testCompile)：`**只在单元测试代码的编译以及最终打包测试 APK 时有效。
 - **`debugImplementation(debugCompile)：`**只在 debug 模式的编译和最终的 Debug APK 打包时有效。
 - **`releaseImplementation(releaseCompile)：`**仅仅针对 release 模式的编译和最终的 Release APK 打包。

### implementation 和 api的区别 ###
 - **`implementation：`**该命令编译的依赖，对该项目有依赖的项目将无法访问到使用该命令编译的依赖中的任何程序，也就是将该依赖隐藏在内部，而不对外部公开，不具有依赖的传递性。Gradle 构建速度比 api 要快。
 - **`api：`**等同于 compile 指令，该操作命令会修改外部接口，Module 和所有依赖的 Module 都需要重新编译，具有依赖传递性。Gradle 构建速度比 implementation 慢。

## 使用注解处理器依赖项配置 ##
在之前版本的 Android Plugin for Gradle 中，编译类路径中的依赖项将自动添加到处理器类路径中。也就是说，可以向编译类路径中添加一个注解处理器，它将按预期工作。不过，由于向处理器添加了大量不需要的依赖项，这会对性能造成显著影响。

使用新插件时，必须使用 **`annotationProcessor`** 依赖项配置将注解处理器添加到处理器类路径中，如下所示：
```gradle
dependencies {
    ...
    annotationProcessor 'com.google.dagger:dagger-compiler:<version-number>'
}
```

如果 JAR 文件包含以下文件，插件会将依赖项假设为注解处理器：`META-INF/services/javax.annotation.processing.Processor`。如果插件在编译类路径中检测到注解处理器，构建将失败，并且可以看到一条错误消息，其中列出了编译类路径中的每一个注解处理器。要修复错误，只需更改这些依赖项的配置以使用 `annotationProcessor`。如果依赖项包含也需要位于编译类路径中的组件，请再次声明该依赖项并使用 `compile` 依赖项配置。

> **android-apt 插件用户：**此行为变更目前不影响 `android-apt` 插件。不过，插件将不会与未来版本的 Android Plugin for Gradle 兼容。

### 停用错误检查 ###
如果编译类路径中的依赖项包含不需要的注解处理器，可以将以下代码添加到 `build.gradle` 文件中，停用错误检查。请记住，添加到编译类路径中的注解处理器仍不会添加到处理器类路径中。
```gradle
android {
    ...
    defaultConfig {
        ...
        javaCompileOptions {
            annotationProcessorOptions {
                includeCompileClasspath false
            }
        }
    }
}
```

## API 变更 ##
Android Plugin 3.0.0 引入了一些移除特定功能的 API 变更，可能会破坏您现有的构建。后期版本的插件可能会引入新的公共 API 来取代破坏的功能。

### 变体输出 ###
使用 Variant API 操作变体输出在新插件中已不再受支持。不过，这种方式仍然适用于简单任务，例如在构建时更改 APK 名称，如下所示：
```gradle
// If you use each() to iterate through the variant objects,
// you need to start using all(). That's because each() iterates
// through only the objects that already exist during configuration time—
// but those object don't exist at configuration time with the new model.
// However, all() adapts to the new model by picking up object as they are
// added during execution.
android.applicationVariants.all { variant ->
    variant.outputs.all {
        outputFileName = "${variant.name}-${variant.versionName}.apk"
    }
}
```

不过，涉及访问 `outputFile` 对象的更复杂任务不再奏效。这是因为配置阶段期间不再创建变体特定的任务。这就让插件无法提前了解其全部输出，不过这会加快配置速度。

### manifestOutputFile ###
`processManifest.manifestOutputFile()` 函数不再可用，如果您尝试调用该函数，您将遇到以下错误：
```
A problem occurred configuring project ':myapp'.
   Could not get unknown property 'manifestOutputFile' for task ':myapp:processDebugManifest'
   of type com.android.build.gradle.tasks.ProcessManifest.
```

可以调用 `processManifest.manifestOutputDirectory()` 来返回包含所有已生成 manifest 的目录的路径，而不是调用 `manifestOutputFile()` 来获取每个变体的 manifest 文件。然后，可以找到某个 manifest 并向其应用自己的逻辑。

下面的示例可以在 manifest 中动态更改版本代码：
```gradle
android.applicationVariants.all { variant ->
    variant.outputs.all { output ->
        output.processManifest.doLast {
            // Stores the path to the maifest.
            String manifestPath = "$manifestOutputDirectory/AndroidManifest.xml"
            // Stores the contents of the manifest.
            def manifestContent = file(manifestPath).getText()
            // Changes the version code in the stored text.
            manifestContent = manifestContent.replace('android:versionCode="1"',
                    String.format('android:versionCode="%s"', generatedCode))
            // Overwrites the manifest with the new text.
            file(manifestPath).write(manifestContent)
        }
    }
}
```

