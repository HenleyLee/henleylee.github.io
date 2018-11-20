---
title: Java 9 新特性
date: 2018-11-17 12:45:38
categories: Java
tags:
  - Java
  - JDK
---

Java 9 正式发布于 2017 年 9 月 21 日。作为 Java 8 之后 3 年半才发布的新版本，Java 9 带来了很多重大的变化。其中最主要的变化是 Java 平台模块系统的引入。除此之外，还有一些新的特性。

## Java 9 的新特性 ##
Java 9 新增了非常多的特性，主要特性包含以下几个：
 - **`模块系统`** - 模块是一个包的容器，Java 9 最大的变化之一是引入了模块系统(Jigsaw 项目)。
 - **`REPL (JShell)`** - 交互式编程环境。
 - **`HTTP/2 客户端`** - HTTP/2 标准是 HTTP 协议的最新版本，新的 HTTPClient API 支持 WebSocket 和 HTTP2 流以及服务器推送特性。
 - **`多版本兼容 JAR 包`** - 多版本兼容 JAR 功能能让你创建仅在特定版本的 Java 环境中运行库程序时选择使用的 class 版本。
 - **`集合工厂方法`** - List，Set 和 Map 接口中，新的静态工厂方法可以创建这些集合的不可变实例。
 - **`私有接口方法`** - 在接口中使用 private 私有方法。我们可以使用 private 访问修饰符在接口中编写私有方法。
 - **`改进的 Javadoc`** - Javadoc 现在支持在 API 文档中的进行搜索。另外，Javadoc 的输出现在符合兼容 HTML5 标准。
 - **`改进的 Optional 类`** - java.util.Optional 添加了很多新的有用方法，Optional 可以直接转为 stream。
 - **`改进的 Stream API`** - 改进的 Stream API 添加了一些便利的方法，使流处理更容易，并使用收集器编写复杂的查询。
 - **`改进的 try-with-resources`** - 如果你已经有一个资源是 final 或等效于 final 变量，您可以在 try-with-resources 语句中使用该变量，而无需在 try-with-resources 语句中声明一个新变量。
 - **`改进的进程 API`** - 改进的 API 来控制和管理操作系统进程。引进 java.lang.ProcessHandle 及其嵌套接口 Info 来让开发者逃离时常因为要获取一个本地进程的 PID 而不得不使用本地代码的窘境。
 - **`改进的弃用注解 @Deprecated`** - 注解 @Deprecated 可以标记 Java API 状态，可以表示被标记的 API 将会被移除，或者已经破坏。
 - **`改进钻石操作符(Diamond Operator)`** - 匿名类可以使用钻石操作符(Diamond Operator)。
 - **`改进的 CompletableFuture API`** - CompletableFuture 类的异步机制可以在 ProcessHandle.onExit 方法退出时执行操作。
 - **`轻量级的 JSON API`** - 内置了一个轻量级的 JSON API。
 - **`多分辨率图像 API`** - 定义多分辨率图像API，开发者可以很容易的操作和展示不同分辨率的图像了。
 - **`响应式流(Reactive Streams) API`** - Java 9 中引入了新的响应式流 API 来支持 Java 9 中的响应式编程。

更多的新特性可以参阅官网：[What's New in JDK 9](https://docs.oracle.com/javase/9/whatsnew/toc.htm)
JDK 9 下载地址：[Java 9 Downloads](https://www.oracle.com/technetwork/java/javase/downloads/jdk9-doc-downloads-3850606.html)

### 模块系统 ###
Java 9 最大的变化之一是引入了模块系统（Jigsaw 项目）。

Java 平台模块系统(Modular System)，把模块化开发实践引入到了 Java 平台中。在引入了模块系统之后，JDK 被重新组织成 94 个模块。Java 应用可以通过新增的 jlink 工具，创建出只包含所依赖的 JDK 模块的自定义运行时镜像。这样可以极大的减少 Java 运行时环境的大小。这对于目前流行的不可变基础设施的实践来说，镜像的大小的减少可以节省很多存储空间和带宽资源 。 

 模块化开发的实践在软件开发领域并不是一个新的概念。Java 开发社区已经使用这样的模块化实践有相当长的一段时间。主流的构建工具，包括 Apache Maven 和 Gradle 都支持把一个大的项目划分成若干个子项目。子项目之间通过不同的依赖关系组织在一起。每个子项目在构建之后都会产生对应的 JAR 文件。 在 Java9 中 ，已有的这些项目可以很容易的升级转换为 Java 9 模块 ，并保持原有的组织结构不变。

Java 9 模块的重要特征是在其工件（artifact）的根目录中包含了一个描述模块的 `module-info.class` 文件。工件的格式可以是传统的 JAR 文件或是 Java 9 新增的 JMOD 文件。这个文件由根目录中的源代码文件 `module-info.java` 编译而来。该模块声明文件可以描述模块的不同特征。模块声明文件中可以包含的内容如下：
 - `模块导出的包：`使用 `exports` 可以声明模块对其他模块所导出的包。包中的 `public` 和 `protected` 类型，以及这些类型的 `public` 和 `protected` 成员可以被其他模块所访问。没有声明为导出的包相当于模块中的私有成员，不能被其他模块使用。
 - `模块的依赖关系：`使用 `requires` 可以声明模块对其他模块的依赖关系。使用 requires transitive 可 以把一个模块依赖声明为传递的。传递的模块依赖可以被依赖当前模块的其他模块所读取。如果一个模块所导出的类型的型构中包含了来自它所依赖的模块的类型，那么对该模块的依赖应该声明为传递的。
 - `服务的提供和使用：`如果一个模块中包含了可以被 `ServiceLocator` 发现的服务接口的实现，需要使用 provides with 语句来声明具体的实现类；如果一个模块需要使用服务接口，可以使用 `uses` 语句来声明。 

下面给出了一个模块 com.mycompany.sample 的最基本的模块声明：
```java
module com.mycompany.sample { 
    exports com.mycompany.sample; 
    requires com.mycompany.common; 
    provides com.mycompany.common.DemoService with
        com.mycompany.sample.DemoServiceImpl; 
}
```

模块系统中增加了模块路径的概念。模块系统在解析模块时，会从模块路径中进行查找。为了保持与之前 Java 版本的兼容性，CLASSPATH 依然被保留。所有的类型在运行时都属于某个特定的模块。对于从 CLASSPATH 中加载的类型，它们属于加载它们的类加载器对应的未命名模块。可以通过 Class 的 getModule()方法来获取到表示其所在模块的 Module 对象。

在 JVM 启动时，会从应用的根模块开始，根据依赖关系递归的进行解析，直到得到一个表示依赖关系的图。如果解析过程中出现找不到模块的情况，或是在模块路径的同一个地方找到了名称相同的模块，模块解析过程会终止，JVM 也会退出。Java 也提供了相应的 API 与模块系统进行交互。

如果想了解和学习更详细的内容，可以参考[官方文档](http://openjdk.java.net/projects/jigsaw/quick-start)。

### REPL (JShell) ###
REPL(Read Eval Print Loop)意为交互式的编程环境。

JShell 是 Java 9 新增的一个`交互式的编程环境工具`。JShell 为 Java 增加了类似 NodeJS 和 Python 中的读取-求值-打印循环(Read-Evaluation-Print Loop)。在 JShell 中可以`直接输入表达式并查看其执行结果`。当需要测试一个方法的运行效果，或是快速的对表达式进行求值时，JShell 都非常实用。只需要通过 JShell 命令启动 jshell，然后直接输入表达式即可。

执行 JShell：
```shell
$ jshell
|  Welcome to JShell -- Version 9-ea
|  For an introduction type: /help intro
jshell>
```

执行 JShell 简单计算：
```shell
jshell> 3+1
$1 ==> 4
jshell> 13%7
$2 ==> 6
jshell> $2
$2 ==> 6
jshell>
```

JShell 创建与使用函数：
```shell
jshell> int doubled(int i){ return i*2;}
|  created method doubled(int)
jshell> doubled(6)
$3 ==> 12
jshell>
```

退出 JShell：
```shell
jshell> /exit
| Goodbye 
```

### HTTP/2 客户端 ###
在此之前，JDK 提供的 Http 访问功能，几乎都需要依赖于 HttpURLConnection，但是这个类大家在写代码的时候很少使用，此次在Java 9的版本中引入了一个新的 package：`java.net.http`，里面提供了对 Http 访问很好的支持，不仅支持 Http/1.1 而且还支持 HTTP/2，以及 WebSocket，据说性能可以超过 Apache HttpClient，Netty，Jetty。
下面是一个简单的使用例子：
```java
URI httpURI = new URI("http://www.baidu.com");
HttpRequest request = HttpRequest.create(httpURI).GET();
HttpResponse response = request.response();
String responseBody = response.body(HttpResponse.asString());
```

### 集合工厂方法 ###
Java 9 在 `List`，`Set` 和 `Map` 接口中，添加了可以创建这些集合的不可变实例新的静态工厂方法。

在 Java 9 之前，Java 只能利用一些实用方法（例如：Collections.unmodifiableCollection(Collection<? extends T> c)）创建一个不可修改视图的集合。现在，Java 9 引入了一些有用的工厂方法来创建不可修改的集合。

Java 9 中，以下方法被添加到 List、Set 和 Map 接口以及它们的重载对象。
```java
static <E> List<E> of(E e1, E e2, E e3);
static <E> Set<E>  of(E e1, E e2, E e3);
static <K,V> Map<K,V> of(K k1, V v1, K k2, V v2, K k3, V v3);
static <K,V> Map<K,V> ofEntries(Map.Entry<? extends K,? extends V>... entries)
```
 - List 和 Set 接口：of(...) 方法重载了 0 ~ 10 个参数的不同方法。
 - Map 接口：of(...) 方法重载了 0 ~ 10 个参数的不同方法。
 - Map 接口：如果超过 10 个参数, 可以使用 ofEntries(...) 方法。

### 私有接口方法 ###
在 Java 8 之前，接口可以有`常量变量`和`抽象方法`。我们不能在接口中提供方法实现，如果我们要提供抽象方法和非抽象方法（方法与实现）的组合，那么我们就得使用抽象类。

在 Java 8 接口引入了`默认方法`和`静态方法`。我们可以在 Java 8 的接口中编写方法实现，仅仅需要使用 `default` 关键字来定义它们。在 Java 8 中，一个接口中能定义如下几种变量/方法：常量、抽象方法、默认方法、静态方法。

Java 9 不仅像 Java 8 一样支持接口默认方法，同时还支持`私有方法`。在 Java 9 中，一个接口中能定义如下几种变量/方法：常量、抽象方法、默认方法、静态方法、私有方法、私有静态方法

默认方法和静态方法可以共享接口中的私有方法，因此避免了代码冗余，这也使代码更加清晰。如果私有方法是静态的，那这个方法就属于这个接口的，并且没有 `static` 修饰的私有方法只能被在接口中的实例调用。
```java
public interface InterfaceWithPrivateMethods {
    
    private static String staticPrivate() {
        return "static private";
    }

    private String instancePrivate() {
        return "instance private";
    }

    default void check() {
        String result = staticPrivate();
        InterfaceWithPrivateMethods methods = new InterfaceWithPrivateMethods() {
            // anonymous class 匿名类
        };
        result = methods.instancePrivate();
    }

}
```

### 改进的 Javadoc ###
javadoc 工具可以生成 Java 文档。Javadoc 现在支持在 API 文档中的进行搜索。另外， Java 9 的 javadoc 的输出现在符合兼容 HTML5 标准。使用 Java 9 javadoc 命令中的 `-html5` 参数可以让生成的文档支持 HTML5 标准。

### 改进 Optional 类 ###
Optional 类在 Java 8 中引入，Optional 类的引入很好的解决空指针异常。在 java 9 中, 添加了三个方法来改进它的功能：
 - **`stream()：`**将 Optional 转为一个 `Stream`，如果该 Optional 中包含值，那么就返回包含这个值的 Stream，否则返回一个空的 Stream（`Stream.empty()`）。
 - **`ifPresentOrElse()：`**如果一个 Optional 包含值，则对其包含的值调用函数 `action`，即 `action.accept(value)`，这与 ifPresent 一致；与 ifPresent 方法的区别在于，ifPresentOrElse 还有第二个参数 `emptyAction` —— 如果 Optional 不包含值，那么 ifPresentOrElse 便会调用 `emptyAction`，即 `emptyAction.run()`。
 - **`or()：`**如果值存在，返回 Optional 指定的值，否则返回一个预设的值。

### 改进的 Stream API ###
Java 9 改进的 Stream API 添加了一些便利的方法，使流处理更容易，并使用收集器编写复杂的查询。Java 9 为 Stream 新增了几个方法：`dropWhile`、`takeWhile`、`ofNullable`，为 `iterate` 方法新增了一个重载方法。
 - **`takeWhile()：`**使用一个断言作为参数，返回给定 Stream 的子集直到断言语句第一次返回 false。如果第一个值不满足断言条件，将返回一个空的 Stream。
 - **`dropWhile()：`**和 takeWhile 作用相反的，使用一个断言作为参数，直到断言语句第一次返回 true 才返回给定 Stream 的子集。
 - **`ofNullable()：`**可以预防 NullPointerExceptions 异常， 可以通过检查流来避免 null 值。
 - **`iterate()：`**允许使用初始种子值创建顺序（可能是无限）流，并迭代应用指定的下一个方法。 当指定的 hasNext 的 predicate 返回 false 时，迭代停止。
 
### 改进的 try-with-resources ###
try-with-resources 是 JDK 7 中一个新的异常处理机制，它能够很容易地关闭在 try-catch 语句块中使用的资源。所谓的资源（resource）是指在程序完成后，必须关闭的对象。try-with-resources 语句确保了每个资源在语句结束时关闭。所有实现了 java.lang.AutoCloseable 接口（其中，它包括实现了 java.io.Closeable 的所有对象），可以使用作为资源。

try-with-resources 声明在 JDK 9 已得到改进。如果你已经有一个资源是 `final` 或`等效于 final 变量`，您可以在 try-with-resources 语句中使用该变量，而无需在 try-with-resources 语句中声明一个新变量。

```java
public String readData(String message) throws IOException {
    Reader inputString = new StringReader(message);
    BufferedReader br = new BufferedReader(inputString);
    try (br) {
        return br.readLine();
    }
}
```
在处理必须关闭的资源时，使用 try-with-resources 语句替代 try-finally 语句。 生成的代码更简洁，更清晰，并且生成的异常更有用。try-with-resources 语句在编写必须关闭资源的代码时会更容易，也不会出错，而使用 try-finally 语句实际上是不可能的。

### 改进的进程 API ###
在 Java 9 之前，Process API 仍然缺乏对使用本地进程的基本支持，例如获取进程的 PID 和所有者，进程的开始时间，进程使用了多少 CPU 时间，多少本地进程正在运行等。

Java 9 向 Process API 添加了一个名为 `ProcessHandle` 的接口来增强 `java.lang.Process` 类，可以对原生进程进行管理，尤其适合于管理长时间运行的进程。

在使用 `ProcessBuilder` 来启动一个进程之后，可以通过 `Process.toHandle()` 方法来得到一个 `ProcessHandle` 对象的实例。通过 `ProcessHandle` 可以获取到由 `ProcessHandle.Info` 表示的进程的基本信息，如命令行参数、可执行文件路径和启动时间等。`ProcessHandle` 的 `onExit()` 方法返回一个 `CompletableFuture<ProcessHandle>` 对象，可以在进程结束时执行自定义的动作。
```java
final ProcessBuilder processBuilder = new ProcessBuilder("top") 
    .inheritIO(); 
final ProcessHandle processHandle = processBuilder.start().toHandle(); 
processHandle.onExit().whenCompleteAsync((handle, throwable) -> { 
    if (throwable == null) { 
        System.out.println(handle.pid()); 
    } else { 
        throwable.printStackTrace(); 
    } 
});
```

### 改进的弃用注解 @Deprecated ###
注解 `@Deprecated` 可以标记 Java API 状态，可以是以下几种：
 - 使用它存在风险，可能导致错误；
 - 可能在未来版本中不兼容；
 - 可能在未来版本中删除；
 - 一个更好和更高效的方案已经取代它。

Java 9 中注解增加了两个新元素：`since` 和 `forRemoval`。
 - **`since：`**元素指定已注解的 API 元素已被弃用的版本。
 - **`forRemoval：`**元素表示注解的 API 元素在将来的版本中被删除，应该迁移 API。 

Java 9 中也提供了扫描 jar 文件的工具 `jdeprscan`。这款工具也可以扫描一个聚合类，这个类使用了 Java SE 中的已废弃的 API 元素。这个工具将会对使用已经编译好的库的应用程序有帮助，这样使用者就不知道这个已经编译好的库中使用了那些已废弃的 API。

### 改进钻石操作符(Diamond Operator) ###
钻石操作符是在 Java 7 中引入的，可以让代码更易读，但它不能用于匿名的内部类。

在 Java 9 中，它可以与匿名的内部类一起使用，从而提高代码的可读性。在 Java 9 中，我们可以在匿名类中使用 `<>` 操作符。

### 轻量级的 JSON API ###
Java 9 带来一个轻量级的 API，用于通过 JSON(JavaScriopt 对象符号) 数据交换格式处理和生成文件以及数据流，JSON 是基于 JavaScript 的子集，用来代替 XML。

这个 API 的主要目标如下：
 - 解析和生成 JSON。
 - 满足 Java 开发者使用 JSON 的功能性需求。
 - 解析 API 可以选择标记流，事件(包括文件层次结构)流，或不可变树的方式来呈现文档或数据流视图。
 - 用于紧凑配置和 Java ME 的 API 子集。
 - 使用创建者模式 API 构造不可变的树型结构。
 - 生成器风格 API，用于输出 JSON 数组流和 JSON “文本”。
 - 一个转换 API，将已有的树形值输入转换成另一个树形值输出。

预计 JEP 会把它作为 java.util 的子包交付，至少包含 4 个模块：事件、流、树和生成器。预期不会修改现有的模块、包或类。预计 JSON API 不会依赖 Java 基础模块之外的模块。

### 多分辨率图像 API ###
Java 9 定义了多分辨率图像 API，开发者可以很容易的操作和展示不同分辨率的图像。

以下是多分辨率图像的主要操作方法：
 - **`Image getResolutionVariant(double destImageWidth, double destImageHeight)：`**获取特定分辨率的图像变体-表示一张已知分辨率单位为DPI的特定尺寸大小的逻辑图像，并且这张图像是最佳的变体。
 - **`List<Image> getResolutionVariants()：`**返回可读的分辨率的图像变体列表。

基于当前屏幕分辨率大小和运用的图像转换算法，`java.awt.Graphics` 类可以从 `MultiResolutionImage` 接口获取所需的变体。`java.awt.image.AbstractMultiResolutionImage` 类提供了 `java.awt.image.AbstractMultiResolutionImage` 默认实现。`AbstractMultiResolutionImage` 的基础实现是 `java.awt.image.BaseMultiResolutionImage`。

### 改进的 CompletableFuture API ###
Java 8 引入了 `CompletableFuture<T>` 类，可能是 `java.util.concurrent.Future<T>` 明确的完成版（设置了它的值和状态），也可能被用作 `java.util.concurrent.CompleteStage`。支持 `future` 完成时触发一些依赖的函数和动作。Java 9 引入了一些 `CompletableFuture` 的改进：

Java 9 对 CompletableFuture 做了改进：
1. 支持 delays 和 timeouts
```java
public CompletableFuture<T> completeOnTimeout(T value, long timeout, TimeUnit unit)
```
在 `timeout` 前以给定的 `value` 完成这个 CompletableFutrue，并返回这个 CompletableFutrue。
```java
public CompletableFuture<T> orTimeout(long timeout, TimeUnit unit)
```
如果没有在给定的 `timeout` 内完成，就以 `java.util.concurrent.TimeoutException` 完成这个 CompletableFutrue，并返回这个 CompletableFutrue。

2. 增强了对子类化的支持
做了许多改进使得 `CompletableFuture` 可以被更简单的继承。比如，你也许想重写新的 `public Executor defaultExecutor()` 方法来代替默认的 `executor`。
另一个新的使子类化更容易的方法是：
```java
public <U> CompletableFuture<U> newIncompleteFuture()
```

3. 新的工厂方法
Java 8 引入了 `<U> CompletableFuture<U> completedFuture(U value)` 工厂方法来返回一个已经以给定 value 完成了的 `CompletableFuture`。Java 9 以 一个新的 `<U> CompletableFuture<U> failedFuture(Throwable ex)` 来补充了这个方法，可以返回一个以给定异常完成的 `CompletableFuture`。
除此以外，Java 9 引入了下面这对 stage-oriented 工厂方法，返回完成的或异常完成的 completion stages:
 - **`<U> CompletionStage<U> completedStage(U value)：`**返回一个新的以指定 value 完成的 `CompletionStage`，并且只支持 `CompletionStage` 里的接口。
 - **`<U> CompletionStage<U> failedStage(Throwable ex)：`**返回一个新的以指定异常完成的 `CompletionStage`，并且只支持 `CompletionStage` 里的接口。

### 响应式流(Reactive Streams) API ###
响应式编程的思想最近得到了广泛的流行。在 Java 平台上有流行的响应式库 `RxJava` 和 `Reactor`。反应式流规范的出发点是提供一个带非阻塞负压(non-blocking backpressure)的异步流处理规范。响应式流规范的核心接口已经添加到了 Java 9 中的 `java.util.concurrent.Flow` 类中。

`java.util.concurrent.Flow` 中包含以下 4 个核心接口：
 - **`Flow.Publisher：`**发布者
 - **`Flow.Subscriber：`**订阅者
 - **`Flow.Subscription：`**订阅管理器
 - **`Flow.Processor：`**处理器

Java 9 还提供了 `SubmissionPublisher` 作为 `Flow.Publisher` 的一个实现。RxJava 2 和 Reactor 都可以很方便的与 Flow 类的核心接口进行互操作。 

### I/O 流新特性 ###
类 `java.io.InputStream` 中增加了新的方法来读取和复制 `InputStream` 中包含的数据：
 - **`readAllBytes()：`**读取 InputStream 中的所有剩余字节。
 - **`readNBytes()：`**从 InputStream 中读取指定数量的字节到数组中。
 - **`transferTo()：`**读取 InputStream 中的全部字节并写入到指定的 OutputStream 中 。 

 `ObjectInputFilter` 可以对 `ObjectInputStream` 中包含的内容进行检查，来确保其中包含的数据是合法的。可以使用 ObjectInputStream 的方法 `setObjectInputFilter` 来设置。`ObjectInputFilter` 在进行检查时，可以检查如对象图的最大深度、对象引用的最大数量、输入流中的最大字节数和数组的最大长度等限制，也可以对包含的类的名称进行限制。 

### 改进应用安全性能 ###
Java 9 新增了 4 个 `SHA-3` 哈希算法，`SHA3-224`、`SHA3-256`、`SHA3-384` 和 `SHA3-512`。另外也增加了通过 `java.security.SecureRandom` 生成使用 `DRBG` 算法的强随机数。

### 变量句柄 ###
变量句柄是一个变量或一组变量的引用，包括静态域，非静态域，数组元素和堆外数据结构中的组成部分等。变量句柄的含义类似于已有的方法句柄。变量句柄由 Java 类 `java.lang.invoke.VarHandle` 来表示。可以使用类 `java.lang.invoke.MethodHandles.Lookup` 中的静态工厂方法来创建 `VarHandle` 对象。通过变量句柄，可以在变量上进行各种操作。这些操作称为访问模式。不同的访问模式尤其在内存排序上的不同语义。目前一共有31种访问模式，而每种访问模式都在 `VarHandle` 中有对应的方法。这些方法可以对变量进行读取、写入、原子更新、数值原子更新和比特位原子操作等。`VarHandle` 还可以用来访问数组中的单个元素，以及把 `byte[]` 数组和 `ByteBuffer` 当成是不同原始类型的数组来访问。

### 改进方法句柄(Method Handle) ###
类 `java.lang.invoke.MethodHandles` 增加了更多的静态方法来创建不同类型的方法句柄：
 - **`arrayConstructor：`**创建指定类型的数组。
 - **`arrayLength：`**获取指定类型的数组的大小。
 - **`varHandleInvoker 和 varHandleExactInvoker：`**调用 VarHandle 中的访问模式方法。
 - **`zero：`**返回一个类型的默认值。
 - **`empty：`**返 回 MethodType 的返回值类型的默认值。
 - **`loop、countedLoop、iteratedLoop、whileLoop 和 doWhileLoop：`**创建不同类型的循环，包括 for 循环、while 循环 和 do-while 循环。
 - **`tryFinally：`**把对方法句柄的调用封装在 try-finally 语句中。 

### 用户界面 ###
类 `java.awt.Desktop` 增加了新的与桌面进行互动的能力。可以使用 `addAppEventListener` 方法来添加不同应用事件的监听器，包括应用变为前台应用、应用隐藏或显示、屏幕和系统进入休眠与唤醒、以及用户会话的开始和终止等。还可以在显示关于窗口和配置窗口时，添加自定义的逻辑。在用户要求退出应用时，可以通过自定义处理器来接受或拒绝退出请求。在 AWT 图像支持方面，可以在应用中使用多分辨率图像。

### 统一 JVM 日志 ###
Java 9 中，JVM 有了统一的日志记录系统，可以使用新的命令行选项 `-Xlog` 来控制 JVM 上 所有组件的日志记录。该日志记录系统可以设置输出的日志消息的标签、级别、修饰符和输出目标等。Java 9 移除了在 Java 8 中被废弃的垃圾回收器配置组合，同时把 G1 设为默认的垃圾回收器实现。另外，CMS 垃圾回收器已经被声明为废弃。Java 9 也增加了很多可以通过 `jcmd` 调用的诊断命令。 

### 平台日志 API 和 服务 ###
Java 9 允许为 JDK 和应用配置同样的日志实现。新增的 `System.LoggerFinder` 用来管理 JDK 使用的日志记录器实现。JVM 在运行时只有一个系统范围的 `LoggerFinder` 实例。`LoggerFinder` 通过服务查找机制来加载日志记录器实现。默认情况下，JDK 使用 `java.logging` 模块中的 `java.util.logging` 实现。通过 `LoggerFinder` 的 `getLogger()` 方法就可以获取到表示日志记录器的`System.Logger` 实现。应用同样可以使用 `System.Logger` 来记录日志。这样就保证了 JDK 和应用使用同样的日志实现。我们也可以通过添加自己的 `System.LoggerFinder` 实现来让 JDK 和应用使用 SLF4J 等其他日志记录框架。

## 参考资源 ##
 - 参考 [Java 9 官方文档](https://docs.oracle.com/javase/9/)，了解 Java 9 的更多内容。
 - 参考 [Java 9 API 文档](https://docs.oracle.com/javase/9/docs/api/index.html)，了解 Java 9 API 的细节。 
 - 参考 [What's New in JDK 9](https://docs.oracle.com/javase/9/whatsnew/toc.htm)，了解 Java 9 的新特性。
 - 参考 [Java 9 Release Notes](https://www.oracle.com/technetwork/java/javase/9-relnotes-3622618.html)，了解 Java 9 的更新说明。
