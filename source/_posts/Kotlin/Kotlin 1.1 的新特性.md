---
title: Kotlin 1.1 的新特性
date: 2018-12-10 18:36:25
categories: Kotlin
tags:
  - Kotlin
---

## Kotlin 1.1 新特性 ##
Kotlin 1.1 新特性包含以下几个方面：
 - 协程
 - 其它语言特性
 - 标准库
 - JVM 后端
 - JavaScript 后端

## 协程 ##
Kotlin 1.1 的关键新特性就是**`协程`**，它带来了 `async/wait`、`yield` 以及类似的编程模式的支持。Kotlin 设计的关键特性是所有协程都是由库实现的，而不是语言。所以开发者不需要与任何特定的编程范例或者并行库进行绑定。

协程是一个高效且轻量级的线程，可以挂起并稍后恢复执行。协程是又挂起函数支持的：调用这个函数可能会挂起一个协程，并开启一个新的协程，大多数情况下采用匿名挂起函数(也就是可挂起 lambda 表达式)。

让我们看一下在 [kotlinx.coroutines](https://github.com/kotlin/kotlinx.coroutines) 库中实现的 `async/await`:
```kotlin
// 在后台线程池中运行该代码
fun asyncOverlay() = async(CommonPool) {
    // 启动两个异步操作
    val original = asyncLoadImage("original")
    val overlay = asyncLoadImage("overlay")
    // 然后应用叠加到两个结果
    applyOverlay(original.await(), overlay.await())
}

// 在 UI 上下文中启动新的协程
launch(UI) {
    // 等待异步叠加完成
    val image = asyncOverlay().await()
    // 然后在 UI 中显示
    showImage(image)
}
```
`async { …… }` 启动一个协程，当使用 `await()` 时，协程被挂起，而执行正在等待的操作，并且在等待的操作完成时恢复执行(可能在不同的线程上)。

标准库通过 `yield` 和 `yieldAll` 函数使用协程来支持 `lazily generated sequences` 惰性生成序列。在这样的序列中，在取回每个元素之后挂起返回序列元素的代码块， 并在请求下一个元素时恢复。代码示例如下：
```kotlin
val seq = buildSequence {
    for (i in 1..5) {
        // 产生一个 i 的平方
        yield(i * i)
    }
    // 产生一个区间
    yieldAll(26..28)
}

// 输出该序列
println(seq.toList())
```

更多信息请参考 [Coroutines](https://kotlinlang.org/docs/reference/coroutines.html) 和 [Tutorials](https://kotlinlang.org/docs/tutorials/coroutines-basic-jvm.html)。

> 请注意，协程目前还是一个**`实验性的特性`**，这意味着 Kotlin 团队不承诺在最终的 1.1 版本时保持该功能的向后兼容性。

## 其它语言特性 ##
### 类型别名 ###
类型别名允许给现有的类型定义别名。主要在泛型类型（如集合）以及函数类型中很常用。代码示例如下：
```kotlin
typealias OscarWinners = Map<String, String>

fun countLaLaLand(oscarWinners: OscarWinners) =
        oscarWinners.count { it.value.contains("La La Land") }

// 请注意，类型名称（初始名和类型别名）是可互换的：
fun checkLaLaLandIsTheBestMovie(oscarWinners: Map<String, String>) =
        oscarWinners["Best picture"] == "La La Land"
```

更多信息请参考 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/type-aliases.md) 或 [Type aliases](https://kotlinlang.org/docs/reference/type-aliases.html)。

### 已绑定的可调用引用 ###
使用 `::` 操作符可以获取一个指向特定对象实例的方法或属性的成员引用。之前这只能用在 lambda 表达式上。代码示例如下：
```kotlin
val numberRegex = "\\d+".toRegex()
val numbers = listOf("abc", "123", "456").filter(numberRegex::matches)
```

更多信息请参考 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/bound-callable-references.md) 或 [Bound Function and Property References](https://kotlinlang.org/docs/reference/reflection.html#bound-function-and-property-references-since-11)。

### 密封类和数据类 ###
Kotlin 1.1 移除了一些对 Kotlin 1.0 中已存在的密封类和数据类的限制。现在可以在同一个文件中的任何地方定义一个密封类的子类，而不只是以作为密封类嵌套类的方式。数据类现在可以扩展其它类，这样可以更优雅简洁的定义表达式类的层次结构。代码示例如下：
```kotlin
sealed class Expr

data class Const(val number: Double) : Expr()
data class Sum(val e1: Expr, val e2: Expr) : Expr()
object NotANumber : Expr()

fun eval(expr: Expr): Double = when (expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
}
val e = eval(Sum(Const(1.0), Const(2.0)))
```

密封类更多信息请参考 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/sealed-class-inheritance.md)、[Sealed Classes](https://kotlinlang.org/docs/reference/sealed-classes.html#relaxed-rules-for-sealed-classes-since-11)。
数据类更多信息请参考 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/data-class-inheritance.md) 或 [Data Classes](https://kotlinlang.org/docs/reference/data-classes.html)。

### lambda 表达式中的解构 ###
现在可以使用 [解构声明](https://kotlinlang.org/docs/reference/multi-declarations.html) 语法取出 lambda 中的参数。代码示例如下：
```kotlin
val map = mapOf(1 to "one", 2 to "two")
// 之前
println(map.mapValues { entry ->
    val (key, value) = entry
  "$key -> $value!"
})
// 现在
println(map.mapValues { (key, value) -> "$key -> $value!" })
```

更多信息请参考 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/destructuring-in-parameters.md) 或 [Destructuring in Lambdas](https://kotlinlang.org/docs/reference/multi-declarations.html#destructuring-in-lambdas-since-11)。

### 用下划线表示未使用的参数 ###
对于具有多个参数的 lambda 表达式，可以使用 `_` 字符替换不使用的参数的名称。代码示例如下：
```kotlin
map.forEach { _, value -> println("$value!") }
```

这也适用于 [解构声明](https://kotlinlang.org/docs/reference/multi-declarations.html)。代码示例如下：
```kotlin
val (_, status) = getResult()
```

更多信息请参考 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/underscore-for-unused-parameters.md)。

### 数字字面值中的下划线 ###
正如在 Java 8 中一样，Kotlin 现在支持在数字字面值中使用下划线来划分组。代码示例如下：
```kotlin
val oneMillion = 1_000_000
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
```

更多信息请参考 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/underscores-in-numeric-literals.md)。

### 属性简写 ###
对于没有自定义访问器、或者将 getter 定义为表达式主体的属性，现在可以省略属性的类型。代码示例如下：
```kotlin
data class Person(val name: String, val age: Int) {
    val isAdult get() = age >= 20 // 属性类型推断为 “Boolean”
}
```

### 内联属性访问器 ###
如果属性没有幕后字段，可以使用 inline 修饰符来标记该属性访问器。这些属性访问器的编译方式与[内联函数](http://kotlinlang.org/docs/reference/inline-functions.html)相同。
```kotlin
public val <T> List<T>.lastIndex: Int
    inline get() = this.size - 1
```

更多信息请参考 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/inline-properties.md) 或 [Inline properties](http://kotlinlang.org/docs/reference/inline-functions.html#inline-properties-since-11)。

### 局部委托属性 ###
现在可以对局部变量使用[委托属性](http://kotlinlang.org/docs/reference/delegated-properties.html)语法。一个应用场景就是定义一个延迟求值的局部变量。代码示例如下：
```kotlin
val answer by lazy {
    println("Calculating the answer...")
    42
}
if (needAnswer()) {                     // 返回随机值
    println("The answer is $answer.")   // 此时计算出答案
}
else {
    println("Sometimes no answer is the answer...")
}
```

更多信息请参考 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/local-delegated-properties.md) 或 [Local Delegated Properties](http://kotlinlang.org/docs/reference/delegated-properties.html#local-delegated-properties-since-11)。

### 拦截委托属性的绑定 ###
对于[委托属性](http://kotlinlang.org/docs/reference/delegated-properties.html)，现在可以使用 `provideDelegate` 操作符拦截委托到属性之间的绑定。例如，如果我们想要在绑定之前检查属性名称，我们可以这样写：
```kotlin
class ResourceLoader<T>(id: ResourceID<T>) {
    operator fun provideDelegate(thisRef: MyUI, property: KProperty<*>): ReadOnlyProperty<MyUI, T> {
        checkProperty(thisRef, property.name)
        …… // 属性创建
    }

    private fun checkProperty(thisRef: MyUI, name: String) { …… }
}

fun <T> bindResource(id: ResourceID<T>): ResourceLoader<T> { …… }

class MyUI {
    val image by bindResource(ResourceID.image_id)
    val text by bindResource(ResourceID.text_id)
}
```

`provideDelegate` 方法可以可以在创建 MyUI 实例的每个属性时调用，并做相应的验证检查。

更多信息请参考 [Providing a delegate](http://kotlinlang.org/docs/reference/delegated-properties.html#providing-a-delegate-since-11)。

### 泛型枚举值的访问 ###
现在可以用泛型的方式来对枚举类的值进行枚举。代码示例如下：
```kotlin
enum class RGB { RED, GREEN, BLUE }

inline fun <reified T : Enum<T>> printAllValues() {
    print(enumValues<T>().joinToString { it.name })
}
```

### 对于 DSL 中隐式接收者的作用域控制 ###
`@DslMarker` 注解允许限制来自 DSL 上下文中的外部作用域的接收者的使用。考虑那个典型的 [HTML 构建器示例](http://kotlinlang.org/docs/reference/type-safe-builders.html)：
```kotlin
table {
    tr {
        td { + "Text" }
    }
}
```

在 Kotlin 1.0 中，传递给 `td` 的 lambda 表达式中的代码可以访问三个隐式接收者：传递给 `table`、`tr` 和 `td` 的。这允许调用在上下文中没有意义的方法——例如在 `td` 里面调用 `tr`，从而在 `<td>` 中放置一个 `<tr>` 标签。

在 Kotlin 1.1 中，可以限制这种情况，以使只有在 `td` 的隐式接收者上定义的方法会在传给 `td` 的 lambda 表达式中可用。可以通过定义标记有 `@DslMarker` 元注解的注解并将其应用于标记类的基类。

### rem 操作符 ###
`mod` 操作符现已弃用，而使用 `rem` 取代。动机参见这个 [issue](https://youtrack.jetbrains.com/issue/KT-14650)。

## 标准库 ##
### 字符串到数字的转换 ###
在 String 类中有一些新的扩展，用来将它转换为数字，而不会在无效数字上抛出异常： `String.toIntOrNull(): Int?`、 `String.toDoubleOrNull(): Double?` 等。
```kotlin
val port = System.getenv("PORT")?.toIntOrNull() ?: 80
```

还有整数转换函数，如 `Int.toString()`、`String.toInt()`、`String.toIntOrNull()`，每个都有一个带有 `radix` 参数的重载，它允许指定转换的基数(2~36)。

### onEach() ###
`onEach` 是一个小、但对于集合和序列很有用的扩展函数，它允许对操作链中的集合/序列的每个元素执行一些操作，可能带有副作用。对于迭代其行为像 `forEach` 但是也进一步返回可迭代实例。对于序列它返回一个包装序列，它在元素迭代时延迟应用给定的动作。
```kotlin
inputDir.walk()
        .filter { it.isFile && it.name.endsWith(".txt") }
        .onEach { println("Moving $it to $outputDir") }
        .forEach { moveFile(it, File(outputDir, it.toRelativeString(inputDir))) }
```

### also()、takeIf() 和 takeUnless() ###
`also()`、`takeIf()` 和 `takeUnless()` 是适用于任何接收者的三个通用扩展函数。

`also` 就像 `apply`：它接受接收者、做一些动作、并返回该接收者。二者区别是在 `apply` 内部的代码块中接收者是 `this`， 而在 `also` 内部的代码块中是 `it` (并且如果你想的话，你可以给它另一个名字)。当你不想掩盖来自外部作用域的 `this` 时这很方便：
```kotlin
fun Block.copy() = Block().also {
    it.content = this.content
}
```

`takeIf` 就像单个值的 `filter`。它检查接收者是否满足该谓词，并在满足时返回该接收者否则不满足时返回 `null`。结合 elvis-操作符和及早返回，它允许编写如下结构：
```kotlin
val outDirFile = File(outputDir.path).takeIf { it.exists() } ?: return false
// 对现有的 outDirFile 做些事情
```

```kotlin
val index = input.indexOf(keyword).takeIf { it >= 0 } ?: error("keyword not found")
// 对输入字符串中的关键字索引做些事情，鉴于它已找到
```

`takeUnless` 与 `takeIf` 相同，只是它采用了反向谓词。当它不满足谓词时返回接收者，否则返回 `null`。因此，上面的示例之一可以用 `takeUnless` 重写如下：
```kotlin
val index = input.indexOf(keyword).takeUnless { it < 0 } ?: error("keyword not found")
```

当你有一个可调用的引用而不是 lambda 时，使用也很方便：
```kotlin
val result = string.takeUnless(String::isEmpty)
```

### groupingBy() ###
此 API 可以用于按照键对集合进行分组，并同时折叠每个组。例如，它可以用于计算文本中字符的频率：
```kotlin
val frequencies = words.groupingBy { it.first() }.eachCount()
```

### Map.toMap() 和 Map.toMutableMap() ###
`Map.toMap()` 和 `Map.toMutableMap()` 函数可以用来简易复制映射：
```kotlin
class ImmutablePropertyBag(map: Map<String, Any>) {
    private val mapCopy = map.toMap()
}
```

### Map.minus(key) ###
运算符 `plus` 提供了一种将键值对添加到只读映射中以生成新映射的方法，但是没有一种简单的方法来做相反的操作：从映射中删除一个键采用不那么直接的方式如 `Map.filter()` 或 `Map.filterKeys()`。现在运算符 `minus` 填补了这个空白。有 4 个可用的重载：用于删除单个键、键的集合、键的序列和键的数组。
```kotlin
val map = mapOf("key" to 42)
val emptyMap = map - "key"
```

### Map.getValue() ###
`Map` 上的这个扩展函数返回一个与给定键相对应的现有值，或者抛出一个异常，提示找不到该键。如果该映射是用 `withDefault` 生成的，这个函数将返回默认值，而不是抛异常。
```kotlin​
val map = mapOf("key" to 42)
// 返回不可空 Int 值 42
val value: Int = map.getValue("key")
​
val mapWithDefault = map.withDefault { k -> k.length }
// 返回 4
val value2 = mapWithDefault.getValue("key2")
​
// map.getValue("anotherKey") // <- 这将抛出 NoSuchElementException
```

### minOf() 和 maxOf() ###
`minOf()` 和 `maxOf()` 函数可用于查找两个或三个给定值中的最小和最大值，其中值是原生数字或 `Comparable` 对象。每个函数还有一个重载，它接受一个额外的 `Comparator` 实例，如果你想比较自身不可比的对象的话。
```kotlin
val list1 = listOf("a", "b")
val list2 = listOf("x", "y", "z")
val minSize = minOf(list1.size, list2.size)
val longestList = maxOf(list1, list2, compareBy { it.size })
```

### 类似数组的列表实例化函数 ###
类似于 `Array` 构造函数，现在有创建 `List` 和 `MutableList` 实例的函数，并通过调用 lambda 表达式来初始化每个元素：
```kotlin
val squares = List(10) { index -> index * index }
val mutable = MutableList(10) { 0 }
```

### 抽象集合 ###
这些抽象类可以在实现 Kotlin 集合类时用作基类。对于实现只读集合，有 `AbstractCollection`、`AbstractList`、`AbstractSet` 和 `AbstractMap`，而对于可变集合，有 `AbstractMutableCollection`、`AbstractMutableList`、`AbstractMutableSet` 和 `AbstractMutableMap`。在 JVM 上，这些抽象可变集合从 JDK 的抽象集合继承了大部分的功能。

### 数组处理函数 ###
标准库现在提供了一组用于逐个元素操作数组的函数：比较(`contentEquals` 和 `contentDeepEquals`)、哈希码计算(`contentHashCode` 和 `contentDeepHashCode`)、以及转换成一个字符串(`contentToString` 和 `contentDeepToString`)。它们都支持 JVM(它们作为 `java.util.Arrays` 中的相应函数的别名)和 JS(在 Kotlin 标准库中提供实现)。
```kotlin
val array = arrayOf("a", "b", "c")
println(array.toString())  // JVM 实现：类型及哈希乱码
println(array.contentToString())  // 良好格式化为列表
```

## JVM 后端 ##
### Java 8 字节码支持 ###
Kotlin 现在可以选择生成 Java 8 字节码(命令行选项 `-jvm-target 1.8` 或者 Ant/Maven/Gradle 中的相应选项)。目前这并不改变字节码的语义(特别是，接口和 lambda 表达式中的默认方法的生成与 Kotlin 1.0 中完全一样)，但我们计划在以后进一步使用它。

### Java 8 标准库支持 ###
现在有支持在 Java 7 和 8 中新添加的 JDK API 的标准库的独立版本。如果你需要访问新的 API，请使用 `kotlin-stdlib-jre7` 和 `kotlin-stdlib-jre8` Maven 构件，而不是标准的 `kotlin-stdlib`。这些构件是在 `kotlin-stdlib` 之上的微小扩展，它们将它作为传递依赖项带到项目中。

### 字节码中的参数名 ###
Kotlin 现在支持在字节码中存储参数名。这可以使用命令行选项 `-java-parameters` 启用。

### 常量内联 ###
编译器现在将 `const val` 属性的值内联到使用它们的位置。

### 可变闭包变量 ###
用于在 lambda 表达式中捕获可变闭包变量的装箱类不再具有 `volatile` 字段。此更改提高了性能，但在一些罕见的使用情况下可能导致新的竞争条件。如果受此影响，你需要提供自己的同步机制来访问变量。

### javax.scripting 支持 ###
Kotlin 现在与 [`javax.script`](https://docs.oracle.com/javase/8/docs/api/javax/script/package-summary.html) API(JSR-223)集成。其 API 允许在运行时求值代码段：
```kotlin
val engine = ScriptEngineManager().getEngineByExtension("kts")!!
engine.eval("val x = 3")
println(engine.eval("x + 2"))  // 输出 5
```

关于使用 API 的示例项目参见 [kotlin-jsr223-local-example](https://github.com/JetBrains/kotlin/tree/master/libraries/examples/kotlin-jsr223-local-example) 。

### kotlin.reflect.full ###
[为 Java 9 支持准备](https://blog.jetbrains.com/kotlin/2017/01/kotlin-1-1-whats-coming-in-the-standard-library/)，在 `kotlin-reflect.jar` 库中的扩展函数和属性已移动到 `kotlin.reflect.full` 包中。旧包(`kotlin.reflect`)中的名称已弃用，将在 Kotlin 1.2 中删除。请注意，核心反射接口(如 `KClass`)是 Kotlin 标准库(而不是 `kotlin-reflect`)的一部分，不受移动影响。

## JavaScript 后端 ##
从 Kotlin 1.1 开始，JavaScript 目标平台不再当是实验性的。所有语言功能都支持， 并且有许多新的工具用于与前端开发环境集成。

### 统一的标准库 ###
Kotlin 标准库的大部分目前可以从代码编译成 JavaScript 来使用。 特别是，关键类如集合(`ArrayList`、`HashMap` 等)、异常(`IllegalArgumentException` 等)以及其他几个关键类(`StringBuilder`、`Comparator`)现在都定义在 `kotlin` 包下。在 JVM 平台上，一些名称是相应 JDK 类的类型别名，而在 JS 平台上，这些类在 Kotlin 标准库中实现。

### 更好的代码生成 ###
JavaScript 后端现在生成更加可静态检查的代码，这对 JS 代码处理工具(如 minifiers、optimisers、linters 等)更加友好。

### external 修饰符 ###
如果你需要以类型安全的方式在 Kotlin 中访问 JavaScript 实现的类， 你可以使用 `external` 修饰符写一个 Kotlin 声明。在 Kotlin 1.0 中，使用了 `@native` 注解。与 JVM 目标平台不同，JS 平台允许对类和属性使用 `external` 修饰符。例如，可以按以下方式声明 DOM Node 类：
```kotlin
external class Node {
    val firstChild: Node
​
    fun appendChild(child: Node): Node
​
    fun removeChild(child: Node): Node
​
    // 等等
}
```

### 改进的导入处理 ###
现在可以更精确地描述应该从 JavaScript 模块导入的声明。 如果在外部声明上添加 `@JsModule("＜模块名＞")` 注解，它会在编译期间正确导入到模块系统(CommonJS 或 AMD)。例如，使用 CommonJS，该声明会通过 `require(……)` 函数导入。 此外，如果要将声明作为模块或全局 JavaScript 对象导入， 可以使用 `@JsNonModule` 注解。

例如，以下是将 JQuery 导入 Kotlin 模块的方法：
```kotlin
external interface JQuery {
    fun toggle(duration: Int = definedExternally): JQuery
    fun click(handler: (Event) -> Unit): JQuery
}
​
@JsModule("jquery")
@JsNonModule
@JsName("$")
external fun jquery(selector: String): JQuery
```

在这种情况下，JQuery 将作为名为 `jquery` 的模块导入。或者，它可以用作 $-对象， 这取决于Kotlin编译器配置使用哪个模块系统。

你可以在应用程序中使用如下所示的这些声明：
```kotlin
fun main(args: Array<String>) {
    jquery(".toggle-button").click {
        jquery(".toggle-panel").toggle(300)
    }
}
fun main(args: Array<String>) {
    jquery(".toggle-button").click {
        jquery(".toggle-panel").toggle(300)
    }
}
```

## 参考 ##
[What's New in Kotlin 1.1](https://kotlinlang.org/docs/reference/whatsnew11.html)

