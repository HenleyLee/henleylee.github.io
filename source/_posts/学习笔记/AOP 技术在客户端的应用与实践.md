---
title: AOP 技术在客户端的应用与实践
categories: 学习笔记
tags:
  - 学习笔记
abbrlink: 4c9f1b82
date: 2019-05-15 19:16:25
---

## 常见的编程架构思想 ##
 - 面向对象(Object Oriented Programming) 
 - 面向过程(Procedure Oriented Programming)
 - 面向切面(Aspect Oriented Programming)

以上三种编程架构思想中，面向对象和面向过程是我们大多数开发人员接触比较多的。

其实，对于大部分开发者，前两者编程思想并不陌生。接触更少的应该是面向切面的编程，本文重点和大家浅析一下切面编程在客户端的应用。

## 面向切面编程(AOP) ##
**`AOP`**是 `Aspect Oriented Programming` 的缩写，意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能统一维护的一种技术。

`AOP` 是 `OOP` 的延续，是软件开发中的一个热点，是函数式编程的一种衍生范型。利用 `AOP` 可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

`AOP` 其实是 `OOP` 的补充，`OOP` 从横向上区分出一个个的类来，而 `AOP` 则从纵向上向对象中加入特定的代码。用一句更加形象的话讲，`AOP` 就像一把刀，它能将整个过程按照你想要的切点进行切片，让整个过程清晰可见。
![面向切面编程(AOP)](https://henleylee.github.io/medias/study/aop_transverse.webp)

## AOP 的分类以及常见概念 ##
`AOP` 技术可以简单的分为两类，一类是运行期的 `AOP`，例如 Java 的动态代理；另一类可以归结为编译期的 `AOP`，例如经常听说的 `ASM`、`AspecJ` 等框架。

对于客户端而言，对性能的影响度是一个重要的考量目标，因为客户端本身运行的环境是有限的，不过随着手机技术的不断更新迭代，性能不断的飙升，这个问题也逐渐慢慢退化。然而，和大型服务器的处理能力相比，手机的处理能力还是略显逊色的。因此，在客户端应用我们重点考虑的仍然是编译期的 AOP 方案，以提高运行期的效率。

`AOP` 常见概念一般包含：`连接点(Joinpoint)`、`切点(Pointcut)`、`增强(Advice)`、`目标对象(Target)`、`引介(Introduction)`、`织入(Weaving)`、`代理(Proxy)`、`切面(Aspect)` 等：
**`连接点(Joinpoint)：`**程序执行的某个特定的位置，如：类开始初始化前、类初始化后、类的某个方法调用前、类的某个方法调用后、方法抛出异常后等。
**`切点(Pointcut)：`**指定一个 Adivce 将被引发的一系列连接点的集合。AOP框架必须允许开发者指定切入点，例如，使用正则表达式。
**`增强(Advice)：`**在特定的连接点，AOP 框架执行的动作。Advice 的类型的包括 around、before 和 throws。
**`目标对象(Target)：`**需要对它进行增强的业务类。
**`引介(Introduction)：`**添加方法或字段到被增强的类，引入新的接口到任何被增强的对象。
**`织入(Weaving)：`**将增强添加到目标类具体连接点上的过程，分为编译期织入、类装载期织入、动态代理织入。
**`代理(Proxy)：`**一个类被 AOP 织入后生成出了一个结果类，它是融合了原类和增强逻辑的代理类。
**`切面(Aspect)：`**由切点和增强(或引介)组成，或者只由增强(或引介)实现。

## 客户端开发中的应用 ##
目前来说 `AOP` 在后端应用的场景已经非常完善了，比如说 Spring 框架自身对就 AOP 编程有一个很好的支持。对于起步稍晚的客户端而言，AOP 却显得没有那么多的应用场景。如何能将 AOP 的思想，融入到客户端的开发，从而提升客户端的开发效率，是我们一直在思考与实践的一个问题。

了解了以上内容之后，下边着重介绍一下 `AOP` 编程思想在网易新闻客户端的应用与实践：

### Hot Fix (热修复) ###
两年之前 `Hot Fix` 技术炒的非常热，各大友商也是花样百出，各自献计，一时间出现了百家齐鸣的现象。当然网易其实也不甘落后，公司各个部门也研发了自己的热更新热补丁方案。网易新闻也是不断的探索未知，先后实践过一些方案，最终还是基于 `AOP` 的方案，制定了一套相对成熟稳定的热修复技术。
 - 原理
基于 AOP 技术，编译期修改 Java 字节码，对每一个方法进行插桩操作，以便于 hook 每一个方法，做到方法级别的热修复。

 - AOP框架选型
编译期的 AOP 框架有很多，例如 ASM、AspectJ、Javassist 等等。如何选择框架其实是我们研发初期面临的一个问题。我们最终选的是使用 AspectJ 作为研发框架，原因有以下几点：
1. 易用性/门槛低：API 设计非常简单，无需你深入的了解 .class 文件的结构，就能轻易的驾驭它。
2. 对 Java 语言的完美支持：支持各种表达式，满足你对切点的自定义。

 - Android热修复的实现
![Android热修复的实现](https://henleylee.github.io/medias/study/aop_android_hot.webp)
如上图，实现过程主要有以下几个关键点：
1. 方法级别的 hook
如图所示，gradle 进行构建的时候，在 Java 源码编译完成之后，生成 dex 文件之前，我们调用 AspectJ 的编译器进行插桩。插桩的目的是给每一个方法注入一段寻找 patch 方法的逻辑。简单来说就是每个方法执行时先去根据自己方法的签名寻找是否有自己对应的 patch 方法，如果有，执行 patch 方法；没有，执行自己原有的逻辑。大致代码逻辑如下：
```java
public Object weaveJoinPoint(ProceedingJoinPoint joinPoint) throws Throwable {
    try {
        String e = joinPoint.toLongString(); //获取当前方法的签名
        PatchDebug.printMethodDesByFile(e);
        if (PatchUtils.hasMethodPatch(e)) {  //查询是否有自己方法的patch方法
            Object target = joinPoint.getTarget(); //获取当前的运行对象，方便设置给patch方法
            Object[] params = joinPoint.getArgs(); //获取运行参数
              .................................          //省略一些处理逻辑
            return joinPoint.proceed();//执行原有逻辑
        }
    } catch (Exception var8) {
        var8.printStackTrace();
    }
    return joinPoint.proceed();
}
```
2. 何时进行插桩操作以及如何进行插桩
插桩的时机：最基本的原则是：java 源码编译完成之后，Dex 文件生成之前。
对于 gradle 构建来说，我们可以有两种选择：
	a. 直接 hook java Compiler 的 Task，在 java 源码编译完成之后执行 AspectJ 的编译器，进行字节码插桩操作。
	```gradle
	project.android.applicationVariants.all { variant ->
	    if (variant.buildType.name != "release") {
		log.debug("Skipping non-release build type '${variant.buildType.name}'.")
		return
	    }
	    JavaCompile javaCompile = variant.javaCompile
	    javaCompile.doLast {
		//执行aspectJ编译操作
		String[] args = ["-showWeaveInfo",
				 "-1.5",
				 "-inpath", javaCompile.destinationDir.toString(),
				 "-aspectpath", javaCompile.classpath.asPath,
				 "-d", javaCompile.destinationDir.toString(),
				 "-classpath", javaCompile.classpath.asPath,
				 "-bootclasspath", project.android.bootClasspath.join(File.pathSeparator)]
		log.debug "ajc args: " + Arrays.toString(args)
		MessageHandler handler = new MessageHandler(true)
		new Main().run(args, handler)
	    }
	}
	```
	b. gradle Transform API 方案(推荐使用)
	Transform API 是 Android gradle plugin 1.5 之后新 API， 作用就是在生成 dex 之前，给开发者一个机会能够统一进行修改字节码。

	上述 a 方案存在一些问题，对于一些 jar 包或者三方 sdk 的类，不太方便处理。因此采用 Android 官方推荐的 Transform API 的方案是一个比较理想的方案，可以根据需求选择性定制处理哪些文件。 用 Transform API 实现一个 gradle 插件，对字节码处理。能够动态可配置的选择处理哪些 jar 或者哪些文件。这就大大的方便了对自己的代码和第三方的代码进行字节码的处理。
3. 补丁包如何生成
出现 bug 时将需要进行替换的方法放到指定类，然后生成一个只含有此 java 类字节码的 apk 包，进行下发。
如何将 patch 包做到最小？我们的方案是：使用发版时的 mapping 文件，正常进行 apk 的编译，在生成 dex 之前，我们强制性的移除掉现有的类，只保留新增的含有 patch 方法的 class 文件；按照同样的思想移除掉多余的资源文件，最终生一个只含有补丁方法的类的 apk 或者 jar。
4. 补丁包加载与运行
运行时通过 DexFile.loadDex() 将 apk 加载。然后按照方法签名将补丁方法做缓存处理，每一个方法运行时会去缓存里查找是否有自己对应的 patch 方法，如果有则执行 patch 方法。
5. 优缺点
| 优点           | 缺点                             | 
| -------------- | -------------------------------- |
| 无兼容行问题   | apk 原始包方法数增多，体积增大   |
| 实时生效       | 打 release 包时间变长            |
| patch 包体积小 | 运行时性能有一定影响             |
对性能的影响：我们通过对线上埋点测试发现，对方法的影响是毫秒级的，但是从整体来讲，例如从整个启动到主页面加载完成这个过程性能影响在 500ms 左右。
6. 优化方案
	a. 使用更轻量级别的 AOP 框架，减少包体积和性能影响；
	b. 使用类级别的热修复方案。

### 自动化工具 ###
 - 自动化监测方法耗时
如何自动化的检测和监控主线程方法耗时，降低 ANR 产生的风险，是一个客户端开发非常关注的问题。为了能快速针对方法耗时做一个监控，我们利用 AspectJ 框架，hook 每一个方法，获取方法执行的耗时时间，如果是主线程方法并且超过了我们设置的耗时阀值，则自动输出警告日志，便于开发人员检测此方法是否存在耗时逻辑。
 - 自动化日志记录
用户行为操作日志，无论是客户端排查问题，还是对用户行为的分析都是一个非常重要的埋点日志。因此为了能够自动化的记录用户的操作日志，我们通过使用 AOP 的思想，自动化 hook Activity 生命周期和 Fragment 声明周期的方法，从而自动化的做到用户行为的埋点上报和日志记录。
 - 三方 SDK 的检测与修改
	a. 检测三方 SDK 方法级别耗时，以便于对 SDK 有一个整体的评估。
	b. 上线前临时修改或者 hook 三方 SDK 的方法，做到快速容灾上线。
	c. 自动化抓取三方 SDK 网络和行为日志。
 - 统一处理点击抖动
客户端点击抖动是一个比较尴尬的问题，一方面此问题是客观存在的，确实对用户的体验有一定的影响；另一方面从开发角度讲，case by case 的处理是一个繁琐的过程。所以整体来看是一个收益不高但是代价不小的问题。
为了能够以最小的代价，解决这个全局性的问题，我们使用了 AOP 的思想，编译阶段统一 hook android.view.View.OnClickListener#onClick(View v) 方法；做一个快速点击无效的防抖动效果，统一解决客户端快速点击多次响应的问题。
这种方案的优点在于，统一处理，全局生效，开发阶段代码无侵入性，开发人员完全无感知，同时代价又很低。

### APM(应用性能监控) ###
应用程序性能的监控，是一个商业化应用发展到一定阶段不得不关注的问题。如果纯粹依靠开发人员手动埋点，去监控客户端性能，则存在一些弊端，比如说：维护成本高，和业务耦合性强，无法快速迁移到其他产品中。

为了解决这一问题，我们公司内部也是依靠 AOP 思想，开发了一套完整的应用性能监控平台。尽量做到开发人员无感知，无侵入的实现，应用网路、UI 交互的性能监控。

网络数据监控：网络层通过对网络库的方法 hook 和拦截器的自动化注入，实现网络过程的全过程监控，包括：握手时长，首包时间，DNS 耗时，网络耗时等阶段信息的获取。通过对整个网络过程的数据表现分析，找到网络层面性能的问题点，做出相关的优化措施。

通过 APM 对网络性能的监控，新闻客户端不断的采取了 HttpDNS，错误数据同步 CND 优化 CND 调度链等一些列措施，将网络错误率降最小化。

## AOP 与 APT 区别 ##
APT(Annotation Processing Tool)即注解处理器，是一种处理注解的工具，确切的说它是 javac 的一个工具，它用来在编译时扫描和处理注解。注解处理器以 Java 代码(或者编译过的字节码)作为输入，生成 .java 文件作为输出。

简单来说 APT 就是在编译期，通过注解生成 .java 文件，然后 .java 文件仍然需要进一步编译生成 .class 文件。而 AOP 是在编译完成后直接通过修改 .class 文件，添加或者修改代码逻辑。

## 致谢 ##
[AOP概念详解笔记](https://my.oschina.net/liuyuantao/blog/1439275)
[AOP技术在客户端的应用与实践](https://mp.weixin.qq.com/s/WRrpC30g4r_AyJa6BPNf8Q)

