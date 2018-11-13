---
title: Kotlin 简介与入门指南
date: 2018-11-08 18:36:32
categories: Kotlin
tags:
  - Kotlin
---

## Kotlin 是什么？ ##
Kotlin 是 JetBrains 开发的针对 JVM、Android 和浏览器的静态编程语言。

JetBrains，作为目前广受欢迎的 Java IDE IntelliJ IDEA 的开发商，在 Apache 许可下已经开源其 Kotlin 编程语言。JetBrains 作为最智能的 Java IDE 的开发商，对 Java 的了解是毋庸置疑的，在使用 Java 过程中，JetBrains 的工程师们发现了大量的问题，为了更高效的开发以及解决 Java 中的一些问题，JetBrains 开发了致力于替代 Java 的 Kotlin。

Kotlin 可以编译成Java字节码，也可以编译成 JavaScript，方便在没有 JVM 的设备上运行。

在2017年的 Google I/O 中，Google 宣布 Kotlin 成为 Android 官方开发语言。

Kotlin 被称之为 Android 世界的 Swift。

## 为什么选择 Kotlin？ ##
使用 Kotlin 开发 Android 的好处太多了，它简单、易用、代码量少。有人说代码量减少3倍，别不信，这并不夸张，当代码量越大的时候我们就越会发现这一点。事实上，Kotlin 生来就是为了弥补 Java 缺失的现代语言的特性的，并极大的简化了代码，使得开发者可以编写尽量少的样板代码。

### 简洁——大大减少样板代码的数量 ###
使用一行代码创建一个包含 getters、 setters、 equals()、 hashCode()、 toString() 以及 copy() 的 POJO：
```kotlin
data class Customer(val name: String, val email: String, val company: String)
```

或者使用 lambda 表达式来过滤列表：
```kotlin
val positiveNumbers = list.filter { it > 0 }
```

想要单例？创建一个 object 就可以了：
```kotlin
object ThisIsASingleton {
    val companyName: String = "JetBrains"
}
```

### 安全——避免空指针异常等整个类的错误 ###
彻底告别那些烦人的 NullPointerException——著名的[十亿美金的错误](http://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)：
```kotlin
var output: String
output = null   // 编译错误
```

Kotlin 可以保护你避免对可空类型的误操作：
```kotlin
val name: String? = null    // 可控类型
println(name.length())      // 编译错误
```

并且如果你检查类型是正确的，编译器会为你做自动类型转换：
```kotlin
fun calculateTotal(obj: Any) {
    if (obj is Invoice)
        obj.calculateTotal()
}
```

### 多用途——支持多中类型的应用程序 ###
1. Android开发：没有性能影响。运行时非常小。
2. 服务器应用。100％兼容所有JVM框架。
3. JavaScript。在Kotlin中编写代码，并转换为 JavaScrip 在 Node.js 或浏览器中运行。
4. 企业。使用Kotlin进行任何类型的企业Java EE开发。
5. 网页。无论您是要强制类型的HTML，CSS构建器还是简单的Web开发。
6. 其他所有(iOS、嵌入式等等)：Kotlin/Native 在2017年4月份推出了预览版，并在在官方博客中描述了对 Kotlin/Native 的美好愿景。

### 互操作性——充分利用 JVM、Android 和浏览器的现有库 ###
使用 JVM 上的任何现有库，因为有 100％ 的兼容性，包括 SAM 支持。
```kotlin
import io.reactivex.Flowable
import io.reactivex.schedulers.Schedulers

Flowable
    .fromCallable {
        Thread.sleep(1000) // 模仿高开销的计算
        "Done"
    }
    .subscribeOn(Schedulers.io())
    .observeOn(Schedulers.single())
    .subscribe(::println, Throwable::printStackTrace)
```

无论是 JVM 还是 JavaScript 目标平台，都可用 Kotlin 写代码然后部署到你想要的地方
```kotlin
import kotlin.browser.window

fun onLoad() {
    window.document.body!!.innerHTML += "<br/>Hello, Kotlin!"
}
```

Kotlin 和 Java 都属于基于 JVM 的编程语言，Kotlin 和 Java 的交互性很好，可以说是无缝连接，这表现在：
 - Kotlin 可以自由的引用 Java 的代码，反之亦然
 - Kotlin 可以使用现有的全部的 Java 框架和库
 - Java 文件可以很轻松的借助 IntelliJ 的插件转成 Kotlin

### 工具友好——自由选择命令行编译器或一级 IDE 支持 ###
一门语言需要工具化，而在 JetBrains，这正是我们做得最好的地方！Kotlin目前提供了四种编写方式：
 - [命令行编译工具](http://kotlinlang.org/docs/tutorials/command-line.html)
 - 在线编辑 [Try Kotlin](https://try.kotlinlang.org/) 和 [Try Online](https://play.kotlinlang.org/)
 - [Android Studio](https://developer.android.com/studio/) 和 [Eclipse](https://www.eclipse.org/downloads/)
 - [IntelliJ IDEA](https://www.jetbrains.com/idea/download/)

> 其中 [`IntelliJ IDEA`](https://www.jetbrains.com/idea/download/) 提供了对 Kotlin 最新功能的支持，也是 Kotlin 最智能的编辑器。

## Kotlin 相关资源 ##
 - [Kotlin 官网](https://kotlinlang.org/)
 - [Kotlin 中文站](https://www.kotlincn.net/)
 - [Kotlin 官方博客](https://blog.jetbrains.com/kotlin/)
 - [Kotlin 中文博客](https://www.kotliner.cn/)
 - [Kotlin 官方论坛](https://discuss.kotlinlang.org/)
 - [Kotlin 中文论坛](https://discuss.kotliner.cn/)
 - [Kotlin on GitHub](https://github.com/JetBrains/kotlin)
