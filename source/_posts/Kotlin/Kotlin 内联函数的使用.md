---
title: Kotlin 内联函数的使用
categories: Kotlin
tags:
  - Kotlin
abbrlink: edcfdfb8
date: 2019-04-28 11:26:36
---

Kotlin 一个强大之处就在于它的扩展函数，巧妙的运用这些扩展函数可以让你写出的代码更加优雅，阅读起来更加流畅。

## 内联函数 ##
在写代码的时候难免会遇到这种情况，就是很多处的代码是一样的，于是通常会抽取出一个公共方法来进行调用，这样看起来就会很简洁；但是也出现了一个问题，就是这个方法会被频繁调用，就会很耗费资源。
举个例子：
```kotlin
fun <T> method(lock: Lock, body: () -> T): T {
    lock.lock()
    try {
        return body()
    } finally {
        lock.unlock()
    }
}
```
这里的 `method()` 方法在调用的时候是不会把形参传递给其他方法的，调用一下：
```kotlin
method(lock, { "我是body的方法体" })// lock是一个Lock对象
```

对于编译器来说，调用 `method()` 方法就要将参数 `lock` 和 `lambda` 表达式 `{"我是body的方法体"}` 进行传递，就要将 `method()` 方法进行压栈出栈处理，这个过程就会耗费资源。如果是很多地方调用，就会执行很多次，这样就非常消耗资源了，于是就引入了内联函数。

被 `inline` 关键字标记的函数就是内联函数，其原理就是：在编译时期，把调用这个函数的地方用这个函数的方法体进行替换。

## run() ##
### 定义 ###
`run()` 函数的定义如下：
```kotlin
public inline fun <R> run(block: () -> R): R
public inline fun <T, R> T.run(block: T.() -> R): R
```

### 功能 ###
`run()` 函数的功能：调用指定的函数 `block`，返回值为函数的最后一行或 `return` 表达式。

### 示例 ###
返回 `return` 表达式，`return` 后面的代码不再执行(注意写法 `@run`)：
```kotlin
run {
    return@run println("11")
    println("22")
}
```

## with() ##
### 定义 ###
`with()` 函数的定义如下：
```kotlin
public inline fun <T, R> with(receiver: T, block: T.() -> R): R
```

### 功能 ###
`with()` 函数的功能：使用给定的 `receiver` 作为接收器调用指定的函数 `block`，在函数内可以通过 `this` 指代该对象，返回值为函数的最后一行或 `return` 表达式。

### 示例 ###
1. 在自定义 View 中初始化画笔时很多时候会写下以下代码：
```kotlin
var paint = Paint()
paint.color = Color.BLACK
paint.strokeWidth = 1.0f
paint.textSize = 18.0f
paint.isAntiAlias = true
```
如果使用 `with()` 函数，那么就可以写成这样
```kotlin
var paint = Paint()
with(paint) {
    color = Color.BLACK
    strokeWidth = 1.0f
    textSize = 18.0f
    isAntiAlias = true
}
```
省去了 `paint.`，后书写起来感觉会更加自然。

2. 在声明一些集合的场景，比如：
```kotlin
var list = mutableListOf<String>()
list.add("1")
list.add("2")
list.add("3")
```
如果使用 `with()` 函数，那么就可以写成这样
```kotlin
var list = with(mutableListOf<String>()) {
    add("1")
    add("2")
    add("3")
    this
}
```

## apply() ##
### 定义 ###
`apply()` 函数的定义如下：
```kotlin
public inline fun <T> T.apply(block: T.() -> Unit): T
```

### 功能 ###
`apply()` 函数的功能：使用 `this` 值作为接收器调用指定的函数 `block`，在函数范围内可以任意调用该对象的任意方法，返回值为 `this` 值(该对象)。

### 示例 ###
1. 在自定义 View 中初始化画笔时很多时候会写下以下代码：
```kotlin
var paint = Paint()
paint.color = Color.BLACK
paint.strokeWidth = 1.0f
paint.textSize = 18.0f
paint.isAntiAlias = true
```
如果使用 `apply()` 函数，那么就可以写成这样
```kotlin
var paint = Paint().apply {
    color = Color.BLACK
    strokeWidth = 1.0f
    textSize = 18.0f
    isAntiAlias = true
}
```

2. 在声明一些集合的场景，比如：
```kotlin
var list = mutableListOf<String>()
list.add("1")
list.add("2")
list.add("3")
```
如果使用 `apply()` 函数，那么就可以写成这样
```kotlin
var list = mutableListOf<Int>().apply {
    add(1)
    add(2)
    add(3)
}
```

> 注意 `apply()` 和 `run()` 函数的区别，`run()` 函数返回的是最后一行，`apply()` 函数返回的是对象本身。由 `apply()` 函数的定义可以看出 `apply()` 函数适用于那些对象初始化需要给其属性赋值的情况。

## also() ##
### 定义 ###
`also()` 函数的定义如下：
```kotlin
public inline fun <T> T.also(block: (T) -> Unit): T
```

### 功能 ###
`also()` 函数的功能：使用 `this` 值作为参数调用指定的函数 `block`，在函数范围内可以通过 `it` 指代该对象，返回值为 `this` 值(该对象)。

### 示例 ###
在自定义 View 中初始化画笔时很多时候会写下以下代码：
```kotlin
var paint = Paint()
paint.color = Color.BLACK
paint.strokeWidth = 1.0f
paint.textSize = 18.0f
paint.isAntiAlias = true
```
如果使用 `apply()` 函数，那么就可以写成这样
```kotlin
var textView = Paint().also {
    it.color = Color.BLACK
    it.strokeWidth = 1.0f
    it.textSize = 18.0f
    it.isAntiAlias = true
}
```

## let() ##
### 定义 ###
`let()` 函数的定义如下：
```kotlin
public inline fun <T, R> T.let(block: (T) -> R): R
```

### 功能 ###
`let()` 函数的功能：使用 `this` 值作为参数调用指定的函数 `block`，在函数范围内可以通过 `it` 指代该对象，返回值为函数的最后一行或 `return` 表达式。

### 示例 ###
`let()` 函数有点类似于 `run()` 函数，`let()` 函数在使用中可用于空安全验证，`变量?.let{}`：
```kotlin
var paint: Paint?

paint?.let {
    it.color = Color.BLACK
    it.strokeWidth = 1.0f
    it.textSize = 18.0f
    it.isAntiAlias = true
}
```

## takeIf() ##
### 定义 ###
`takeIf()` 函数的定义如下：
```kotlin
public inline fun <T> T.takeIf(predicate: (T) -> Boolean): T?
```

### 功能 ###
`takeIf()` 函数的功能：使用 `this` 值作为参数判断是否满足给定的 `predicate`，如果满足则返回 `this`，否则返回 `null`。

## takeUnless() ##
### 定义 ###
`takeUnless()` 函数的定义如下：
```kotlin
public inline fun <T> T.takeUnless(predicate: (T) -> Boolean): T?
```

### 功能 ###
`takeUnless()` 函数的功能：使用 `this` 值作为参数判断是否满足给定的 `predicate`，如果不满足则返回 `this`，否则返回 `null`，和 `takeIf()` 函数正好相反。

## repeat() ##
### 定义 ###
`repeat()` 函数的定义如下：
```kotlin
public inline fun repeat(times: Int, action: (Int) -> Unit)
```

### 功能 ###
`repeat()` 函数的功能：连续执行指定次数的指定函数 `action`，其实就是简化了 `for` 循环。

## 函数选择 ##
![Kotlin内联函数选择](https://henleylee.github.io/medias/kotlin/kotlin_inline_function.png)

