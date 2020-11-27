---
title: Kotlin 1.4 的新特性
categories: Kotlin
tags:
  - Kotlin
abbrlink: 664dda02
date: 2020-09-03 12:36:22
---

## Kotlin 1.4 新特性 ##
Kotlin 1.4 新特性包含以下几个方面：
 - 语言特性与改进
  - Kotlin 接口的 SAM 转换
  - 面向库作者的显式 API 模式
  - 混用命名参数与位置参数
  - 尾随逗号
  - 可调用引用改进
  - 循环中的 when 内部可以 break 及 continue
 - IDE 中的新工具
  - 新的灵活项目向导
  - 协程调试器
 - 新的更强大的类型推断算法
  - 会自动推断类型的更多情况
  - lambda 表达式最后一个表达式的智能转换
  - 可调用引用的智能转换
  - 属性委托的更佳推导
  - 具有不同参数的 Java 接口的 SAM 转换
 - Gradle 项目改进
  - 现在默认添加了对标准库的依赖
  - Kotlin 项目需要最近版本的 Gradle
  - 改进了 IDE 对 Kotlin Gradle DSL 的支持

## 语言特性与改进 ##
### Kotlin 接口的 SAM 转换 ###
一个就是大家期待已久的 Kotlin 接口和函数的 `SAM(Single Abstract Method)` 转换。得益于新的类型推导算法，Kotlin 1.4 之前一直只有调用接收 Java 单一方法接口的 Java 的方法时才可以有 SAM 转换，现在，Kotlin 1.4 可以将 Kotlin 接口标记为功能接口，并通过添加 `fun` 关键字让它们以类似方式工作：

更多信息请参考 [函数式(SAM)接口](https://www.kotlincn.net/docs/reference/fun-interfaces.html)

### 面向库作者的显式 API 模式 ###
Kotlin 编译器为库作者提供了显式的 API 模式。在这种模式下，编译器会执行其他检查，以帮助使库的 API 更清晰，更一致。它为暴露于库的公共 API 的声明增加了以下要求：
 - 如果默认可见性将其公开给公共 API，则声明必须使用可见性修饰符。这有助于确保任何声明都不会无意间暴露给公共 API。
 - 对于公开 API 公开的属性和函数，需要明确的类型规范。这样可以保证 API 用户知道他们使用的 API 成员的类型。

根据配置，这些显式 API 可能会产生错误(严格模式)或警告(警告模式)。为了便于阅读和常识，某些类型的声明从此类检查中排除：
 - 主构造方法
 - 数据类的属性
 - 属性的 getters/setters
 - 被重写的方法

显式 API 模式仅分析模块的生产源。要以显式API模式编译模块，请在Gradle构建脚本中添加以下几行：
```gradle
kotlin {    
    // for strict mode
    explicitApi() 
    // or
    explicitApi = 'strict'
    
    // for warning mode
    explicitApiWarning()
    // or
    explicitApi = 'warning'
}
```

使用命令行编译时，通过添加 `-Xexplicit-api` 编译器选项(其值为 `strict` 或 `warning`)来切换到显式 API 模式。
```gradle
-Xexplicit-api={strict|warning}
```

有关显式 API 模式的更多详细信息，请参见 [Explicit API mode](https://github.com/Kotlin/KEEP/blob/master/proposals/explicit-api-mode.md)。

### 混用命名参数与位置参数 ###
在 Kotlin 1.3 中，当使用命名参数调用函数时，必须将所有不带名称的参数(位置参数)放在第一个命名参数之前。例如，可以调用 `f(1，y = 2)`，但不能调用 `f(x = 1，2)`。

在 Kotlin 1.4 中，没有这种限制。现在可以在一组位置参数的中间为参数指定名称。此外，还可以按自己喜欢的任何方式混合使用位置和命名参数，只要它们保持正确的顺序即可。这对于明确弄清布尔值或 `null` 值属于哪个属性特别有用。

### 尾随逗号 ###
使用 Kotlin 1.4，可以在枚举中添加尾随逗号，例如参数和参数列表，`when` 条目以及解构声明的组件。使用结尾逗号，可以添加新项并更改其顺序，而无需添加或删除逗号。

如果对参数或值使用多行语法，这将特别有用。 添加结尾逗号后，可以轻松地将行与参数或值交换。

### 在循环中的 when 内部使用 break 与 continue ###
在 Kotlin 1.3 中，不能循环中的 `when` 表达式内部使用非限定的 `break` 和 `continue`，原因是这些关键字是为 `when` 表达式中可能出现的失败行为保留的。如果要在循环中的 `when` 表达式内部使用 `break` 和 `continue` 的原因，则必须给它们加上标签。

在 Kotlin 1.4 中，当 `when` 表达式包含在循环中时，可以在没有标签的情况下使用 `break` 和 `continue`，它们通过终止最近的封闭循环或继续执行下一个步骤来执行预期的行为。

## 新的更强大的类型推断算法 ##
### 会自动推断类型的更多情况 ###
类型推导让 Kotlin 的语法获得了极大的简洁性。不过，大家在使用 Kotlin 开发时，一定会发现有些情况下明明类型是很确定的，编译器却一定要让我们显式的声明出来，这其实就是类型推导算法没有覆盖到的场景了。

在旧算法要求显式指定类型的许多情况下，新的类型推导算法可以推导出类型，能够应付更加复杂的推导场景。

### lambda 表达式最后一个表达式的智能转换 ###
在 Kotlin 1.3 中，lambda 中的最后一个表达式没有进行智能强制转换，除非指定了预期的类型。因此，在下面的例子中，Kotlin 1.3推断字符串?作为结果变量的类型:在 Kotlin 1.4 中，由于有了新的推理算法，lambda 内部的最后一个表达式进行了智能类型转换，这个新的、更精确的类型用于推断得到的 lambda 类型。

在 Kotlin 1.3 中，经常需要添加显式类型转换(或者 `!!`)或者类型强制转换(`as String`)，以使这些用例工作，现在这些强制转换已经没有必要了。

### 可调用引用的智能转换 ###
在 Kotlin 1.3 中，不能直接访问智能转换类型的成员引用。在 Kotlin 1.4 中，类型检查之后，可以直接访问对应于子类型的成员引用。

### 具有不同参数的 Java 接口的 SAM 转换 ###
Kotlin 从一开始就支持 Java 接口的 SAM 转换，但有一种情况不受支持，在使用现有 Java 库时，这有时很烦人。如果调用的 Java 方法采用两个 SAM 接口作为参数，那么两个参数都需要是 lambdas 或常规对象，不能将一个参数作为 lambda 传递，另一个参数作为对象传递。新的算法解决了这个问题，并且可以在任何情况下传递一个 lambda 而不是 SAM 接口。

## Gradle 项目改进 ##
### 默认添加了对标准库的依赖 ###
在任何 Kotlin Gradle 项目(包括多平台项目)中，您不再需要声明对 `stdlib` 库的依赖。默认情况下添加该依赖项。自动添加的标准库与 Kotlin Gradle 插件的版本相同，因为它们有相同的版本控制。

对于特定于平台的源集，使用库的相应特定于平台的变体，而其余部分则添加一个通用的标准库。 Kotlin Gradle 插件将根据 Gradle 构建脚本的 `kotlinOptions.jvmTarget` 编译器选项选择适当的 JVM 标准库。

### Kotlin 项目的最低 Gradle 版本 ###
要使用 Kotlin 项目中的新特性，需要将 Gradle 升级到最新版本。多平台项目需要 Gradle 6.0 或更高版本，而其他 Kotlin 项目使用 Gradle 5.4 或更高版本。

### 改进了 IDE 对 *.gradle.kts 的支持 ###
在 Kotlin 1.4 中，继续改进了对 Gradle Kotlin DSL脚本(`*.gradle.kts` 文件)的IDE支持。显式加载脚本配置以获得更好的性能。以前，您对构建脚本所做的更改将在后台自动加载。为了提高性能，在 Kotlin 1.4 中禁用了构建脚本配置的自动加载。现在只有在显式应用更改时，IDE 才加载更改。

在 Gradle 6.0 之前的版本中，需要通过在编辑器中单击 `Load Configuration` 手动加载脚本配置。在 Gradle 6.0 及以上版本中，可以通过点击 `Load Gradle Changes` 或通过重新导入 Gradle 项目来显式地应用变化。

在Gradle 6.0及更高版本的 IntelliJ IDEA 2020.1 中添加了另一项操作——`Load Script Configurations(加载脚本配置)`，该操作可加载对脚本配置的更改，而无需更新整个项目。这比重新导入整个项目所需的时间少得多。还应该为新创建的脚本加载脚本配置，或者在第一次使用新的 Kotlin 插件打开项目时加载脚本配置。

## 参考 ##
[What's New in Kotlin 1.4](https://kotlinlang.org/docs/reference/whatsnew14.html)

