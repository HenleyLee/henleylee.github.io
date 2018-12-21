---
title: Kotlin 1.2 的新特性
categories: Kotlin
tags:
  - Kotlin
abbrlink: 9e8f91b4
date: 2018-12-12 18:45:36
---

## Kotlin 1.2 新特性 ##
Kotlin 1.2 新特性包含以下几个方面：
 - 多平台项目
 - 其他语言特性
 - 标准库
 - JVM 后端
 - JavaScript 后端

## 多平台项目 ##
多平台项目是 Kotlin 1.2 中的一个新的**`实验性`**的特性，允许你在支持 Kotlin 的目标平台——JVM、JavaScript 以及(将来的) Native 之间重用代码。在多平台项目中，你有三种模块：
 - 一个公共模块包含平台无关代码，以及无实现的依赖平台的 API 声明。
 - 平台模块包含通用模块中的平台相关声明在指定平台的实现，以及其他平台相关代码。
 - 常规模块针对指定的平台，既可以是平台模块的依赖，也可以依赖平台模块。

当你为指定平台编译多平台项目时，既会生成公共代码也会生成平台相关代码。

多平台项目支持的一个主要特点是可以通过**预期声明与实际声明**来表达公共代码对平台相关部分的依赖关系。一个预期声明指定一个 API(类、接口、注解、顶层声明等)。一个实际声明要么是该 API 的平台相关实现，要么是一个引用到在一个外部库中该 API 的一个既有实现的别名。下面是一个示例：

在公共代码中：
```kotlin
// 预期平台相关 API:
expect fun hello(world: String): String
​
fun greet() {
    // 该预期 API 的用法：
    val greeting = hello("multi-platform world")
    println(greeting)
}
​
expect class URL(spec: String) {
    open fun getHost(): String
    open fun getPath(): String
}
```

在 JVM 平台代码中：
```kotlin
actual fun hello(world: String): String =
    "Hello, $world, on the JVM platform!"
​
// 使用既有平台相关实现：
actual typealias URL = java.net.URL
```

关于构建多平台项目的详细信息与步骤，请参考 [Multiplatform Programming](http://kotlinlang.org/docs/reference/multiplatform.html)。

## 其他语言特性 ##
### 注解中的数组字面值 ###
自 Kotlin 1.2 起，注解的数组参数可以使用新的数组常量语法传入，而无需使用 `arrayOf` 函数：
```kotlin
@CacheConfig(cacheNames = ["books", "default"])
public class BookRepositoryImpl {
    // ……
}
```

数组常量语法被限制为注释参数。

### lateinit 顶层属性与局部变量 ###
`lateinit` 修饰符现在可以用在顶级属性和局部变量上。例如，当一个 lambda 作为构造函数参数传递给一个对象时，后者可以用于引用另一个必须稍后定义的对象：
```kotlin
class Node<T>(val value: T, val next: () -> Node<T>)
​
fun main(args: Array<String>) {
    // 三个节点的环：
    lateinit var third: Node<Int>
​
    val second = Node(2, next = { third })
    val first = Node(1, next = { second })
​
    third = Node(3, next = { first })
​
    val nodes = generateSequence(first) { it.next() }
    println("Values in the cycle: ${nodes.take(7).joinToString { it.value.toString() }}, ...")
}
```

### 检查 lateinit 变量是否已初始化 ###
现在可以通过属性引用的 `isInitialized` 来检测该 `lateinit` 变量是否已初始化：
```kotlin
println("isInitialized before assignment: " + this::lateinitVar.isInitialized)
lateinitVar = "value"
println("isInitialized after assignment: " + this::lateinitVar.isInitialized)
```

### 内联函数带有默认函数式参数 ###
内联函数现在允许其内联函数参数具有默认值：
```kotlin
inline fun <E> Iterable<E>.strings(transform: (E) -> String = { it.toString() }) =
    map { transform(it) }
​
val defaultStrings = listOf(1, 2, 3).strings()
val customStrings = listOf(1, 2, 3).strings { "($it)" } 
```

### 源自显式类型转换的信息会用于类型推断 ###
Kotlin 编译器现在可以使用类型转换信息进行类型推断。如果调用一个返回类型参数 `T` 并将返回值转换为特定类型 `Foo` 的泛型方法，则编译器现在可以理解此调用的 `T` 需要绑定到 `Foo` 类型。

这对 Android 开发者来说尤其重要，因为编译器现在可以在 Android API level 26 中正确分析范型 `findViewById` 调用：
```kotlin
val button = findViewById(R.id.button) as Button
```

### 智能类型转换改进 ###
当一个变量从一个安全调用表达式中被赋值并且被检查为 null 时，`智能转换`也被应用到安全调用接收器中：
```kotlin
val firstChar = (s as? CharSequence)?.firstOrNull()
if (firstChar != null)
return s.count { it == firstChar } // s: Any 会智能转换为 CharSequence
​
val firstItem = (s as? Iterable<*>)?.firstOrNull()
if (firstItem != null)
return s.count { it == firstItem } // s: Any 会智能转换为 Iterable<*>
```

智能转换现在也允许用于在 lambda 表达式中局部变量，只要这些局部变量仅在 lambda 表达式之前修改即可：
```kotlin
val flag = args.size == 0
var x: String? = null
if (flag) x = "Yahoo!"

run {
    if (x != null) {
        println(x.length) // x 会智能转换为 String
    }
}
```

### 支持 ::foo 作为 this::foo 的简写 ###
现在绑定到 `this` 成员的可调用引用可以无需显式接收者，可以使用 `::foo` 替代 `this::foo`，写入一个绑定的可调用的引用，而不用明确的接收器。这也使得可调用的引用在你引用外部接收者的成员的 lambda 中更方便使用。

### 阻断性变更：try 块后可靠智能转换 ###
Kotlin 以前将 `try` 块中的赋值语句用于块后的智能转换，这可能会破坏类型安全与空安全并导致运行时失败。这个版本修复了此问题，使智能转换更加严格，但可能会破坏一些依靠这种智能转换的代码。

如果要切换到旧版智能转换行为，请传入回退标志 `-Xlegacy-smart-cast-after-try` 作为编译器参数，该参数会在 Kotlin 1.3中弃用。

### 弃用：数据类弃用 copy ###
当从已经具有相同签名的 `copy` 函数的类型派生数据类时，为数据类生成的 `copy` 实现使用父类型的默认函数，会导致出现与预期相反的行为，如果父类型没有默认参数，则在运行时失败

导致 `copy` 冲突的继承在 Kotlin 1.2 中已弃用并带有警告，而在 Kotlin 1.3中将会是错误。

### 弃用：枚举项中的嵌套类型 ###
在枚举项中，由于初始化逻辑中的问题，定义一个非 `inner class` 的嵌套类型的功能已经被弃用。在 Kotlin 1.2 中这将会引起警告，并将在 Kotlin 1.3 中报错。

### 弃用：vararg 中的单命名参数 ###
为了与注解中的数组常量保持一致，在命名的表单(`foo(items = i)`) 中为 `vararg` 参数传递的单个项目已被弃用。请使用具有相应数组工厂函数的展开运算符：
```kotlin
foo(items = *intArrayOf(1))
```

在这种情况下，有一种优化可以消除冗余数组的创建，从而防止性能下降。单一参数的表单在 Kotlin 1.2 中会引起警告，并将在 Kotlin 1.3 中被移除。

### 弃用：扩展 Throwable 的泛型类的内部类 ###
继承自 `Throwable` 的泛型的内部类可能会违反 throw-catch 场景中的类型安全性，因此已被弃用，在 Kotlin 1.2 中会被警告，在 Kotlin 1.3 中将会报错。

### 弃用：修改只读属性的幕后字段 ###
在自定义 getter 中通过赋值 `field = ...` 来改变只读属性的幕后字段字段已被弃用，在 Kotlin 1.2 中会被警告，在 Kotlin 1.3 中将会报错。

## 标准库
### Kotlin 标准库构件与拆分包 ###
Kotlin 标准库现在完全兼容 Java 9 的模块系统，它会禁止对包进行拆分(多个 jar 文件声明的类在同一包中)。为了支持这一点，引入了新的构件 `kotlin-stdlib-jdk7` 和 `kotlin-stdlib-jdk8`，取代了旧版的 `kotlin-stdlib-jre7` 和 `kotlin-stdlib-jre8`。

新构件中的声明从 Kotlin 的角度来看在相同的包名下是可见的，但是对 Java 而言它们有不同的包名。因此，切换到新的构件不需要对源代码进行任何更改。

确保与新的模块系统兼容的另一个更改是从 `kotlin-reflect` 库中移除了 `kotlin.reflect` 包中的弃用声明。如果正在使用它们，则需要使用 `kotlin.reflect.full` 包中的声明，自 Kotlin 1.1 起就支持这个包了。

### windowed、chunked、zipWithNext ###
用于 `Iterable<T>`、`Sequence<T>` 与 `CharSequence` 的新的扩展包含了诸如：缓冲或批处理(`chunked`)、滑动窗口和计算滑动平均值(`windowed`)以及处理成对的后续条目(`zipWithNext`)等用例：
```kotlin
val items = (1..9).map { it * it }

val chunkedIntoLists = items.chunked(4)
val points3d = items.chunked(3) { (x, y, z) -> Triple(x, y, z) }
val windowed = items.windowed(4)
val slidingAverage = items.windowed(4) { it.average() }
val pairwiseDifferences = items.zipWithNext { a, b -> b - a }
```
​
### fill、replaceAll、shuffle/shuffled ###
添加了一系列扩展函数用于处理列表：针对 `MutableList` 的 `fill`、`replaceAll` 和 `shuffle` ，以及针对只读 `List` 的 `shuffled`：
```kotlin
val items = (1..5).toMutableList()

items.shuffle()
println("Shuffled items: $items")

items.replaceAll { it * 2 }
println("Items doubled: $items")

items.fill(5)
println("Items filled with 5: $items")
```

### kotlin-stdlib 中的数学运算 ###
为满足用户长期以来的需求，Kotlin 1.2 中增加了用于数学运算的 `kotlin.math` API，也是 JVM 和 JS 的通用 API，包含以下内容：
 - 常量：`PI` 与 `E`；
 - 三角函数：`cos`、`sin`、`tan` 及其反函数：`acos`、`asin`、`atan`、`atan2`；
 - 双曲函数：`cosh`、`sinh`、`tanh` 及其反函数：`acosh`、`asinh`、`atanh`
 - 指数函数：`pow`(扩展函数)、`sqrt`、`hypot`、`exp`、`expm1`；
 - 对数函数：`log`、`log2`、`log10`、`ln`、`ln1p`；
 - 取整函数:
   - `ceil`、`floor`、`truncate`、`round`(奇进偶舍)函数；
   - `roundToInt`、`roundToLong`(四舍五入)扩展函数；
 - 符号函数与绝对值：
   - `abs` 与 `sign` 函数；
   - `absoluteValue` 与 `sign` 扩展属性；
   - `withSign` 扩展函数；
 - 两个数的最值函数：`max` 与 `min`；
 - 二进制表示：
   - `ulp` 扩展属性；
   - `nextUp`、`nextDown`、`nextTowards` 扩展函数；
   - `toBits`、`toRawBits`、`Double.fromBits`(这些在 kotlin 包中)。

同系列(但不包括常量)的函数也针对 `Float` 型参数提供了。

### 用于 BigInteger 与 BigDecimal 的操作符与转换 ###
Kotlin 1.2 引入了一组用于操作 `BigInteger` 和 `BigDecimal` 以及使用从其他数字类型进行转换的函数。具体如下：
 - `toBigInteger` 用于 `Int` 与 `Long`；
 - `toBigDecimal` 用于 `Int`、`Long`、`Float`、`Double` 以及 `BigInteger`；
 - 算术与位运算操作符函数：
   - 二元操作符 `+`、`-`、`*`、`/`、`%` 以及中缀函数 `and`、`or`、`xor`、`shl`、`shr`；
   - 一元操作符 `-`、`++`、`--` 以及函数 `inv`。

### 浮点数到位的转换 ###
添加了用于将 `Double` 及 `Float` 与其位表示形式相互转换的函数：
 - `toBits` 与 `toRawBits` 对于 `Double` 返回 `Long` 而对于 `Float` 返回 `Int`；
 - `Double.fromBits` 与 `Float.fromBits` 用于从位表示形式中转换为浮点数。

### 正则表达式现在可序列化 ###
`kotlin.text.Regex` 类现在已经是 `Serializable` 的了，并且可以在可序列化的层次结构中使用。

### 如果满足条件，Closeable.use 可以调用 Throwable.addSuppressed ###
当在其他异常处理后，关闭资源期间抛出异常时，`Closeable.use` 函数可调用 `Throwable.addSuppressed`。

要启用这个行为，需要在依赖关系中包含 `kotlin-stdlib-jdk7`。

## JVM 后端 ##
### 构造函数调用规范化 ###
自 1.0 以来，Kotlin 就开始支持复杂控制流的表达式，例如 try-catch 表达式和内联函数调用。根据 Java 虚拟机规范这样的代码是合法的。不幸的是，当构造函数调用的参数中存在这样的表达式时，一些字节码处理工具不能很好地处理这些代码。

为了减少使用此类字节码处理工具的用户的这个问题，我们添加了一个命令行选项(`-Xnormalize-constructor-calls=MODE`)，它会告诉编译器为这样的结构生成更多的类 Java 字节码。这里 `MODE` 的值是以下之一：
 - `disable`(默认值)——以和 Kotlin 1.0 和 1.1 相同的方式生成字节码；
 - `enable`——为构造函数调用生成类似 Java 的字节码，这可以改变类加载和初始化的顺序；
 - `preserve-class-initialization`——为构造函数调用生成类似 Java 的字节码，并确保保持类初始化顺序。这可能会影响应用程序的整体性能；仅用在多个类之间共享一些复杂状态并在类初始化时更新的场景中。

“手工”的解决方法是将控制流的子表达式的值存储在变量中，而不是直接在调用参数中对它们进行求值。它类似于 `-Xnormalize-constructor-calls=enable`。

### Java 默认方法调用 ###
在 Kotlin 1.2 之前，接口成员在使用 JVM 1.6 的情况下重写 Java 默认方法会在父调用中产生警告：`Super calls to Java default methods are deprecated in JVM target 1.6. Recompile with '-jvm-target 1.8'`。在 Kotlin 1.2 中，这将会报错，因此需要使用 JVM 1.8 来编译这些代码。


### 阻断性变更：平台类型 x.equals(null) 的一致行为 ###
在映射到 Java 原生类型(`Int!`、`Boolean!`、`Short!`、`Long!`、`Float!`、`Double!`、`Char!`)的平台类型上调用 `x.equals(null)`，当 `x` 为 `null` 时错误地返回了 `true`。自 Kotlin 1.2 起，在平台类型的空值上调用 `x.equals(……)` 都会**抛出 NPE**(但 `x == ...` 不会)。

要返回到 1.2 之前的行为，请将标志 `-Xno-exception-on-explicit-equals-for-boxed-null` 传给编译器。

### 阻断性变更：通过内联的扩展接收器修复平台的 null 转义 ###
在平台类型空值上调用的内联扩展函数并没有检查接收器是否为 null，并因此允许 null 转义到其他代码中。Kotlin 1.2 在调用点强制执行此检查，如果接收方为空，则抛出异常。

要切换到旧版行为，请将回退标志 `-Xno-receiver-assertions` 传给编译器。

## JavaScript 后端
### 默认启用对类型化数组(TypedArrays)的支持 ###
JS typed arrays 支持将 Kotlin 原生数组(如 `IntArray`、`DoubleArray` 等)转换为 [JavaScript 的类型数组](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays)，以前这是可选功能，现在默认情况下已启用。

## 工具 ##
### 将警告视为错误 ###
编译器现在提供了一个将所有警告视为错误的选项。在命令行中使用 `-Werror`，或使用以下的 Gradle 代码：
```gradle
compileKotlin {
    kotlinOptions.allWarningsAsErrors = true
}
```

## 参考 ##
[What's New in Kotlin 1.2](https://kotlinlang.org/docs/reference/whatsnew12.html)

