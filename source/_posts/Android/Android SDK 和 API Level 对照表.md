---
title: Android SDK 和 API Level 对照表
categories: Android
tags:
  - Android
abbrlink: c4a7cb6a
date: 2019-12-30 12:36:32
---

Android 开发版本按照字母代号划分为不同的系列，这些代号的灵感源自美味的点心。

代号、版本号、API 级别的对应关系如下：

| 代号               | 版本        | API level |
| :----------------: | :---------: | :-------: |
| Android 10         | 10          | API 29    |
| Pie	             | 9	   | API 28    |
| Oreo	             | 8.1.0	   | API 27    |
| Oreo	             | 8.0.0	   | API 26    |
| Nougat             | 7.1	   | API 25    |
| Nougat             | 7.0	   | API 24    |
| Marshmallow        | 6.0	   | API 23    |
| Lollipop	     | 5.1	   | API 22    |
| Lollipop	     | 5.0	   | API 21    |
| KitKat	     | 4.4-4.4.4   | API 19    |
| Jelly Bean	     | 4.3.x	   | API 18    |
| Jelly Bean	     | 4.2.x	   | API 17    |
| Jelly Bean	     | 4.1.x	   | API 16    |
| Ice Cream Sandwich | 4.0.3-4.0.4 | API 15    |
| Ice Cream Sandwich | 4.0.1-4.0.2 | API 14    |
| Honeycomb	     | 3.2.x	   | API 13    |
| Honeycomb	     | 3.1	   | API 12    |
| Honeycomb	     | 3.0	   | API 11    |
| Gingerbread	     | 2.3.3-2.3.7 | API 10    |
| Gingerbread	     | 2.3-2.3.2   | API 9     |
| Froyo		     | 2.2.x       | API 8     |
| Eclair             | 2.1         | API 7     |
| Eclair             | 2.0.1	   | API 6     |
| Eclair             | 2.0	   | API 5     |
| Donut		     | 1.6         | API 4     |
| Cupcake	     | 1.5	   | API 3     |
| (NONE)	     | 1.1	   | API 2     |
| (NONE)	     | 1.0	   | API 1     |


如果需要查看当前 Android 设备的 SDK 版本，可以通过一下代码：
```java
android.os.Build.VERSION.SDK_INT
```

具体数值对应的 Android SDK 版本可以查看 [Build.VERSION_CODES](https://developer.android.com/reference/android/os/Build.VERSION_CODES)


本文摘自：[Codenames, Tags, and Build Numbers](https://source.android.com/setup/start/build-numbers)

