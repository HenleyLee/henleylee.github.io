---
title: Android Gradle 常用命令
categories: Android
tags:
  - Android
  - Gradle
abbrlink: f592fe8e
date: 2020-02-19 12:56:26
---

`Gradle` 是基于 `Groovy` 语言实现的一个编译系统， Google 针对 `Android` 编译用 Groovy 语言开发了一套 DSL。

## gradle wrapper ##
每个基于 `Gradle` 构建的工程都有一个 `gradle` 本地代理，叫做 `gradle wrapper`，在 `/gradle/wrapper/gralde-wrapper.properties` 目录中声明了指向目录和版本。

本地建立文件 `gradle.properties` 或者在用户的 `.gradle` 目录下建立 `gradle.properties` 文件作为全局设置，常用的有以下参数：
```gradle
# 开启守护进程
org.gradle.daemon=true
# 开启并行编译
org.gradle.parallel=true
# 按需编译
org.gradle.configureondemand=true
# 开启JNI编译支持过时API
android.useDeprecatedNdk=true
# 设置编译jvm参数
org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
# 设置Socks代理
systemProp.socks.proxyHost=127.0.0.1
systemProp.socks.proxyPort=1080
# 设置HTTP代理
systemProp.http.proxyHost=127.0.0.1
systemProp.http.proxyPort=10384
systemProp.https.proxyHost=127.0.0.1
systemProp.https.proxyPort=10384
```

## Gradle 常用命令 ##

## 任务命令 ##
```gradle
# 查看所有任务
./gradlew tasks --all

# 执行指定Module的指定任务
./gradlew :moduleName:taskName
```

## 快速构建命令 ##
```gradle
# 查看构建版本
./gradlew -v

# 清除build文件夹
./gradlew clean

# 检查依赖并编译打包
./gradlew build

# 编译并打印日志
./gradlew build --info

# 调试模式构建并打印日志
./gradlew build --info --debug --stacktrace

# 强制更新最新依赖，清除构建并构建 
./gradlew clean --refresh-dependencies build
```

## 指定构建目标命令 ##
```gradle
# 编译并打Debug包
./gradlew assembleDebug

# 编译并打Release包
./gradlew assembleRelease
```

## 构建并安装调试命令 ##
```gradle
# 编译app module 并打Debug包
./gradlew install app:assembleDebug

#  Release模式打包并安装
./gradlew installRelease

# 卸载Release模式包
./gradlew uninstallRelease
```

## 查看包依赖 ##
```gradle
# 查看包依赖
./gradlew dependencies --info

# 编译时的依赖库
./gradlew app:dependencies --configuration compile

# 运行时的依赖库
./gradlew app:dependencies --configuration runtime
```

## 离线编译模式 ##
```gradle
# 离线编译模式
./gradlew aDR --offline
```

## 守护进程编译模式 ##
```gradle
# 守护进程编译模式
./gradlew dependencies --info
```

## 并行编译模式 ##
```gradle
# 并行编译模式
./gradle build --parallel --parallel-threads=N
```

## 按需编译模式 ##
```gradle
# 按需编译模式
./gradle build --configure-on-demand
```

## 多渠道打包 ##
```gradle
# 编译并打Debug包
./gradlew assemble[productFlavorsName]Debug

# 编译并打Release包
./gradlew assemble[productFlavorsName]Release

# Release模式打包并安装
./gradlew install[productFlavorsName]Release

# 卸载Release模式包
./gradlew uninstall[productFlavorsName]Release
```

