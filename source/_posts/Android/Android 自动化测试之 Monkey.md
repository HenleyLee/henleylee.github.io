---
title: Android 自动化测试之 Monkey
date: 2018-07-08 12:23:45
categories: Android
tags:
  - Android
  - Monkey
---

## Monkey 简介 ##
### Monkey 介绍 ###
Monkey 是一个命令行工具，可以运行在模拟器里或实际设备中。它向系统发送伪随机的用户事件流，实现对正在开发的应用程序进行压力测试。

Monkey 测试也有人叫做搞怪测试。就是用一些稀奇古怪的操作方式去测试被测试系统，以测试系统的稳定性。

Monkey 测试是 Android 自动化测试的一种手段，Monkey 测试本身非常简单，就是模拟用户的按键输入，触摸屏输入，手势输入等，看设备多长时间会出异常。

当 Monkey 程序在模拟器或设备运行的时候，如果用户触发了比如点击，触摸，手势或一些系统级别的事件的时候，它就会产生随机脉冲，所以可以用 Monkey 用随机重复的方法去负荷测试你开发的软件。

Monkey 包括许多选项，它们大致分为四大类：
 - 基本配置 选项，如设置尝试的事件数量。
 - 运行约束选项，如设置只对单独的一个包进行测试。
 - 事件类型和频率。
 - 调试选项。

在 Monkey 运行的时候，它生成事件，并把它们发给系统。同时，Monkey 还对测试中的系统进行监测，对下列三种情况进行特殊处理：
 - 如果限定了 Monkey 运行在一个或几个特定的包上，那么它会监测试图转到其它包的操作，并对其进行阻止。
 - 如果应用程序崩溃或接收到任何失控异常，Monkey 将停止并报错。
 - 如果应用程序产生了应用程序不响应(application not responding)的错误，Monkey 将会停止并报错。

按照选定的不同级别的反馈信息，在 Monkey 中还可以看到其执行过程报告和生成的事件。谈到 Monkey，必须介绍一下 ADB。

ADB 是 Android SDK 里的一个工具，用这个工具可以直接操作管理 Android 模拟器或者真实的 Android 设备。它的主要功能有：
 - 运行设备的shell(命令行)
 - 管理模拟器或设备的端口映射
 - 计算机和设备之间上传/下载文件
 - 将本地 APK 软件安装至模拟器或 Android 设备
 - ADB 是一个客户端-服务器端程序，其中客户端是你用来操作的电脑，服务器端是 Android 设备。

### ADB 的常用命令 ###
| 命令                                            | 作用                            |
| ----------------------------------------------- | ------------------------------- |
| `adb devices`                                   | 获取所有连接ADB的模拟器或者真机 |
| `adb install c:/xxx.apk`                        | 安装自己的apk到设备上           |
| `adb uninstall <package-name>`                  | 从设备上卸载apk                 |
| `adb shell pm list packages`                    | 获取所有应用的包名              |
| `adb -s emulator-5556 uninstall <package-name>` | 指定某设备卸载apk               |
| `adb start-server`                              | 重启adb                         |
| `adb kill-server`                               | 杀死adb                         |

### Monkey 的常用命令 ###
| 命令                                         | 作用                                    |
| -------------------------------------------- | --------------------------------------- |
| `adb shell monkey –help`                     | 获取帮助命令                            |
| `adb shell monkey <count>`                   | 随机执行 `count` 个模拟事件             |
| `adb shell monkey -p <package-name> <count>` | 指定某个应用随机执行 `count` 个模拟事件 |
| `adb shell monkey [options] <count>`         | 带参数执行 `count` 个模拟事件           |

## Monkey 参数介绍 ##
### 基本参数 ###
| 命令         | 作用                           | 注意                           |
| ------------ | ------------------------------ |                           |
| `-v`         | 日志详细程度                   |每一个-v将增加反馈信息的级别。Level 0(缺省值)除启动提示、测试完成和最终结果之外，提供较少信息。Level 1提供较为详细的测试信息，如逐个发送到Activity的事件。Level 2提供更加详细的设置信息，如测试中被选中的或未被选中的Activity。 |
| `-s`         | 伪随机数生成器的seed值         |如果用相同的seed值再次运行Monkey， 两次monkey测试将生成相同的事件序列。  |
| `--throttle` | 两次事件的时间间隔，单位是毫秒 |通过这个选项可以减缓Monkey的执行速度。如果不指定该选项，Monkey将 不会被延迟，事件将尽可能快地被产成。  |

### 发送事件的类型 ###
| 命令                        | 作用                                                                                                                      |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `--pct-touch <percent>`     | 指定触摸事件百分比，一个点上先后有按下和抬起的操作。                                                                      |
| `--pct-motion <percent>`    | 指定滑动事件百分比，先按下，滑动一段距离，然后抬起。                                                                      |
| `--pct-trackball <percent>` | 轨迹球事件百分比，一系列的随机移动和单击操作。                                                                            |
| `--pct-nav <percent>`       | 基本导航事件百分比(硬件)，设置基本的导航事件(上/下/左/右导航键)的生成比例。                                               |
| `--pct-majornav <percent>`  | 主要导航事件百分比，会导致UI产生回馈的事件，如单击5个方向键中的中间按钮，单击后退键或者菜单键。                           |
| `--pct-syskeys <percent>`   | 系统按键事件百分比(Home、Back、startCall、endCall、volumeControl)。                                                       |
| `--pct-appswitch <percent>` | 指定启动Activity的百分比。在随机间隔里，Monkey将执行一个startActivity()调用，作为最大程度覆盖包中全部Activity的一种方法。 |
| `--pct-anyevent <percent>`  | 指定其他事件百分比,普通的按键消息，设备上一些不常用的按钮事件。                                                           |

### 约束条件 ###
| 命令                        | 作用                           | 注意                           |
| --------------------------- | ------------------------------ |                           |
| `-p <allowed-package-name>` | 如果用此参数指定了一个或几个包，Monkey将只允许系统启动这些包里的Activity。 如果你的应用程序还需要访问其它包里的Activity(如选择取一个联系人)，那些包也需要在此 同时指定。如果不指定任何包，Monkey将允许系统启动全部包里的Activity。 | 要指定多个包，需要使用多个-p选项，每个-p选项只能用于一个包。 |
| `-c <main-category>`        | 如果用此参数指定了一个或几个类别，Monkey将只允许系统启动被这些类别中的某个类别列出的Activity。 如果不指定任何类别，Monkey将选 择下列类别中列出的Activity： Intent.CATEGORY_LAUNCHER或Intent.CATEGORY_MONKEY。 | 要指定多个类别，需要使用多个-c选项，每个-c选项只能用于一个类别。  |

### 调试选项 ###
| 命令                        | 作用                                                                                                                       |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `--dbg-no-events`     | 指定了此选项，monkey会启动待测应用，但不发送任何消息，建议与-v,-p,-throttle一起使用。                                            |
| `--hprof`    | 指定此选项，monkey会在发送事件前后生成性能报告(即内存的快照文件)，一般在设备的/data/misc目录下生成一个5M左右的文件。                      |
| `--ignore-crashes` | 指定了此选项，待测应用崩溃或发生异常时，继续发送系统消息，直到指定个数的消息全部发送完毕，否则停止运行。                            |
| `--ignore-timeouts`       | 指定了此选项，待测应用停止响应(如弹出“应用无响应”对话框)时，继续发送系统消息，直到指定个数的消息全部发送完毕，否则停止运行。 |
| `--ignore-security-exceptions`  | 指定了此选项，待测应用碰到权限方面的错误时，继续发送系统消息，直到指定个数的消息全部发送完毕，否则停止运行。           |
| `--kill-process-after-error`   | 一般情况下，当monkey因为某个错误指定运行时，出问题的应用会留在系统上继续执行，这个选项通知系统当错误发生时杀掉进程。    |
| `--monitor-native-crashes` | 监视由Android C/C++代码部分(cpu计算部分)引起的崩溃，此时如果设置了“--kill-process-after-error”，整个系统会关机。            |
| `--wait-dbg`  | 停止执行中的Monkey，直到有调试器和它相连接。                                                                                             |

## Monkey 测试命令 ##
### 不输出日志 ###
```shell
adb shell monkey -v --throttle 300 --pct-touch 30 --pct-motion 20 --pct-nav 20 --pct-majornav 15 --pct-appswitch 5 --pct-anyevent 5 --pct-trackball 0 --pct-syskeys 0 -p <package-name> 1000
```

### 输出日志 ###
```shell
adb shell monkey -v --throttle 300 --pct-touch 30 --pct-motion 20 --pct-nav 20 --pct-majornav 15 --pct-appswitch 5 --pct-anyevent 5 --pct-trackball 0 --pct-syskeys 0 -p <package-name> 1000>D:\Monkey\log.txt
```

