---
title: Gradle 学习之 Android DSL 扩展
categories: Android
tags:
  - Android
  - Gradle
abbrlink: 5b2977c1
date: 2018-12-29 12:36:55
---

在 `Android` 项目的每个 `Module` 中的  `build.gradle` 配置文件中都有一个名称为 `android` 的函数，该函数接收闭包作为参数，然而其实在 [Gradle API 文档](https://docs.gradle.org/current/dsl/org.gradle.api.invocation.Gradle.html)中是不存在这个函数的。

那么 `android` 函数怎么会出现在这里呢？答案就是 `apply plugin: 'com.android.application'`。这个插件提供了 `Android` 构建所需要的各种 `script`。 

[Android Plugin DSL Reference](http://google.github.io/android-gradle-dsl/)：Android 插件 `DSL` 扩展文档，各个版本的都有。 

## Gradle 扩展类型 ##
`BaseExtension` 是所有 `Android` 插件的基本扩展，使用过程中不直接使用此扩展，而是使用以下之一：
 - **`AppExtension：`**用于构建 `Android app module` 的 `com.android.application` 插件扩展。
 - **`LibraryExtension：`**用于构建 `Android library module` 的 `com.android.library` 插件扩展。
 - **`TestExtension：`**用于构建 `Android test module` 的 `com.android.test` 插件扩展。
 - **`FeatureExtension：`**用于构建 `Android Instant Apps` 的 `com.android.feature` 插件扩展。

## Gradle 通用配置块 ##
`Android Plugin` 一些通用的 `Script blocks`，对于以上四种类型的扩展都支持。

### defaultConfig ###
**`defaultConfig{}`** 闭包用于配置 Android 插件应用于所有构建变体的变体属性。DSL 对象为 `DefaultConfig`。

`DefaultConfig` 具有以下属性：
 - **`applicationId：`**应用程序 ID。
 - **`applicationIdSuffix：`**应用程序 ID 后缀。
 - **`consumerProguardFiles：`**包含在发布的 AAR 中的 ProGuard 规则文件。
 - **`dimension：`**指定 product flavor 所属的 flavor 维度。
 - **`externalNativeBuild：`**指定使用 CMake 或 ndk-build 的外部 native 构建选项。
 - **`javaCompileOptions：`**配置 Java 编译器选项。
 - **`manifestPlaceholders：`**清单占位符。
 - **`multiDexEnabled：`**是否为此变体启用 Multi-Dex。
 - **`multiDexKeepFile：`**文本文件，指定将编译到主 dex 文件中的其他类。
 - **`multiDexKeepProguard：`**带有其他 ProGuard 规则的文本文件，用于确定将哪些类编译到主 dex 文件中。
 - **`ndk：`**封装 NDK 的每个变体配置，例如 ABI 过滤器。
 - **`proguardFiles：`**指定插件应使用的 ProGuard 配置文件。
 - **`signingConfig：`**签署此产品风格使用的配置。
 - **`testApplicationId：`**测试应用程序 ID。
 - **`testInstrumentationRunner：`**测试仪器运行器类名称。
 - **`testInstrumentationRunnerArguments：`**测试仪器运行器自定义参数。
 - **`vectorDrawables：`**用于配置 VectorDrawable 的构建时支持的选项。
 - **`versionCode：`**版本号。
 - **`versionName：`**版本名称。
 - **`versionNameSuffix：`**版本名称后缀。
 - **`wearAppUnbundled：`**返回是否为嵌入式磨损应用启用非捆绑模式。如果为true，则可以使应用程序从嵌入式磨损应用程序转换为直接由Play商店分发的应用程序。

### compileOptions ###
**`compileOptions{}`** 闭包用于配置 Java 编译器选项。DSL 对象为 `CompileOptions`。

`CompileOptions` 具有以下属性：
 - **`encoding：`**Java 源文件编码。
 - **`incremental：`**Java 编译是否应该使用 Gradle 的新增量模型。
 - **`sourceCompatibility：`**Java 源代码的语言级别。
 - **`targetCompatibility：`**生成的 Java 字节码的版本。

### dexOptions ###
**`dexOptions{}`** 闭包用于配置 DEX 工具选项。DSL 对象为 `DexOptions`。

`DexOptions` 具有以下属性：
 - **`additionalParameters：`**要传递给 dx 的附加参数列表。
 - **`javaMaxHeapSize：`**指定调用 dx 时的 -Xmx 值。
 - **`jumboMode：`**是否在 dx(--force-jumbo) 中启用 jumbo 模式。
 - **`keepRuntimeAnnotatedClasses：`**在遗留 multidex 的主 dex 中保留所有具有运行时注解的类。
 - **`maxProcessCount：`**可用于 dex 的最大并发进程数(默认为 4)。
 - **`preDexLibraries：`**是否预先 dex 库。这可以改善增量构建，但是干净的构建可能会更慢。
 - **`threadCount：`**运行 dx 时要使用的线程数(默认为 4)。

### aaptOptions ###
**`aaptOptions{}`** 闭包用于配置 Android Asset Packaging Tool(AAPT)的选项。DSL 对象为 `AaptOptions`。

`AaptOptions` 具有以下属性：
 - **`additionalParameters：`**要传递给 aapt 的参数列表。
 - **`cruncherEnabled：`**是否要处理 PNG(已弃用)。
 - **`cruncherProcesses：`**获得要使用的 cruncher 进程数(更多的 cruncher 进程会更快地处理文件，但需要更多的内存和CPU)。
 - **`failOnMissingConfigEntry：`**如果找不到配置条目，则强制 aapt 返回错误。
 - **`ignoreAssets：`**指定要忽略的资产的模式。
 - **`ignoreAssetsPattern：`**指定要忽略的资产的模式。
 - **`noCompress：`**不会在APK中压缩存储的文件的扩展名(添加空扩展名，即设置为`''`将禁用所有文件的压缩)。

### adbOptions ###
**`adbOptions{}`** 闭包用于配置 Android Debug Bridge(ADB)的选项。DSL 对象为 `AdbOptions`。

`AdbOptions` 具有以下属性：
 - **`installOptions：`**配置 FULL_APK 安装选项列表。
 - **`timeOutInMs：`**配置用于所有 ADB 操作的超时时间。

### dataBinding ###
**`dataBinding{}`** 闭包用于配置数据绑定库的选项。DSL 对象为 `DataBindingOptions`。

`DataBindingOptions` 具有以下属性：
 - **`addDefaultAdapters：`**是否添加默认数据绑定适配器。
 - **`enabled：`**是否启用数据绑定。
 - **`enabledForTests：`**是否为测试项目运行数据绑定代码生成
 - **`version：`**要使用的数据绑定版本。

### externalNativeBuild ###
**`externalNativeBuild{}`** 闭包用于配置使用 CMake 或 ndk-build 的外部 native 构建选项。DSL 对象为 `ExternalNativeBuild`。

`ExternalNativeBuild` 具有以下属性：
 - **`cmake：`**配置 CMake 构建选项。
 - **`ndkBuild：`**配置 ndk-build 选项。

### lintOptions ###
**`lintOptions{}`** 闭包用于配置 lint 工具的选项。DSL 对象为 `LintOptions`。

`LintOptions` 具有以下属性：
 - **`abortOnError：`**lint 是否应该在发现错误时设置进程的退出代码
 - **`absolutePaths：`**lint 是否应在错误输出中显示完整路径(默认情况下，路径相对于从中调用 lint 的路径)。
 - **`check：`**要检查的确切问题集，或者为 null 以运行默认启用的问题以及通过 LintOptions.getEnable() 和不通过问题禁用的任何问题 LintOptions.getDisable()。如果为非 null，则允许调用者修改此集合。
 - **`checkAllWarnings：`**返回 lint 是否应检查所有警告，包括默认情况下关闭的警告。
 - **`checkReleaseBuilds：`**返回 lint 是否应在发布版本期间检查致命错误(默认为 true)。如果找到严重性为 fatal 的问题，则会中止发布版本。
 - **`disable：`**要解决的问题 ID 集。允许呼叫者修改此集合。
 - **`enable：`**要启用的问题 ID 集。允许呼叫者修改此集合。要启用给定问题，请将问题 ID 添加到返回的集。
 - **`explainIssues：`**返回 lint 是否应包含问题错误的解释(请注意，HTML 和 XML 报告有意无条件地执行此操作，忽略此设置)。
 - **`htmlOutput：`**指定写入 HTML 报告的可选路径。
 - **`htmlReport：`**是否应该编写 HTML 报告(默认为 true)。
 - **`ignoreWarnings：`**返回 lint 是否仅检查错误(忽略警告)。
 - **`lintConfig：`**用作回退的默认配置文件。
 - **`noLines：`**lint 是否应在输出中包含发生错误的源行(默认为 true)。
 - **`quiet：`**返回 lint 是否应该是安静的(例如，不写信息性消息，例如写入报告文件的路径)。
 - **`severityOverrides：`**严重性覆盖的可选映射。映射从问题 ID 映射到要使用的相应严重性，必须是 fatal、error、warning 或 ignore。
 - **`showAll：`**返回 lint 是否应包含所有输出(例如，包括所有备用位置，不截断长消息等)。
 - **`textOutput：`**指定写入文本报告的可选路径。特殊值 stdout 可用于指向标准输出。
 - **`textReport：`**我们是否应该撰写文本报告(默认为 false)。
 - **`warningsAsErrors：`**返回 lint 是否应将所有警告视为错误。
 - **`xmlOutput：`**指定写入 XML 报告的可选路径
 - **`xmlReport：`**我们是否应该编写XML报告(默认为 true)。

### sourceSets ###
**`sourceSets{}`** 闭包用于配置所有变体的源集。DSL 对象为 `NamedDomainObjectContainer<AndroidSourceSet>`。

`AndroidSourceSet` 具有以下属性：
 - **`name：`**此源集的名称。
 - **`aidl：`**此源集的 Android AIDL 源目录。
 - **`assets：`**此源集的 Android Assets 目录。
 - **`java：`**将由 Java 编译器编译到类输出目录中的 Java 源代码。
 - **`jni：`**此源集的 Android JNI 源目录。
 - **`jniLibs：`**此源集的 Android JNI libs 目录。
 - **`manifest：`**此源集的 Android Manifest 文件。
 - **`renderscript：`**此源集的 Android RenderScript 源目录。
 - **`res：`**此源集的 Android 资源目录。
 - **`resources：`**要复制到 javaResources 输出目录中的 Java 资源。
 - **`compileConfigurationName：`**此源集的编译配置的名称(已弃用)。
 - **`packageConfigurationName：`**此源集的运行时配置的名称(已弃用)。
 - **`providedConfigurationName：`**此源集的仅编译配置的名称(已弃用)。

### packagingOptions ###
**`packagingOptions{}`** 闭包用于配置 APK 打包选项。DSL 对象为 `PackagingOptions`。

`PackagingOptions` 具有以下属性：
 - **`doNotStrip：`**不应删除调试符号的 native 库模式列表。
 - **`excludes：`**排除路径列表。
 - **`merges：`**在 APK 中连接并打包所有出现的模式的列表。
 - **`pickFirsts：`**选择第一个出现在 APK 中的模式列表。第一个选择模式确实打包在 APK 中，但是只有第一个出现的模式被打包。

### signingConfigs ###
**`signingConfigs{}`** 闭包用于配置可以应用于BuildType和ProductFlavor配置的签名信息。DSL 对象为 `NamedDomainObjectContainer<SigningConfig>`。

`SigningConfig` 具有以下属性：
 - **`storeFile：`**存储签名时使用的文件。
 - **`storePassword：`**存储签名时使用的密码。
 - **`storeType：`**签名时使用的存储类型。
 - **`keyAlias：`**签名时使用的密钥别名。
 - **`keyPassword：`**签名时使用的密钥密码。
 - **`v1SigningEnabled：`**是否启用了使用 JAR 签名方案(即 V1签名)进行签名。
 - **`v2SigningEnabled：`**是否启用了使用 APK 签名方案(即 V2签名)进行签名。

### buildTypes ###
**`buildTypes{}`** 闭包用于配置项目的所有构建类型。DSL 对象为 `NamedDomainObjectContainer<BuildType>`。

`BuildType` 具有以下属性：
 - **`applicationIdSuffix：`**应用程序 ID 后缀。
 - **`consumerProguardFiles：`**包含在发布的 AAR 中的 ProGuard 规则文件。
 - **`crunchPngs：`**是否要处理 PNG。
 - **`debuggable：`**此构建类型是否应生成可调试的 APK。
 - **`embedMicroApp：`**是否应使用此构建类型将链接的 Android Wear 应用程序嵌入到 Variant 中。
 - **`javaCompileOptions：`**配置 Java 编译器选项。
 - **`jniDebuggable：`**是否将此构建类型配置为生成具有可调试 JNI 代码的 APK。
 - **`manifestPlaceholders：`**清单占位符。
 - **`matchingFallbacks：`**指定当直接变体与本地模块依赖项不匹配时插件应尝试使用的构建类型的排序列表。
 - **`minifyEnabled：`**是否启用删除未使用的 Java 代码。
 - **`multiDexEnabled：`**是否为此变体启用了 Multi-Dex。
 - **`multiDexKeepFile：`**文本文件，指定将编译到主 dex 文件中的其他类。
 - **`multiDexKeepProguard：`**带有其他 ProGuard 规则的文本文件，用于确定将哪些类编译到主 dex 文件中。
 - **`name：`**此构建类型的名称。
 - **`proguardFiles：`**指定插件应使用的ProGuard配置文件。
 - **`pseudoLocalesEnabled：`**指定插件是否应为pseudolocales生成资源。
 - **`renderscriptDebuggable：`**是否将构建类型配置为生成具有可调试 RenderScript 代码的 APK。
 - **`renderscriptOptimLevel：`**renderscript编译器使用的优化级别。
 - **`shrinkResources：`**是否启用了未使用资源的缩减。默认为 false;
 - **`signingConfig：`**签名配置。
 - **`testCoverageEnabled：`**是否为此构建类型启用了测试覆盖率。
 - **`useProguard：`**指定是否始终使用 ProGuard 进行代码和资源收缩。
 - **`versionNameSuffix：`**版本名称后缀。
 - **`zipAlignEnabled：`**是否为此构建类型启用了 ZipAlign。

### productFlavors ###
**`productFlavors{}`** 闭包用于配置项目的所有产品风味。DSL 对象为 `NamedDomainObjectContainer<ProductFlavor>`。

`ProductFlavor` 具有以下属性：
 - **`applicationId：`**应用程序 ID。
 - **`applicationIdSuffix：`**应用程序 ID 后缀。
 - **`consumerProguardFiles：`**包含在发布的 AAR 中的 ProGuard 规则文件。
 - **`dimension：`**指定 product flavor 所属的 flavor 维度。
 - **`externalNativeBuild：`**指定使用 CMake 或 ndk-build 的外部 native 构建选项。
 - **`javaCompileOptions：`**配置 Java 编译器选项。
 - **`manifestPlaceholders：`**清单占位符。
 - **`matchingFallbacks：`**指定当直接变体与本地模块依赖项不匹配时，插件应尝试使用的产品样式的排序列表。
 - **`multiDexEnabled：`**是否为此变体启用 Multi-Dex。
 - **`multiDexKeepFile：`**文本文件，指定将编译到主 dex 文件中的其他类。
 - **`multiDexKeepProguard：`**带有其他 ProGuard 规则的文本文件，用于确定将哪些类编译到主 dex 文件中。
 - **`ndk：`**封装 NDK 的每个变体配置，例如 ABI 过滤器。
 - **`proguardFiles：`**指定插件应使用的 ProGuard 配置文件。
 - **`signingConfig：`**签署此产品风格使用的配置。
 - **`testApplicationId：`**测试应用程序 ID。
 - **`testInstrumentationRunner：`**测试仪器运行器类名称。
 - **`testInstrumentationRunnerArguments：`**测试仪器运行器自定义参数。
 - **`vectorDrawables：`**用于配置 VectorDrawable 的构建时支持的选项。
 - **`versionCode：`**版本号。
 - **`versionName：`**版本名称。
 - **`versionNameSuffix：`**版本名称后缀。
 - **`wearAppUnbundled：`**返回是否为嵌入式磨损应用启用非捆绑模式。如果为 true，则可以使应用程序从嵌入式磨损应用程序转换为直接由 Play 商店分发的应用程序。

### splits ###
**`splits{}`** 闭包用于配置构建多个APK或APK分割。DSL 对象为 `Splits`。

`Splits` 具有以下属性：
 - **`abi：`**封装用于构建每个 ABI APK 的设置。
 - **`abiFilters：`**插件将为 ABI 列表生成单独的 APK。
 - **`density：`**封装用于构建每密度 APK 的设置。
 - **`densityFilters：`**该插件将为其生成单独 APK 的屏幕密度配置列表。
 - **`language：`**封装用于构建每个语言(或地区) APK 的设置。
 - **`languageFilters：`**插件将为其生成单独 APK 的语言(或地区)列表。

### testOptions ###
**`testOptions{}`** 闭包用于配置构建多个APK或APK分割。DSL 对象为 `TestOptions`。

`TestOptions` 具有以下属性：
 - **`animationsDisabled：`**在从命令行运行的检测测试期间禁用动画。
 - **`execution：`**指定是否使用设备上的测试编制。
 - **`reportDir：`**报表目录的名称。
 - **`resultsDir：`**结果目录的名称。
 - **`unitTests：`**配置单元测试选项。

### variantFilter ###
**`variantFilter{}`** 闭包用于配置 Android 插件应包含或从 Gradle 项目中删除的变体。DSL 对象为 `Action<VariantFilter>`。

`VariantFilter` 具有以下属性：
 - **`buildType：`**构建类型。
 - **`defaultConfig：`**表示默认配置的 ProductFlavor。
 - **`flavors：`**风味列表或空列表。

### jacoco (已弃用) ###
**`jacoco{}`** 闭包用于配置用于脱机检测和覆盖报告的 JaCoCo 版本。DSL 对象为 `JacocoOptions`。

`JacocoOptions` 具有以下属性：
 - **`version：`**要使用的 JaCoCo 版本。

## AppExtension ##
**`AppExtension`** 就是应用程序插件的 Android 扩展。`com.android.build.gradle.AppExtension` 类继承自 `com.android.build.gradle.BaseExtension`。

`AppExtension` 除了继承自 `com.android.build.gradle.AppExtension` 的属性外，还有自有的属性。

### applicationVariants ###
**`applicationVariants`** 用于返回应用程序项目包含的构建变体集合(`DomainObjectSet<ApplicationVariant>`)。

> 要处理这个集合中的元素，应该使用 `all` 迭代器，这是因为插件只有在项目评估之后才填充这个集合。与 `each` 迭代器不同，在插件创建它们时使用 `all` 进程的未来元素。

以下示例遍历所有 applicationVariants 元素，将构建变量注入清单文件中：
```gradle
android.applicationVariants.all { variant ->
    def mergedFlavor = variant.getMergedFlavor()
    // Defines the value of a build variable you can use in the manifest.
    mergedFlavor.manifestPlaceholders = [hostName:"www.example.com/${variant.versionName}"]
}
```

### testBuildType ###
**`testBuildType`** 指定插件用于测试模块的构建类型。

默认情况下，Android 插件使用 `debug` 构建类型。这意味着当您使用 `gradlew connectedAndroidTest` 部署检测测试时，它会使用模块 `debug` 构建类型中的代码和资源来创建测试 APK。然后，该插件将模块的 APK 和测试 APK 的 `debug` 版本部署到连接的设备，并运行测试。

要将测试构建类型更改为 `debug` 以外的其他类型，请按如下所示进行指定：
```gradle
android {
    // Changes the test build type for instrumented tests to "stage".
    testBuildType "stage"
}
```

### testVariants ###
**`testVariants`** 用于返回 Android 测试构建变体的集合(`DomainObjectSet<TestVariant>`)。

> 要处理这个集合中的元素，应该使用 `all` 迭代器，这是因为插件只有在项目评估之后才填充这个集合。与 `each` 迭代器不同，在插件创建它们时使用 `all` 进程的未来元素。

### unitTestVariants ###
**`unitTestVariants`** 用于返回 Android 单元测试构建变体的集合(`DomainObjectSet<UnitTestVariant>`)。

> 要处理这个集合中的元素，应该使用 `all` 迭代器，这是因为插件只有在项目评估之后才填充这个集合。与 `each` 迭代器不同，在插件创建它们时使用 `all` 进程的未来元素。

### flavorDimensions ###
**`flavorDimensions`** 用于配置项目的产品风味维度的名称。

使用 Android plugin 3.0.0 及更高版本配置产品 product flavors 时，必须使用 `flavorDimensions` 属性指定至少一个 flavor 维度，然后将每个 flavor 分配给维度。否则，您将收到以下构建错误：
```
Error:All flavors must now belong to a named flavor dimension.
The flavor 'flavor_name' is not assigned to a flavor dimension.
```

默认情况下，当仅指定一个维度时，配置的所有 flavor 都将自动属于该维度。如果指定多个维度，则需要手动将每个 flavor 分配给维度，如下面的示例所示：
```gradle
android {
    ...
    // Specifies the flavor dimensions you want to use. The order in which you
    // list each dimension determines its priority, from highest to lowest,
    // when Gradle merges variant sources and configurations. You must assign
    // each product flavor you configure to one of the flavor dimensions.
    flavorDimensions 'api', 'version'

    productFlavors {
      demo {
        // Assigns this product flavor to the 'version' flavor dimension.
        dimension 'version'
        ...
    }

      full {
        dimension 'version'
        ...
      }

      minApi24 {
        // Assigns this flavor to the 'api' dimension.
        dimension 'api'
        minSdkVersion '24'
        versionNameSuffix "-minApi24"
        ...
      }

      minApi21 {
        dimension "api"
        minSdkVersion '21'
        versionNameSuffix "-minApi21"
        ...
      }
   }
}
```

### useLibrary ###
**`useLibrary`** 用于将指定的库包含到类路径中。

通常使用此属性来支持随 Android SDK 一起提供的可选平台库。以下示例将 `Apache HTTP API` 库添加到项目类路径：
```gradle
android {
    // Adds a platform library that ships with the Android SDK.
    useLibrary 'org.apache.http.legacy'
}
```

