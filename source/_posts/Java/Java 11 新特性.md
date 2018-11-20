---
title: Java 11 新特性
date: 2018-11-18 10:25:18
categories: Java
tags:
  - Java
  - JDK
---

自从 2017 年 9 月 21 日 Java 9 正式发布之时，Oracle 就宣布今后会按照每六个月一次的节奏进行更新，JDK 11 已于 2018 年 9 月发布。与 JDK 10 不同，JDK 11 将提供长期支持，还将作为 Java 平台的参考实现以及标准版（Java SE）11。Oracle 直到 2023 年 9 月都会为 JDK 11 提供一级支持，而补丁和安全警告等扩展支持将延续到 2026 年。新的长期支持版本每三年发布一次，根据后续的发行计划，JDK 17 将于 2021 年发布。

## Java 11 的新特性 ##
Java 11 新增了非常多的特性，主要特性包含以下几个：
 - **`基于嵌套的访问控制`** − 嵌套是一种访问控制上下文，与 Java 编程语言中现有的嵌套类型概念一致。 嵌套允许逻辑上属于同一代码实体，但被编译为不同类文件的类，无需编译器插入可访问性扩展桥接方法，即可访问彼此的私有成员。
 - **`动态类文件常量`** − 扩展 Java 类文件格式以支持新的常量池形式，CONSTANT_Dynamic。 加载CONSTANT_Dynamic 会将创建委托给 bootstrap 方法，就像链接 invokedynamic 调用站点将链接委托给 bootstrap 方法一样。
 - **`改进 Aarch64 内联函数`** − 改进现有的字符串和数组内联函数，并在 AArch64 处理器上为 java.lang.Math sin，cos 和 log 函数实现新的内联函数。
 - **`Epsilon：No-Op 垃圾回收器`** − 开发一个处理内存分配但不实现任何实际内存回收机制的 GC。 一旦可用的 Java 堆耗尽，JVM 将关闭。
 - **`HTTP 客户端（标准）`** − 通过 JEP 110 标准化 JDK 9 中引入的孵化 HTTP 客户端 API，并在 JDK 10 中进行更新。
 - **`Lambda 参数的局部变量语法`** − 在声明隐式类型的 lambda 表达式的形式参数时允许使用 var。
 - **`采用 Curve25519 和 Curve448 加密的密钥协议`** − 使用 RFC 7748 中描述的 Curve25519 和 Curve448 实现密钥协议。
 - **`支持 Unicode 10.0`** − 升级现有平台 API 以支持 Unicode 标准 v10.0。
 - **`飞行记录仪`** − 提供低开销的数据收集框架，用于对 Java 应用程序和 HotSpot JVM 进行故障排除。
 - **`实现 ChaCha20 和 Poly1305 加密算法`** − 实现 RFC 7539 中指定的 ChaCha20 和 ChaCha20-Poly1305 加密。ChaCha20 是一种相对较新的流加密，可以替代旧的、不安全的 RC4 流加密。
 - **`增强 Java 启动器`** − 增强 java 启动程序以运行作为 Java 源代码的单个文件提供的程序，包括通过“shebang”文件和相关技术从脚本中使用。
 - **`低开销堆分析`** − 提供一种低开销的 Java 堆分配采样方法，可通过 JVMTI 访问。
 - **`传输层安全性(TLS) 1.3`** − 实现传输层安全性（TLS）协议 RFC 8446 的 1.3 版。
 - **`ZGC：可扩展的低延迟垃圾收集器`** − Z 垃圾收集器，也称为 ZGC，是一个可扩展的低延迟垃圾收集器。
 - **`弃用 Nashorn JavaScript 引擎`** − 弃用 Nashorn JavaScript 脚本引擎和 API 以及 jjs 工具，意图在将来的版本中删除它们。
 - **`弃用 Pack200 工具和 API`** − 在 java.util.jar 中弃用 pack200 和 unpack200 工具以及 Pack200 API。
 - **`删除 Java EE 和 CORBA 模块`** − 从 Java SE Platform 和 JDK 中删除 Java EE 和 CORBA 模块。这些模块在 Java SE 9 中已弃用，声明的目的是为了在将来的版本中删除它们。
 - **`删除 JavaFX`** − JavaFX 模块已从 JDK 11 发行版中删除。这些模块包含在早期版本的 Oracle JDK 中，但不包含在 OpenJDK 版本中。JavaFX 模块将作为 JDK 之外的单独模块集提供。

更多的新特性可以参阅官网：[What's New in JDK 11](https://www.oracle.com/technetwork/java/javase/11-relnote-issues-5012449.html#NewFeature)
JDK 11 下载地址：[Java 11 Downloads](https://www.oracle.com/technetwork/java/javase/downloads/jdk11-downloads-5066655.html)

### HTTP客户端（标准） ###
这个功能于 JDK 9 中引入并在 JDK 10 中得到了更新，现在终于转正了。该 API 通过 `CompleteableFutures` 提供非阻塞请求和响应语义，可以联合使用以触发相应的动作。自从 JDK 9 和 10 中引入该功能后，JDK 11 完全重写了该功能，现在其实现完全是异步的。`RX Flow` 的概念也得到了实现，这样就无需为了支持 HTTP/2 而创造许多概念了。现在，在用户层请求发布者和响应发布者与底层套接字之间追踪数据流更容易了。这降低了复杂性，并最大程度上提高了 HTTP/1 和 HTTP/2 之间的重用的可能性。

### Epsilon 垃圾回收器 ###
Epsilon 垃圾回收器又被称为 `no-op` 回收器，它是新的实验性无操作垃圾收集器。Epsilon GC 仅处理内存分配，并且不实现任何内存回收机制。它对性能测试非常有用，可以与其他 GC 的成本/收益进行对比。它可用于在测试中方便地断言内存占用和内存压力。在极端情况下，它可能对非常短暂的作业很有用，其中内存回收将在 JVM 终止时发生，或者在低垃圾应用程序中获得最后一次延迟改进。

### Lambda 参数的局部变量语法 ###
可以消除隐含类型表达式中正式参数定义的语法与局部变量定义语法的不一致。这样就能在隐含类型的 `lambda` 表达式中定义正式参数时使用 `var` 了。

### Java的类文件格式将被扩展 ###
以支持新的常量池，`CONSTANT_Dynamic`。其目标是降低开发新形式的可实现类文件约束带来的成本和干扰。

### 采用 Curve25519 和 Curve448 加密的密钥协议 ###
比现有的 `Diffie-Hellman` 椭圆曲线密钥交换方式更有效、更安全。根据 IETF 的资料，`Curve25519` 和 `Curve448` 两种椭圆曲线采用常量时间的实现方式，以及不会发生异常的数乘实现，能更好地抵抗各种旁路，包括时序、缓存等。该提案的目标是为密钥交换方法提供一个 API 和实现，同时开发一个平台无关、纯 Java 的的实现。由于该提案采用了复杂且精密的模算数，因此还是有风险的。

### 飞行记录仪（Flight Recorder） ###
将提供`低开销`的数据收集框架，用来调试 `Java 应用程序`和 `HotSpot JVM`。飞行记录仪是 Oracle 的商业版 JDK 的功能，但在 JDK 11 中，其代码将移动到公开代码库中，这样所有人都能使用该功能了。`Iclouded` 将作为 API，以事件的形式产生或消耗数据，同时提供缓存机制、二进制数据工具，同时支持配置和事件过滤。该提案还提议为 OS、HotSpot 和 JDK 库提供事件。

### 支持 Unicode 10.0 ###
升级现有平台 API 以支持 `Unicode` 标准 `v10.0`，从而使 Java 跟上潮流。
JDK 11 版本包括对 `Unicode 10.0.0` 的支持。自从支持 Unicode 8.0.0 的 JDK 10 发布以来，JDK 11 结合了 Unicode 9.0.0 和 10.0.0 版本，包括：
 - 16018 个新字符
 - 18 个新区块
 - 10 个新脚本

### 实现 ChaCha20 和 Poly1305 加密算法 ###
`ChaCha20` 是种相对较新的流加密算法，能代替旧的、不安全的R4流加密。`ChaCha20` 将与 `Poly1305` 认证算法配对使用。`ChaCha20` 和 `ChaCha20-Poly1305` 加密实现将通过 `crypto.CipherSpi` API 于 SunJCE（Java加密扩展）中提供。

### 增强Java启动器 ###
使之能够运行单一文件的 Java 源代码，使得应用程序可以直接从源代码运行。单文件程序常见于小型工具，或开发者初学 Java 时使用。而且，单一源代码文件有可能会编译成多个类文件，这会增加打包开销。由于这些原因，在运行程序之前进行编译，已成为了不必要的步骤。

### 低开销堆分析 ###
提供一种`低开销`的 Java 堆分配采样方法，可通过 `JVMTI` 访问。它旨在实现以下目标：
 - 低开销足以在默认情况下持续启用；
 - 可通过定义明确的编程接口（JVMTI）访问；
 - 可以对所有分配进行采样（即，不限于在一个特定堆区域中的分配或以特定方式分配的分配）；
 - 可以以独立于实现的方式定义（即，不依赖于任何特定的 GC 算法或 VM 实现）；
 - 可以提供有关实时和死 Java 对象的信息。

### ZGC：可扩展的低延迟垃圾收集器(试验) ###
`Z 垃圾收集器`，也称为 `ZGC`，是一个可扩展的低延迟垃圾收集器（JEP 333）。它旨在实现以下目标：
 - 暂停时间不超过10毫秒；
 - 暂停时间不会随堆或实时设置大小而增加；
 - 处理大小从几百兆到几千兆字节不等的堆；
 - ZGC 的核心是并发垃圾收集器，这意味着在 Java 线程继续执行时，所有繁重的工作（标记，压缩，引用处理，字符串表清理等）都已完成。这极大地限制了垃圾收集对应用程序响应时间的负面影响。

ZGC 作为实验性功能包含在内。要启用它，需要 `-XX:+UnlockExperimentalVMOptions` 选项与 `-XX:+UseZGC` 选项结合使用。ZGC 的这个实验版具有以下限制：
 - 它仅适用于Linux / x64。
 - 不支持使用压缩的 oops 和/或压缩的类点。在 -XX:+UseCompressedOops 和 -XX:+UseCompressedClassPointers 选项默认情况下禁用。启用它们将不起作用。
 - 不支持类卸载。在 -XX:+ClassUnloading 和 -XX:+ClassUnloadingWithConcurrentMark 选项默认情况下禁用。启用它们将不起作用。
 - 不支持将 ZGC 与 Graal 结合使用。

### 传输层安全(TLS) 1.3 ###
JDK 11 版本包括传输层安全性(TLS) 1.3规范（RFC 8446）的实现。对于 `TLS 1.3`，定义了以下新标准算法名称：
 - TLS 协议版本名称：TLSv1.3 
 - SSLContext 算法名称：TLSv1.3 
 - TLS 1.3 的 TLS 密码套件名称：TLS_AES_128_GCM_SHA256，TLS_AES_256_GCM_SHA384 
 - X509KeyManager 的密钥类型：RSASSA-PSS 
 - X509TrustManager 的 authType：RSASSA-PSS

## 参考资源 ##
 - 参考 [Java 11 官方文档](https://docs.oracle.com/javase/11/)，了解 Java 11 的更多内容。
 - 参考 [Java 11 API 文档](https://docs.oracle.com/en/java/javase/11/docs/api/index.html)，了解 Java 11 API 的细节。 
 - 参考 [What's New in JDK 11](https://www.oracle.com/technetwork/java/javase/11-relnote-issues-5012449.html#NewFeature)，了解 Java 11 的新特性。
 - 参考 [Java 11 Release Notes](https://www.oracle.com/technetwork/java/javase/11u-relnotes-5093844.html)，了解 Java 11 的更新说明。
