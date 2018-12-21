---
title: Android Studio 3.0 利用 Android Profiler 测量应用性能
categories: Android
tags:
  - Android
  - Android Studio
  - Android Profiler
abbrlink: 26cb225c
date: 2018-08-18 11:25:35
---

Android Studio 3.0 采用全新的 `Android Profiler` 窗口取代 `Android Monitor` 工具。 这些全新的分析工具能够提供关于应用 `CPU`、`内存`和`网络活动`的实时数据。 

您可以执行基于样本的函数跟踪来记录代码执行时间、采集堆转储数据、查看内存分配，以及查看网络传输文件的详情。

## Android Profiler 的使用方法 ##
### Android Profiler 的打开步骤 ###
要打开 **Android Profiler** 窗口，请按以下步骤操作：
1. 点击 **View > Tool Windows > Android Profiler**（也可以点击工具栏中的 **Android Profiler** <img src="https://user-gold-cdn.xitu.io/2018/8/6/1650e53ac183fbbb?w=24&h=24&f=png&s=1193" style="vertical-align:middle;"/>）。
2. 从 Android Profiler 工具栏中选择您想要分析的设备和应用进程。 如果您通过 USB 连接了某个设备但该设备未在设备列表中列出，请确保您已[启用 USB 调试](https://developer.android.google.cn/studio/debug/dev-options.html#enable)（如果您使用的是 Android Emulator 或已取得 root 权限的设备，Android Profiler 将列出所有正在运行的进程，即使这些进程可能无法调试。 当您发布可调试应用时，将会默认选择此进程）。
3. 在 **Android Profiler** 窗口顶部（如下图所示），选择您想要分析的设备和应用进程。

Android Profiler 共享时间线的视图显示如下图所示：
![](https://user-gold-cdn.xitu.io/2018/8/6/1650e54e3a40d6da?w=1228&h=960&f=png&s=519840)
Android Profiler 目前可显示共享时间线视图，可以在按钮①的位置选择设备，通过按钮②的位置选择想要的app进程，工具最底部显示了一个时间轴，其中包含了CPU、内存和网络使用的实时图。该窗口还包括时间轴缩放控制按钮③，一个跳转到实时更新的按钮④，以及显示活动状态、用户输入事件和屏幕旋转事件⑤的事件时间轴。

当您启动 Android Profiler 后，它会持续收集分析数据，直至您断开设备连接或点击 **Close** <img src="https://user-gold-cdn.xitu.io/2018/8/6/1650e53d003e386b?w=24&h=24&f=png&s=986" style="vertical-align:middle;"/>。

此共享时间线视图只显示时间线图表。 要使用详细分析工具，请点击与您想查看的性能数据对应的图表。 例如，要使用工具查看堆数据和跟踪内存分配，可点击 **MEMORY** 图表。

但并不是所有分析数据均默认可见。 如果您看到一条消息，显示“Advanced profiling is unavailable for the selected process”，则需在运行配置中[启用高级分析](https://developer.android.google.cn/studio/profile/android-profiler#advanced-profiling)。

### 启用高级分析 ###
要显示高级分析数据，Android Studio 必须在您编译后的应用中插入监控逻辑。高级分析工具提供的功能包括：
 - Event 时间线（所有分析器窗口中均有）
 - 分配对象数量（Memory Profiler 中）
 - 垃圾回收 Event（Memory Profiler 中）
 - 有关所有传输的文件的详情（Network Profiler 中）

要启用高级分析，请按以下步骤操作：
1. 选择 **Run > Edit Configurations**。
2. 在左侧窗格中选择您的应用模块。
3. 点击 **Profiling** 标签，然后勾选 **Enable advanced profiling**。

现在重新构建并运行您的应用，即可获取完整的分析功能。但请注意，高级分析会减缓您的构建速度，所以仅当您想要开始分析应用时才启用此功能。

>注：对于原生代码，不可使用高级分析功能。 如果您的应用是纯原生应用（不含 Java `Activity` 类），则不可使用高级分析功能。 如果您的应用使用了 JNI，则可使用部分高级分析功能，例如 Event 时间线、GC Event、Java 分配对象和基于 Java 的网络活动，但不能检测基于原生的分配和网络活动。

## 使用 CPU Profiler 检查 CPU 活动 和函数跟踪 ##
使用 CPU Profiler 检查 CPU 活动 和函数跟踪 CPU Profiler 可帮助您实时检查应用的 CPU 使用率和线程活动，并记录函数跟踪，以便您可以优化和调试您的应用代码。点击 **Android Profiler** 窗口中的 **CPU** 时间线中的任意位置即可打开 **CPU Profiler**。

### 为什么要分析 CPU 使用率 ###
最大限度减少应用的 CPU 使用率具有许多优势，如提供更快更顺畅的用户体验，以及延长设备电池续航时间。 它还可帮助应用在各种新旧设备上保持良好性能。 与应用交互时，您可以使用 CPU Profiler 监控 CPU 使用率和线程活动。 不过，如需了解应用如何执行其代码的详细信息，您应[记录和检查函数跟踪](https://developer.android.google.cn/studio/profile/cpu-profiler#method_traces)。

对于应用进程中的每个线程，您可以查看一段时间内执行了哪些函数，以及在其执行期间每个函数消耗的 CPU 资源。 您还可以使用函数跟踪来识别调用方和被调用方。 调用方指调用其他函数的函数，而被调用方是指被其他函数调用的函数。 您可以使用此信息确定哪些函数负责调用常常会消耗大量特定资源的任务，并尝试优化应用代码以避免不必要的工作。

如果您想收集可帮助您检查原生系统进程的详细系统级数据，并解决掉帧引起的界面卡顿，您应使用 [systrace](https://developer.android.google.cn/studio/profile/systrace-commandline.html)。

或者，如果您想导出您使用 [Debug](https://developer.android.google.cn/reference/android/os/Debug.html) 类捕获的 `.trace` 文件，您应使用 [Traceview](https://developer.android.google.cn/studio/profile/traceview.html)。

### CPU Profiler 概览 ###
当您打开 CPU Profiler 时，它将立即开始显示应用的 CPU 使用率和线程活动。CPU Profiler 的默认视图如下图所示：
![CPU Profiler](https://user-gold-cdn.xitu.io/2018/8/7/165135123618ed9d?w=900&h=546&f=png&s=174876)

如图所示，CPU Profiler 的默认视图包括以下内容：
1. **Event 时间线：** 显示应用中在其生命周期不同状态间转换的活动，并表明用户与设备的交互，包括屏幕旋转 Event。 如需了解有关 Event 时间线的更多信息，包括如何启用它，请阅读 [启用高级分析](https://developer.android.google.cn/studio/profile/android-profiler.html#advanced-profiling)。
2. **CPU 时间线：** 显示应用的实时 CPU 使用率（以占总可用 CPU 时间的百分比表示）以及应用使用的总线程数。 此时间线还显示其他进程的 CPU 使用率（如系统进程或其他应用），以便您可以将其与您的应用使用率进行对比。 通过沿时间线的水平轴移动鼠标，您还可以检查历史 CPU 使用率数据。
3. **线程活动时间线：** 列出属于应用进程的每个线程并使用下面列出的颜色沿时间线标示它们的活动。 在您记录一个函数跟踪后，您可以从此时间线中选择一个线程以在跟踪窗格中检查其数据。
> - **绿色：** 表示线程处于活动状态或准备使用 CPU。 即，它正在“运行中”或处于“可运行”状态。
> - **黄色：** 表示线程处于活动状态，但它正在等待一个 I/O 操作（如磁盘或网络 I/O），然后才能完成它的工作。
> - **灰色：** 表示线程正在休眠且没有消耗任何 CPU 时间。当线程需要访问尚不可用的资源时偶尔会发生这种情况。线程进入自主休眠或内核将此线程置于休眠状态，直到所需的资源可用。

4. **记录配置：** 允许您选择以下选项之一以确定分析器记录函数跟踪的方式。
> - **Sampled：** 一个默认配置，在应用执行期间频繁捕获应用的调用堆栈。 分析器比较捕获的数据集以推导与应用代码执行有关的时间和资源使用信息。 基于“Sampled”的跟踪的固有问题是，如果应用在捕获调用堆栈后进入一个函数并在下一次捕获前退出该函数，则分析器不会记录该函数调用。 如果您对此类生命周期很短的跟踪函数感兴趣，您应使用“Instrumented”跟踪。
> - **Instrumented：** 一个默认配置，在运行时设置应用以在每个函数调用的开始和结束时记录时间戳。 它收集时间戳并进行比较，以生成函数跟踪数据，包括时间信息和 CPU 使用率。 请注意，与设置每个函数关联的开销会影响运行时性能，并可能会影响分析数据，对于生命周期相对较短的函数，这一点更为明显。 此外，如果应用短时间内执行大量函数，则分析器可能会迅速超出它的文件大小限制，且不能再记录更多跟踪数据。
> - **Edit configurations：** 允许您更改上述“Sampled”和“Instrumented”记录配置的某些默认值，并将它们另存为自定义配置。 如需了解更多信息，请转到[创建记录配置](https://developer.android.google.cn/studio/profile/cpu-profiler#configurations)部分。

5. **记录按钮：** 用于开始和停止记录函数跟踪。 如需了解更多信息，请转到[记录和检查函数跟踪](https://developer.android.google.cn/studio/profile/cpu-profiler#method_traces)部分。
>注： 分析器还会报告 Android Studio 和 Android 平台添加到您的应用进程（如 JDWP、Profile Saver、Studio:VMStats、Studio:Perfa 以及 Studio:Heartbeat，尽管它们在线程活动时间线中显示的确切名称可能有所不同）的线程 CPU 使用率。 这表示 CPU 时间线中应用的 CPU 使用率还可反映这些线程使用的 CPU 时间。 您可以在线程活动时间线中查看其中的一些线程并监控其活动。 （不过，由于分析器线程执行原生代码，因此，您无法为它们记录函数跟踪数据。）Android Studio 将报告此数据，以便当线程活动及 CPU 使用率实际上是由应用代码引发时，您可以轻松识别。

### 记录和检查函数跟踪 ###
要开始记录函数跟踪，从下拉菜单中选择 **Sampled** 或 **Instrumented** 记录配置，或选择您创建的[自定义记录配置](https://developer.android.google.cn/studio/profile/cpu-profiler#configurations)，然后点击 **Record** <img src="https://user-gold-cdn.xitu.io/2018/8/6/1650e9376175da3f?w=24&h=24&f=png&s=704" style="vertical-align:middle;"/>。 与应用交互并在完成后点击 **Stop recording** <img src="https://user-gold-cdn.xitu.io/2018/8/6/1650e938a6b85760?w=24&h=24&f=png&s=246" style="vertical-align:middle;"/>。 分析器将自动选择记录的时间范围，并在函数跟踪窗格中显示其跟踪信息。如果您想检查另一个线程的函数跟踪，只需从线程活动时间线中选中它。
记录函数跟踪后的 CPU Profiler 视图，如下图所示：
![CPU Profiler](https://user-gold-cdn.xitu.io/2018/8/7/16513522c74d5886?w=900&h=519&f=png&s=287135)
如图所示，记录函数跟踪后的 CPU Profiler 视图包括以下内容：
1. **选择时间范围：** 用于确定您要在跟踪窗格中检查所记录时间范围的哪一部分。 当您首次记录函数跟踪时，CPU Profiler 将在 CPU 时间线中自动选择您的记录的完整长度。 如果您想仅检查所记录时间范围一小部分的函数跟踪数据，您可以点击并拖动突出显示的区域边缘以修改其长度。
2. **时间戳：** 用于表示所记录函数跟踪的开始和结束时间（相对于分析器从设备开始收集 CPU 使用率信息的时间）。 在选择时间范围时，您可以点击时间戳以自动选择完整记录，如果您有多个要进行切换的记录，则此做法尤其有用。
3. **跟踪窗格：** 用于显示您所选的时间范围和线程的函数跟踪数据。 仅在您至少记录一个函数跟踪后此窗格才会显示。 在此窗格中，您可以选择想如何查看每个堆叠追踪（使用跟踪标签），以及如何测量执行时间（使用时间引用下拉菜单）。
4. 选择后，可通过 **Top Down** 树、**Bottom Up** 树、**调用图表**或**火焰图**的形式显示您的函数跟踪。 您可以在下文中了解每个跟踪窗格标签的更多信息。
5. 从下拉菜单中选择以下选项之一，以确定如何测量每个函数调用的时间信息：
> - **Wall clock time：** 壁钟时间信息表示实际经过的时间。
> - **Thread time：** 线程时间信息表示实际经过的时间减去线程没有消耗 CPU 资源的任意时间部分。对于任何给定函数，其线程时间始终少于或等于其壁钟时间。 使用线程时间可以让您更好地了解线程的实际 CPU 使用率中有多少是给定函数消耗的。

## 使用 Memory Profiler 查看 Java 堆和内存分配 ##
Memory Profiler 是 [Android Profiler](https://developer.android.google.cn/studio/preview/features/android-profiler.html) 中的一个组件，可帮助您识别导致应用卡顿、冻结甚至崩溃的内存泄漏和流失。 它显示一个应用内存使用量的实时图表，让您可以捕获堆转储、强制执行垃圾回收以及跟踪内存分配。点击 **Android Profiler** 窗口中的 **MEMORY** 时间线中的任意位置即可打开 **Memory Profiler**。或者，您可以在命令行中使用 [dumpsys](https://developer.android.google.cn/studio/command-line/dumpsys.html) 检查您的应用内存，同时[查看 logcat 中的 GC Event](https://developer.android.google.cn/studio/debug/am-logcat.html#memory-logs)。

### 为什么应分析您的应用内存 ###
Android 提供一个[托管内存环境](https://developer.android.google.cn/topic/performance/memory-overview.html)—当它确定您的应用不再使用某些对象时，垃圾回收器会将未使用的内存释放回堆中。 虽然 Android 查找未使用内存的方式在不断改进，但对于所有 Android 版本，系统都必须在某个时间点短暂地暂停您的代码。 大多数情况下，这些暂停难以察觉。 不过，如果您的应用分配内存的速度比系统回收内存的速度快，则当收集器释放足够的内存以满足您的分配需要时，您的应用可能会延迟。 此延迟可能会导致您的应用跳帧，并使系统明显变慢。

尽管您的应用不会表现出变慢，但如果存在内存泄漏，则即使应用在后台运行也会保留该内存。 此行为会强制执行不必要的垃圾回收 Event，因而拖慢系统的内存性能。 最后，系统被迫终止您的应用进程以回收内存。 然后，当用户返回您的应用时，它必须完全重启。

为帮助防止这些问题，您应使用 Memory Profiler 执行以下操作：

 - 在时间线中查找可能会导致性能问题的不理想的内存分配模式。
 - 转储 Java 堆以查看在任何给定时间哪些对象耗尽了使用内存。长时间进行多个堆转储可帮助识别内存泄漏。
 - 记录正常用户交互和极端用户交互期间的内存分配以准确识别您的代码在何处短时间分配了过多对象，或分配了泄漏的对象。

如需了解可减少应用内存使用的编程做法，请阅读[管理您的应用内存](https://developer.android.google.cn/topic/performance/memory.html)。

### Memory Profiler 概览 ###
当您首次打开 Memory Profiler 时，您将看到一条表示应用内存使用量的详细时间线，并可访问用于强制执行垃圾回收、捕捉堆转储和记录内存分配的各种工具。Memory Profiler 的默认视图如下图所示：
![Memory Profiler](https://user-gold-cdn.xitu.io/2018/8/7/1651358b73ca33ff?w=900&h=427&f=png&s=147425)

如图所示，Memory Profiler 的默认视图包括以下各项：
1. 用于强制执行垃圾回收 Event 的按钮。
2. [用于捕获堆转储的按钮](https://developer.android.google.cn/studio/profile/memory-profiler#capture-heap-dump)。
3. [用于记录内存分配情况的按钮](https://developer.android.google.cn/studio/profile/memory-profiler#record-allocations)。 此按钮仅在连接至运行 Android 7.1 或更低版本的设备时才会显示。
4. 用于放大/缩小时间线的按钮。
5. 用于跳转至实时内存数据的按钮。
6. Event 时间线，其显示活动状态、用户输入 Event 和屏幕旋转 Event。
7. 内存使用量时间线，其包含以下内容：
> * 一个显示每个内存类别使用多少内存的堆叠图表，如左侧的 y 轴以及顶部的彩色键所示。
> * 虚线表示分配的对象数，如右侧的 y 轴所示。
> * 用于表示每个垃圾回收 Event 的图标。

不过，如果您使用的是运行 Android 7.1 或更低版本的设备，则默认情况下，并不是所有分析数据均可见。 如果您看到一条消息，其显示“Advanced profiling is unavailable for the selected process”，则需要[启用高级分析](https://developer.android.google.cn/studio/preview/features/android-profiler.html#advanced-profiling)以查看下列内容：
 * Event 时间线
 * 分配的对象数
 * 垃圾回收 Event

在 Android 8.0 及更高版本上，始终为可调试应用启用高级分析。

#### 如何计算内存 ####
您在 Memory Profiler 顶部看到的数字取决于您的应用根据 Android 系统机制所提交的所有私有内存页面数。 此计数不包含与系统或其他应用共享的页面。
Memory Profiler 顶部的内存计数图例，如下图所示：
![Memory Profiler Counts](https://user-gold-cdn.xitu.io/2018/8/7/16513597eb498e14?w=900&h=26&f=png&s=22990)

内存计数中的类别如下所示：

 - **Java：** 从 Java 或 Kotlin 代码分配的对象内存。
 - **Native：** 从 C 或 C++ 代码分配的对象内存。
> 即使您的应用中不使用 C++，您也可能会看到此处使用的一些原生内存，因为 Android 框架使用原生内存代表您处理各种任务，如处理图像资源和其他图形时，即使您编写的代码采用 Java 或 Kotlin 语言。

 - **Graphics：** 图形缓冲区队列向屏幕显示像素（包括 GL 表面、GL 纹理等等）所使用的内存。（请注意，这是与 CPU 共享的内存，不是 GPU 专用内存。）
 - **Stack：** 您的应用中的原生堆栈和 Java 堆栈使用的内存。 这通常与您的应用运行多少线程有关。
 - **Code：** 您的应用用于处理代码和资源（如 dex 字节码、已优化或已编译的 dex 码、.so 库和字体）的内存。
 - **Other：** 您的应用使用的系统不确定如何分类的内存。
 - **Allocated：** 您的应用分配的 Java/Kotlin 对象数。 它没有计入 C 或 C++ 中分配的对象。

当连接至运行 Android 7.1 及更低版本的设备时，此分配仅在 Memory Profiler 连接至您运行的应用时才开始计数。 因此，您开始分析之前分配的任何对象都不会被计入。 不过，Android 8.0 附带一个设备内置分析工具，该工具可记录所有分配，因此，在 Android 8.0 及更高版本上，此数字始终表示您的应用中待处理的 Java 对象总数。

与以前的 Android Monitor 工具中的内存计数相比，新的 Memory Profiler 以不同的方式记录您的内存，因此，您的内存使用量现在看上去可能会更高些。 Memory Profiler 监控的类别更多，这会增加总的内存使用量，但如果您仅关心 Java 堆内存，则“Java”项的数字应与以前工具中的数值相似。

然而，Java 数字可能与您在 Android Monitor 中看到的数字并非完全相同，这是因为应用的 Java 堆是从 Zygote 启动的，而新数字则计入了为它分配的所有物理内存页面。 因此，它可以准确反映您的应用实际使用了多少物理内存。

> 注：目前，Memory Profiler 还会显示应用中的一些误报的原生内存使用量，而这些内存实际上是分析工具使用的。 对于大约 100000 个对象，最多会使报告的内存使用量增加 10MB。 在这些工具的未来版本中，这些数字将从您的数据中过滤掉。

### 查看内存分配 ###
内存分配显示内存中每个对象是如何分配的。 具体而言，Memory Profiler 可为您显示有关对象分配的以下信息：
 - 分配哪些类型的对象以及它们使用多少空间。
 - 每个分配的堆叠追踪，包括在哪个线程中。
 - 对象在何时被取消分配（仅当使用运行 Android 8.0 或更高版本的设备时）。

如果您的设备运行 Android 8.0 或更高版本，您可以随时按照下述方法查看您的对象分配： 只需点击并按住时间线，并拖动选择您想要查看分配的区域。 不需要开始记录会话，因为 Android 8.0 及更高版本附带设备内置分析工具，可持续跟踪您的应用分配。如下如所示：
![](https://user-gold-cdn.xitu.io/2018/8/20/1655579d3b39c0c8?w=800&h=552&f=gif&s=2824291)

如果您的设备运行 Android 7.1 或更低版本，则在 Memory Profiler 工具栏中点击 **Record memory allocations** <img src="https://user-gold-cdn.xitu.io/2018/8/6/1650e9376175da3f?w=24&h=24&f=png&s=704" style="vertical-align:middle;"/>。 记录时，Android Monitor 将跟踪您的应用中进行的所有分配。 操作完成后，点击 **Stop recording** <img src="https://user-gold-cdn.xitu.io/2018/8/6/1650e938a6b85760?w=24&h=24&f=png&s=246" style="vertical-align:middle;"/>（同一个按钮）以查看分配。如下如所示：
![](https://user-gold-cdn.xitu.io/2018/8/20/165557bab738c330?w=800&h=552&f=gif&s=1907540)

在选择一个时间线区域后（或当您使用运行 Android 7.1 或更低版本的设备完成记录会话时），已分配对象的列表将显示在时间线下方，按类名称进行分组，并按其堆计数排序。

> 注：在 Android 7.1 及更低版本上，您最多可以记录 65535 个分配。 如果您的记录会话超出此限值，则记录中仅保存最新的 65535 个分配。 （在 Android 8.0 及更高版本中，则没有实际的限制。）

要检查分配记录，请按以下步骤操作：
1. 浏览列表以查找堆计数异常大且可能存在泄漏的对象。 为帮助查找已知类，点击 **Class Name** 列标题以按字母顺序排序。 然后点击一个类名称。 此时在右侧将出现 **Instance View** 窗格，显示该类的每个实例，如下图中所示。
2. 在 **Instance View** 窗格中，点击一个实例。 此时下方将出现 **Call Stack** 标签，显示该实例被分配到何处以及哪个线程中。
3. 在 **Call Stack** 标签中，点击任意行以在编辑器中跳转到该代码。
有关每个已分配对象的详情在右侧的 Instance View 中的显示，如下图所示：
![Memory Profiler Allocations Detail](https://user-gold-cdn.xitu.io/2018/8/7/165135a30bccf7ad?w=900&h=509&f=png&s=355818)

默认情况下，左侧的分配列表按类名称排列。在列表顶部，您可以使用右侧的下拉列表在以下排列方式之间进行切换：
 - **Arrange by class：** 基于类名称对所有分配进行分组。
 - **Arrange by package：** 基于软件包名称对所有分配进行分组。
 - **Arrange by callstack：** 将所有分配分组到其对应的调用堆栈。

### 捕获堆转储 ###
堆转储显示在您捕获堆转储时您的应用中哪些对象正在使用内存。 特别是在长时间的用户会话后，堆转储会显示您认为不应再位于内存中却仍在内存中的对象，从而帮助识别内存泄漏。 在捕获堆转储后，您可以查看以下信息：

 - 您的应用已分配哪些类型的对象，以及每个类型分配多少。
 - 每个对象正在使用多少内存。
 - 在代码中的何处仍在引用每个对象。
 - 对象所分配到的调用堆栈（目前，如果您在记录分配时捕获堆转储，则只有在 Android 7.1 及更低版本中，堆转储才能使用调用堆栈）。

要捕获堆转储，在 Memory Profiler 工具栏中点击 **Dump Java heap** <img src="https://user-gold-cdn.xitu.io/2018/8/7/1651331695f5cd62?w=24&h=24&f=png&s=902" style="vertical-align:middle;"/>。 在转储堆期间，Java 内存量可能会暂时增加。 这很正常，因为堆转储与您的应用发生在同一进程中，并需要一些内存来收集数据。

堆转储显示在内存时间线下，显示堆中的所有类类型，如下图所示：
![Memory Profiler Dump](https://user-gold-cdn.xitu.io/2018/8/7/165132c7acf3cd47?w=800&h=556&f=png&s=258174)

> 注：如果您需要更精确地了解转储的创建时间，可以通过调用 [dumpHprofData()](https://developer.android.google.cn/reference/android/os/Debug.html#dumpHprofData(java.lang.String)) 在应用代码的关键点创建堆转储。

要检查您的堆，请按以下步骤操作：
1. 浏览列表以查找堆计数异常大且可能存在泄漏的对象。 为帮助查找已知类，点击 **Class Name** 列标题以按字母顺序排序。 然后点击一个类名称。此时在右侧将出现 **Instance View** 窗格，显示该类的每个实例，如下图中所示。
2. 在 **Instance View** 窗格中，点击一个实例。此时下方将出现 **References**，显示该对象的每个引用。
或者，点击实例名称旁的箭头以查看其所有字段，然后点击一个字段名称查看其所有引用。 如果您要查看某个字段的实例详情，右键点击该字段并选择 **Go to Instance**。
3. 在 **References** 标签中，如果您发现某个引用可能在泄漏内存，则右键点击它并选择 **Go to Instance**。 这将从堆转储中选择对应的实例，显示您自己的实例数据。

默认情况下，堆转储不会向您显示每个已分配对象的堆叠追踪。 要获取堆叠追踪，在点击 Dump Java heap 之前，您必须先开始记录内存分配。然后，您可以在 Instance View 中选择一个实例，并查看 Call Stack 标签以及 References 标签，如下图所示。不过，在您开始记录分配之前，可能已分配一些对象，因此，调用堆栈不能用于这些对象。包含调用堆栈的实例在图标  上用一个“堆栈”标志表示。（遗憾的是，由于堆叠追踪需要您执行分配记录，因此，您目前无法在 Android 8.0 上查看堆转储的堆叠追踪。）

默认情况下，堆转储不会向您显示每个已分配对象的堆叠追踪。 要获取堆叠追踪，在点击 Dump Java heap 之前，您必须先开始记录内存分配。然后，您可以在 Instance View 中选择一个实例，并查看 Call Stack 标签以及 References 标签，如下图所示。不过，在您开始记录分配之前，可能已分配一些对象，因此，调用堆栈不能用于这些对象。包含调用堆栈的实例在图标 <img src="https://user-gold-cdn.xitu.io/2018/8/7/16513318e337e050?w=24&h=24&f=png&s=971" style="vertical-align:middle;"/> 上用一个“堆栈”标志表示。（遗憾的是，由于堆叠追踪需要您执行分配记录，因此，您目前无法在 Android 8.0 上查看堆转储的堆叠追踪。）

在您的堆转储中，请注意由下列任意情况引起的内存泄漏：

 - 长时间引用 Activity、Context、View、Drawable 和其他对象，可能会保持对 Activity 或 Context 容器的引用。
 - 可以保持 Activity 实例的非静态内部类，如 Runnable。
 - 对象保持时间超出所需时间的缓存。

捕获堆转储需要的持续时间标示在时间线中，如下图所示：
![Memory Profiler Dump Stacktrace](https://user-gold-cdn.xitu.io/2018/8/7/1651337080c57ed4?w=900&h=474&f=png&s=360396)

在类列表中，您可以查看以下信息：

 - **Heap Count：** 堆中的实例数。
 - **Shallow Size：** 此堆中所有实例的总大小（以字节为单位）。
 - **Retained Size：** 为此类的所有实例而保留的内存总大小（以字节为单位）。

在类列表顶部，您可以使用左侧下拉列表在以下堆转储之间进行切换：

 - **Default heap：** 系统未指定堆时。
 - **App heap：** 您的应用在其中分配内存的主堆。
 - **Image heap：** 系统启动映像，包含启动期间预加载的类。 此处的分配保证绝不会移动或消失。
 - **Zygote heap：** 写时复制堆，其中的应用进程是从 Android 系统中派生的。

默认情况下，此堆中的对象列表按类名称排列。 您可以使用其他下拉列表在以下排列方式之间进行切换：

 - **Arrange by class：** 基于类名称对所有分配进行分组。
 - **Arrange by package：** 基于软件包名称对所有分配进行分组。
 - **Arrange by callstack：** 将所有分配分组到其对应的调用堆栈。此选项仅在记录分配期间捕获堆转储时才有效。即使如此，堆中的对象也很可能是在您开始记录之前分配的，因此这些分配会首先显示，且只按类名称列出。

默认情况下，此列表按 **Retained Size** 列排序。 您可以点击任意列标题以更改列表的排序方式。

在 Instance View 中，每个实例都包含以下信息：

 - **Depth：** 从任意 GC 根到所选实例的最短 hop 数。
 - **Shallow Size：** 此实例的大小。
 - **Retained Size：** 此实例支配的内存大小（根据 [dominator 树](https://en.wikipedia.org/wiki/Dominator_(graph_theory))）。

#### 将堆转储另存为 HPROF ####
在捕获堆转储后，仅当分析器运行时才能在 Memory Profiler 中查看数据。 当您退出分析会话时，您将丢失堆转储。 因此，如果您要保存堆转储以供日后查看，可通过点击时间线下方工具栏中的 **Export heap dump as HPROF file** <img src="https://user-gold-cdn.xitu.io/2018/8/7/165133f23446e14d?w=24&h=24&f=png&s=1086" style="vertical-align:middle;"/>，将堆转储导出到一个 HPROF 文件中。 在显示的对话框中，确保使用 `.hprof` 后缀保存文件。

然后，通过将此文件拖到一个空的编辑器窗口（或将其拖到文件标签栏中），您可以在 Android Studio 中重新打开该文件。

要使用其他 HPROF 分析器（如 [jhat](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jhat.html)），您需要将 HPROF 文件从 Android 格式转换为 Java SE HPROF 格式。 您可以使用 `android_sdk/platform-tools/` 目录中提供的 `hprof-conv` 工具执行此操作。 运行包括以下两个参数的 `hprof-conv` 命令：原始 HPROF 文件和转换后 HPROF 文件的写入位置。 例如：
```shell
hprof-conv heap-original.hprof heap-converted.hprof
```

### 分析内存的技巧 ###
使用 Memory Profiler 时，您应对应用代码施加压力并尝试强制内存泄漏。 在应用中引发内存泄漏的一种方式是，先让其运行一段时间，然后再检查堆。 泄漏在堆中可能逐渐汇聚到分配顶部。 不过，泄漏越小，您越需要运行更长时间的应用才能看到泄漏。

您还可以通过以下方式之一触发内存泄漏：

 - 将设备从纵向旋转为横向，然后在不同的 Activity 状态下反复操作多次。 旋转设备经常会导致应用泄漏 [Activity](https://developer.android.google.cn/reference/android/app/Activity.html)、[Context](https://developer.android.google.cn/reference/android/content/Context.html) 或 [View](https://developer.android.google.cn/reference/android/content/Context.html) 对象，因为系统会重新创建 [Activity](https://developer.android.google.cn/reference/android/app/Activity.html)，而如果您的应用在其他地方保持对这些对象之一的引用，系统将无法对其进行垃圾回收。
 - 处于不同的 Activity 状态时，在您的应用与另一个应用之间切换（导航到主屏幕，然后返回到您的应用）。

> 提示： 您还可以使用 [monkeyrunner](https://developer.android.google.cn/tools/help/monkeyrunner_concepts.html) 测试框架执行上述步骤。

## 利用 Network Profiler 检查网络流量 ##
Network Profiler 能够在时间线上显示实时网络活动，包括发送和接收的数据以及当前的连接数。 这便于您查看应用传输数据的方式和时间，并据此对底层代码进行适当优化。点击 **Android Profiler** 窗口中的 **NETWORK** 时间线中的任意位置即可打开 **Network Profiler**。

### 为什么应分析应用的网络活动 ###
当您的应用向网络发出请求时，设备必须使用高功耗的移动或 WLAN 无线装置来收发数据包。无线装置不仅要消耗电力来传输数据，还需要消耗额外的电力来开启并且不锁定屏幕。

使用 Network Profiler，您可以查找频繁出现的短时网络活动峰值，这意味着您的应用需要经常打开无线装置，或需要长时间不锁定屏幕以处理集中出现的大量短时请求。这种模式说明您可以通过批量处理网络请求，减少必须开启无线装置来发送或接收数据的次数，从而优化应用，改善电池续航表现。这种方式还能让无线装置调整到低能耗模式，延长批量处理请求之间的间隔时间，节省能耗。

要详细了解优化应用网络活动 的相关技巧，请参阅[减少网络耗电量](https://developer.android.google.cn/topic/performance/power/network/index.html)。

### Network Profiler 概览 ###
Network Profiler 的默认视图如下图所示：
![Network Profiler](https://user-gold-cdn.xitu.io/2018/8/7/1651348d8674a836?w=900&h=525&f=png&s=363578)
窗口的顶部的①处,可以看见wifi无线信号的强弱，在时间线上可以在②处点击和拖动一部分的时间线来检测流量，然后在窗口③中会显示所选时间段内收发的文件，包括文件名，大小，类型，状态和花费时间，你可以对窗口③的列表根据列来进行排序。还可以查看所选时间段的详细拆分，拆分的timeline可以显示文件是什么时候收发的，点击窗口3的其中一个文件，可以在窗口④中查看文件的详细信息。通过切换窗口④上方标签可以查看响应数据、标题信息和调用堆栈。

> 注： 必须启用高级分析才能从时间线中选择要检查的片段，查看发送和接收的文件列表，或查看有关所发送或接收的选定文件的详细信息。 要启用高级分析，请参阅[启用高级分析](https://developer.android.google.cn/studio/profile/android-profiler.html#advanced-profiling)。

### 排查网络连接问题 ###
如果 Network Profiler 检测到流量值，但无法识别任何受支持的网络请求，您会收到以下错误消息：

"**Network Profiling Data Unavailable:** There is no information for the network traffic you've selected."

Network Profiler 目前只支持 [HttpURLConnection](https://developer.android.google.cn/reference/java/net/HttpURLConnection.html) 和 [OkHttp](http://square.github.io/okhttp/) 网络连接库。如果您的应用使用的是其他网络连接库，则可能无法在 Network Profiler 中查看网络活动。 如果您收到这条错误消息，但您的应用确实使用了 HttpURLConnection 或 OkHttp，请[报告错误](https://developer.android.google.cn/studio/report-bugs.html)或[搜索 Issue Tracker](https://issuetracker.google.com/issues?q=componentid:192708+)，在与您的问题有关的现有报告中加入您的反馈。 此外，您还可以利用以下资源请求提供关于其他库的支持。

## 致谢 ##
[利用 Android Profiler 测量应用性能](https://developer.android.google.cn/studio/profile/android-profiler)

[Measure app performance with Android Profiler](https://developer.android.com/studio/profile/android-profiler)

