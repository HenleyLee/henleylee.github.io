---
title: Android Studio 让开发效率事半功倍的插件整理
categories: Android
tags:
  - Android
  - Android Studio
  - IntelliJ IDEA
  - Plugins
abbrlink: d09e4c2c
date: 2018-08-19 14:26:35
---

Google 在2013年5月的 I/O 开发者大会推出了基于 IntelliJ IDEA Java IDE 上的 Android Studio。Android Studio 是一个功能齐全的开发工具，还提供了对第三方插件的支持，让开发人员更快速更好的开发应用程序。

## 插件安装 ##
首先通过导航栏 **File | Settings** 或是直接 **Ctrl+Alt+S** 打开设置对话框，选择 **Plugins**。找到需要的插件后点击右侧的 **Install** 按钮进行下载安装，安装完成后 **Install** 按钮会变成 **Restart Android Studio**，点击 **Restart** 重启 Android Studio 即可。

Android Studio 支持三种安装插件的方法：
1. **Install JetBrains Plugins：** 安装 JetBrains 的官方插件。官方插件种类非常丰富，包含辅助用户体验、语言支持等，稳定可靠，勤于更新。
2. **Browse repositories：** 浏览官方仓库。官方仓库不仅有官方插件，还有来自社区的插件的第三方插件。
3. **Install plugin from disk：** 从本地安装插件。IDEA 的插件打包之后，用此功能就可以安装，方便了小团体之间的插件开发和共享。

## 插件汇总 ##
### .ignore ###
[Download](https://plugins.jetbrains.com/plugin/7495) | [GitHub](https://github.com/hsz/idea-gitignore)

.ignore 是一个快速生成 `.gitignore` (Git)、`.hgignore` (Mercurial)、`.npmignore` (NPM)、`.dockerignore` (Docker)、`.chefignore` (Chef)、`.cvsignore` (CVS)、`.bzrignore` (Bazaar)、`.boringignore` (Darcs)、`.mtn-ignore` (Monotone)、`ignore-glob` (Fossil)、`.jshintignore` (JSHint)、`.tfignore` (Team Foundation)、`.p4ignore` (Perforce)、`.flooignore` (Floobits)、`.eslintignore` (ESLint)、`.cfignore` (Cloud Foundry)、`.jpmignore` (Jetpack)、`.stylelintignore` (StyleLint)、`.stylintignore` (Stylint)、`.swagger-codegen-ignore` (Swagger Codegen)、`.helmignore` (Kubernetes Helm)、`.upignore` (Up)、`.prettierignore` (Prettier)、`.ebignore` (ElasticBeanstalk) 文件的插件。 它支持的 JetBrains IDE 有 `Android Studio`、`AppCode`、`CLion、IntelliJ IDEA`、`PhpStorm`、`PyCharm`、`RubyMine`、`WebStorm、DataGrip`。
![](https://user-gold-cdn.xitu.io/2018/10/10/1665cd65b27f795c?w=1200&h=887&f=png&s=468405)
![](https://user-gold-cdn.xitu.io/2018/10/10/1665cd681bbde5c0?w=800&h=594&f=png&s=180564)

### 360 FireLine ###
[Download](https://plugins.jetbrains.com/plugin/9292) | [WebSite](http://magic.360.cn/)

360 FireLine 是一款是免费的适用于 Android 和 Java 代码的代码静态分析插件。主打的安全检查规则是根据360业务多年技术沉淀而来。内存类检查的精确度业内领先。
![](https://user-gold-cdn.xitu.io/2018/10/12/16666499c50c29e6?w=1038&h=701&f=png&s=38265)
![](https://user-gold-cdn.xitu.io/2018/10/12/1666649b61e69e9d?w=457&h=764&f=png&s=54499)
![](https://user-gold-cdn.xitu.io/2018/10/12/1666649cf43676ca?w=881&h=389&f=jpeg&s=32588)

### ADB Idea ###
[Download](https://plugins.jetbrains.com/plugin/7380) | [GitHub](https://github.com/pbreault/adb-idea)

ADB Idea 是一款 ADB 调试工具，支持 Uninstall App、Kill App、Start App、Restart App、Clear App Data、Clear App Data and Restart 等操作的插件。
![](https://user-gold-cdn.xitu.io/2018/10/10/1665cdf93e7be9dd?w=651&h=410&f=png&s=44841)
![](https://user-gold-cdn.xitu.io/2018/10/10/1665cdf47adb5048?w=316&h=300&f=png&s=18753)

### ADB WIFI ###
[Download](https://plugins.jetbrains.com/plugin/7856) | [GitHub](https://github.com/layerlre/ADBWIFI)

ADB WIFI 是一款无需 root 就可以通过 WiFi 调试 Android APP 的 Android Studio 插件。
![](https://user-gold-cdn.xitu.io/2018/10/10/1665ce32d2ca1668?w=598&h=92&f=png&s=10422)

### Alibaba Java Coding Guidelines ###
[Download](https://plugins.jetbrains.com/plugin/10046) | [GitHub](https://github.com/alibaba/p3c)

Alibaba Java Coding Guidelines 是一款 Java 代码规约扫描插件。
![](https://user-gold-cdn.xitu.io/2018/10/10/1665ceeb46a11f8c?w=469&h=1133&f=png&s=75360)
![](https://user-gold-cdn.xitu.io/2018/10/10/1665ceed24d82cd1?w=1082&h=342&f=png&s=52811)

### Android ButterKnife Zelezny ###
[Download](https://plugins.jetbrains.com/plugin/7369) | [GitHub](https://github.com/avast/android-butterknife-zelezny)

Android ButterKnife Zelezny 是一款用于根据 activities/fragments/adapters 选中的 xml 布局生成 ButterKnife 注入的插件。选中 activities/fragments/adapters 中引用的 xml 布局，点击 **Generate** 菜单或使用快捷键 **Alt + Insert**，然后选择 **Generate ButterKnife Injections** 即可。
![](https://user-gold-cdn.xitu.io/2018/10/10/1665cf3ca52dd542?w=712&h=591&f=gif&s=224045)

### Android Code Generator ###
[Download](https://plugins.jetbrains.com/plugin/7595) | [GitHub](https://github.com/tmorcinek/android-codegenerator-plugin-intellij)

Android Code Generator 是一款根据布局文件快速生成对应的Activity、Fragment、Adapter、Menu 的插件。
![](https://user-gold-cdn.xitu.io/2018/10/10/1665d579274807b3?w=1152&h=720&f=gif&s=1314525)
![](https://user-gold-cdn.xitu.io/2018/10/10/1665d58067ae589a?w=1152&h=720&f=gif&s=1036291)

### Android Methods Count ###
[Download](https://plugins.jetbrains.com/plugin/8076)

Android Methods Count 是一款统计 Android 依赖库中方法的总个数的插件。
![](https://user-gold-cdn.xitu.io/2018/10/10/1665d0566e9e6342?w=650&h=222&f=gif&s=196419)
![](https://user-gold-cdn.xitu.io/2018/10/10/1665d05853f75575?w=530&h=81&f=png&s=19180)

### Android Parcelable code generator ###
[Download](https://plugins.jetbrains.com/plugin/7332) | [GitHub](https://github.com/mcharmas/android-parcelable-intellij-plugin/)

Android Parcelable code generator 是一款基于数据类中的字段快速实现 Parcelable 接口的插件。在编辑器中点击 **Generate** 菜单或使用快捷键 **Alt + Insert**，然后选择 **Parcelable** 即可。
![](https://user-gold-cdn.xitu.io/2018/10/10/1665d0eecd18e5e3?w=600&h=601&f=png&s=85375)

### AndroidSourceViewer ###
[Download](https://plugins.jetbrains.com/plugin/10187) | [GitHub](https://github.com/pengwei1024/AndroidSourceViewer)

AndroidSourceViewer 是一款在 Android Studio 中在线查看 Android 和 Java 指定版本源码插件。
![](https://user-gold-cdn.xitu.io/2018/10/12/16667115154a6478?w=310&h=254&f=png&s=27812)
![](https://user-gold-cdn.xitu.io/2018/10/12/1666711857bb2332?w=636&h=512&f=png&s=134796)

### CheckStyle-IDEA ###
[Download](https://plugins.jetbrains.com/plugin/1065) | [GitHub](https://github.com/jshiell/checkstyle-idea)

CheckStyle-IDEA 是一款帮助程序员编写符合编码标准的 Java 代码的插件。它可以自动执行检查 Java 代码的过程，从而使人类免于这项无聊但重要的任务，这使其成为希望实施编码标准的项目的理想选择。Checkstyle 具有高度可配置性，可以支持几乎任何编码标准。Checkstyle 提供了一个示例配置文件，支持 Sun Code Conventions 和 Google Java Style。
![](https://user-gold-cdn.xitu.io/2018/10/12/16666ffdb6617ec3?w=1163&h=935&f=png&s=79634)

### CodeGlance ###
[Download](https://plugins.jetbrains.com/plugin/7275) | [GitHub](https://github.com/Vektah/CodeGlance)

CodeGlance 是一款显示类似于 Sublime 中的代码小地图用于快速定位代码的插件。
![](https://user-gold-cdn.xitu.io/2018/10/10/1665d6137c651035?w=1282&h=697&f=png&s=651937)

### EventBus3 Intellij Plugin ###
[Download](https://plugins.jetbrains.com/plugin/8603) | [GitHub](https://github.com/likfe/eventbus3-intellij-plugin)

EventBus3 Intellij Plugin 是一款为 EventBus 提供快速索引和跳转的插件。
![](https://user-gold-cdn.xitu.io/2018/10/12/16666e36dbcea911?w=600&h=400&f=gif&s=432363)

### FindBugs-IDEA ###
[Download](https://plugins.jetbrains.com/plugin/3847) | [GitHub](https://github.com/andrepdo/findbugs-idea)

FindBugs-IDEA 是一款通过提供静态字节码分析以从 IntelliJ IDEA 中查找 Java 代码中的 bug 的插件。FindBugs 是一个 Java 缺陷检测工具，它使用静态分析来查找超过200个错误模式，比如空指针取消引用、无限的递归循环、Java 库的糟糕使用和死锁。FindBugs 可以在大型应用程序中识别数百个严重缺陷(通常每1000-2000行非注释源语句中约有1个缺陷)。
![](https://user-gold-cdn.xitu.io/2018/10/12/16666fa3883acb3e?w=1140&h=534&f=png&s=98882)
![](https://user-gold-cdn.xitu.io/2018/10/12/16666fa739970eae?w=1736&h=1144&f=png&s=340743)

### GsonFormat ###
[Download](https://plugins.jetbrains.com/plugin/7654) | [GitHub](https://github.com/zzz40500/GsonFormat)

GsonFormat 是一款快速格式化 json 数据并自动生成实体类参数的插件。新建实体类并在编辑器中点击 **Generate** 菜单或使用快捷键 **Alt + Insert**，然后选择 **GsonFormat** 即可。
![](https://user-gold-cdn.xitu.io/2018/10/10/1665d1cf88706283?w=819&h=652&f=gif&s=388525)

### Lifecycle Sorter ###
[Download](https://plugins.jetbrains.com/plugin/7742) | [GitHub](https://github.com/armandAkop/Lifecycle-Sorter)

Lifecycle Sorter 是一款可以对 Activity 或 Fragment 的生命周期方法按照它们在应用程序中的调用顺序进行排序的插件。
![](https://user-gold-cdn.xitu.io/2018/10/10/1665d28b507a852d?w=2080&h=1613&f=png&s=555296)

### Markdown Navigator ###
[Download](https://plugins.jetbrains.com/plugin/7896) | [GitHub](https://github.com/vsch/idea-multimarkdown)

Markdown Navigator 是一款带有 GFM 和匹配的预览样式的的 Markdown 插件。
![](https://user-gold-cdn.xitu.io/2018/10/10/1665d3006d96b10b?w=1102&h=596&f=gif&s=2159593)

### MVPHelper ###
[Download](https://plugins.jetbrains.com/plugin/8507) | [GitHub](https://github.com/githubwing/MVPHelper)

MVPHelper 是一款 Intellj IDEA 和 Android Studio 自动生成 MVP 模式所需接口以及实现类的插件。在 Contract 类或者 Presenter 类内部，点击 **Generate** 菜单或使用快捷键 **Alt + Insert**, 然后选择 **Mvp Helper** 即可生成对应文件.
![](https://user-gold-cdn.xitu.io/2018/10/10/1665d351ee1a16df?w=958&h=518&f=gif&s=286497)

### Remove ButterKnife ###
[Download](https://plugins.jetbrains.com/plugin/8432) | [GitHub](https://github.com/u3shadow/RemoveButterKnife)

Remove ButterKnife 是一款用于移除代码中对 ButterKnife 使用的插件。
![](https://user-gold-cdn.xitu.io/2018/10/10/1665d384560cf98f?w=1452&h=674&f=gif&s=458990)

