---
title: Gradle 缓存目录结构说明
categories: Gradle
tags:
  - Gradle
abbrlink: d1b3c0f5
date: 2020-02-07 18:32:50
---

## Gradle 缓存策略 ##
`Gradle` 依赖的版本分为`正式版本`、`快照版本`、`动态版本`：
 - `正式版本`：有明确指明版本号，比如 `implementation 'androidx.appcompat:appcompat:1.1.0'`。
 - `快照版本`：版本号后面有`-SNAPSHOT`，比如 `implementation 'androidx.appcompat:1.1.0-SNAPSHOT'`。
 - `动态版本`：没有具体指明版本号，比如 `implementation 'androidx.appcompat:appcompat:1.+'` 或 `implementation 'androidx.appcompat:appcompat:latest.release'` 或 `implementation 'androidx.appcompat:appcompat:latest.integration'`。

`Gradle` 的缓存策略中，对于 `SNAPSHOT` 版本默认的缓存周期是 `24` 小时，也就是从上次更新之后，24 小时内都会使用上次的缓存。

`Gradle` 对于`动态版本`和`快照版本`的缓存时间默认是 `24` 小时，也就是从上次更新之后，24 小时内都会使用上次的缓存。

> 配置动态版本，会导致额外的构建查询时间，具体可以输出构建性能报告，自行比较。

## Gradle 缓存周期 ##
当远程仓库上传了相同版本依赖时，有时需要为缓存指定一个时效去检查远程仓库的依赖笨版本，`Gradle` 提供了 `cacheChangingModulesFor(int, java.util.concurrent.TimeUnit)` 和 `cacheDynamicVersionsFor(int, java.util.concurrent.TimeUnit)` 两个方法来设置缓存的时效：
```gradle
configurations.all {
    // 每隔24小时检查远程依赖是否存在更新
    resolutionStrategy.cacheChangingModulesFor 24, 'hours'
    // 每隔60分钟检查远程依赖是否存在更新
    // resolutionStrategy.cacheChangingModulesFor 60, 'minutes'
    // 每次构建都检查远程依赖是否存在更新
    // resolutionStrategy.cacheChangingModulesFor 0, 'seconds'
    // 采用动态版本声明的依赖缓存每隔10分钟检查是否存在更新
    resolutionStrategy.cacheDynamicVersionsFor 10 * 60, 'seconds'
}
```

## Gradle 缓存目录 ##

### .gradle 目录 ###

| 目录          | 描述                     |
| ------------- | ------------------------ |
| caches	| gradle 缓存目录          |
| daemon	| daemon 日志目录          |
| native	| gradle 平台相关目录      |
| wrapper	| gradle-wrapper 下载目录  |

### caches 目录 ###
`caches` 目录用于存放 Gradle 构建缓存和依赖缓存。

| 目录          | 描述                     |
| ------------- | ------------------------ |
| 2.14.1	| gradle程序脚本           |
| 3.2.1	        | gradle程序脚本           |
| jars-2        | 未知                     |
| jars-3        | 未知                     |
| modules-2     | 下载缓存目录             |

#### caches/modules-2 目录 ####

| 目录          | 描述                     |
| ------------- | ------------------------ |
| files-2.1	| gradle 下载的jar目录     |
| metadata-2.16	| gradle-2.14.1 的描述文件 |
| metadata-2.23	| gradle-3.2.1 的描述文件  |

### daemon 目录 ###
`daemon` 目录用于存放 Gradle 守护进程的运行日志，按 Gradle 程序版本存放。

| 目录          | 描述                     |
| ------------- | ------------------------ |
| 2.14.1	| gradle-2.14.1 运行的日志 |
| 3.2.1	        | gradle-3.2.1运行的日志 |

### wrapper 目录 ###
`wrapper ` 目录用于存放 gradle-wrapper  下载 Gradle 的压缩包包和解压后的文件夹。

wrapper 的目录规则
```
wrapper/dists/gradle-2.14.1-all/base36/gradle-2.14.1-all.zip
wrapper/dists/gradle-2.14.1-all/base36/gradle-2.14.1-all.zip.lck
wrapper/dists/gradle-2.14.1-all/base36/gradle-2.14.1-all.zip.ok
```

