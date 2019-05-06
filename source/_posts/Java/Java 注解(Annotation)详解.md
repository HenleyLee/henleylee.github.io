---
title: Java 注解(Annotation)详解
categories: Java
tags:
  - Java
  - 注解
abbrlink: 52b68731
date: 2018-11-25 09:46:20
---

## 什么叫注解 ##
**`注解(Annotation)`**就是 Java 提供了一种元程序中的元素关联任何信息和着任何元数据(metadata)的途径和方法。`Annotation` 是一个接口，程序可以通过反射来获取指定程序元素的 `Annotation` 对象，然后通过 `Annotation` 对象来获取注解里面的元数据。

`Annotation` 是 `JDK 5.0` 及以后版本引入的。它可以用于创建文档，跟踪代码中的依赖性，甚至执行基本编译时检查。`Annotation` 是一种应用于`包`、`类型`、`构造方法`、`方法`、`成员变量`、`参数`、`本地变量`的声明中的`特殊修饰符`。这些信息被存储在 `Annotation` 的 `“name=value”` 结构对中。

在注解出现之前，程序的元数据只是通过 java 注释和 javadoc，但是注解提供的功能要远远超过这些。注解不仅包含了元数据，它还可以作用于程序运行过程中、注解解释器可以通过注解决定程序的执行顺序。

比如，下面这段代码：
```java
@Override
public String toString() {
    return "This is a string representation of the current object.";
}
```

## 什么是 metadata 元数据 ##
元数据从 metadata 一词译来，就是“关于数据的数据”的意思。

元数据的功能作用有很多，比如：你可能用过 Javadoc 的注释自动生成文档。这就是元数据功能的一种。总的来说，元数据可以用来创建文档，跟踪代码的依赖性，执行编译时格式检查，代替已有的配置文件。如果要对于元数据的作用进行分类，目前还没有明确的定义，不过可以根据它所起的作用，大致可分为三类： 
 - **`编写文档：`**通过代码里标识的元数据生成文档。
 - **`代码分析：`**通过代码里标识的元数据对代码进行分析。
 - **`编译检查：`**通过代码里标识的元数据让编译器能实现基本的编译检查。

## 为什么要使用注解 ##
使用 Annotation 之前(甚至在使用之后)，XML 被广泛的应用于描述元数据。不知何时开始一些应用开发人员和架构师发现 XML 的维护越来越糟糕了。他们希望使用一些和代码紧耦合的东西，而不是像XML那样和代码是松耦合的(在某些情况下甚至是完全分离的)代码描述。如果你在 Google 中搜索 `XML vs. annotations`，会看到许多关于这个问题的辩论。最有趣的是 XML 配置其实就是为了分离代码和配置而引入的。上述两种观点可能会让你很疑惑，两者观点似乎构成了一种循环，但各有利弊。下面我们通过一个例子来理解这两者的区别。

假如你想为应用设置很多的常量或参数，这种情况下，XML 是一个很好的选择，因为它不会同特定的代码相连。如果你想把某个方法声明为服务，那么使用 Annotation 会更好一些，因为这种情况下需要注解和方法紧密耦合起来，开发人员也必须认识到这点。

另一个很重要的因素是 Annotation 定义了一种标准的描述元数据的方式。在这之前，开发人员通常使用他们自己的方式定义元数据。例如，使用标记 `interfaces`，注释，`transient` 关键字等等。每个程序员按照自己的方式定义元数据，而不像 Annotation 这种标准的方式。

目前，许多框架将 `XML` 和 `Annotation` 两种方式结合使用，平衡两者之间的利弊。

## Annotation 和 Annotation 类型 ##
Annotation 使用了在 Java5 所带来的新语法，它的行为十分类似 `public`、`final` 这样的修饰符。每个 `Annotation` 具有一个名字和零个或多个成员。每个 `Annotation` 的成员具有被称为 `name=value` 对的名字和值（就像 JavaBean 一样），`name=value` 装载了 `Annotation` 的信息。

Annotation 类型定义了 Annotation 的`名字`、`类型`、`成员默认值`。一个 Annotation 类型可以说是一个特殊的 Java 接口，它的成员变量是受限制的，而声明 Annotation 类型时需要使用新语法。当我们通过 Java **`反射`** API 访问 Annotation 时，返回值将是一个实现了该 `Annotation` 类型接口的对象，通过访问这个对象能方便的访问到其 `Annotation` 成员。

## 注解的分类 ##
### 按照运行机制划分 ###
按照注解的参数个数划分，注解可以分为以下三种：
 - **`标记注解：`**没有变量，只有名称标识。这种类型仅仅使用自身的存在与否来为开发者提供信息，如 @annotation
 - **`单值注解：`**在标记注解的基础上提供一段数据。如 @annotation("data")
 - **`完整注解：`**可以包括多个数据成员，每个数据成员由名称和值构成。如 @annotation(value1 = "data1", value2 = "data2")

按照运行机制划分，注解可以分为以下三种：
 - **`源码注解：`**只在源码中存在，编译成 class 文件就不存在了。
 - **`编译时注解：`**在源码和 class 文件中都存在。例如 @Override、@Deprecated、@SuppressWarnings，他们都属于编译时注解。
 - **`运行时注解：`**在运行阶段还起作用，甚至会影响运行逻辑的注解。像 @Autowired 自动注入的这样一种注解就属于运行时注解，它会在程序运行的时候把你的成员变量自动的注入进来。

### 按照来源划分 ###
按照使用方法和用途划分，注解可以分为以下三种：
 - **`JDK 内置注解：`**Java 目前只内置了五种标准注解和五种元注解。
 - **`第三方的注解：`**这一类注解是我们接触最多和作用最大的一类。
 - **`自定义注解：`**可以看作是开发者自己编写的注解。

### 元注解 ###
**`元注解`**是给注解进行注解，可以理解为注解的注解就是元注解。

## JDK 内置注解 ##
在 `java.lang` 包下，JDK 提供了 `5` 个基本注解：`@Override`、`@Deprecated`、`@SuppressWarnings`、`@SafeVarargs`、`@FunctionalInterface`。

### @Override ###
**`@Override`** 用于标注重写了父类的方法。

当我们想要复写父类中的方法时，我们需要使用该注解去告知编译器我们想要复写这个方法，这样一来当父类中的方法移除或者发生更改时编译器将提示错误信息。对于子类中被 `@Override` 修饰的方法，如果存在对应的被重写的父类方法，则正确；如果不存在，则报错。`@Override` 只能作用于方法，不能作用于其他程序元素。

### @Deprecated ###
**`@Deprecated`** 用于表示某个程序元素（类、方法等）已过时。

当我们希望编译器知道某一方法不建议使用时，我们应该使用这个注解。Java 在 javadoc 中推荐使用该注解，我们应该提供为什么该方法不推荐使用以及替代的方法。如果使用了被 `@Deprecated` 修饰的类或方法等，编译器会发出警告。

### @SuppressWarnings ###
**`@SuppressWarnings`** 用于抑制编译器的警告。

这个仅仅是告诉编译器忽略特定的警告信息，例如在泛型中使用原生数据类型。指示被 `@SuppressWarnings` 修饰的程序元素（以及该程序元素中的所有子元素，例如类以及该类中的方法）取消显示指定的编译器警告。例如，常见的 `@SuppressWarnings("unchecked")`。

**`@SuppressWarnings`** 注解的常见参数值主要有以下几种：
 - `deprecation：`使用了不赞成使用的类或方法时的警告(使用 `@Deprecated` 使得编译器产生的警告)；
 - `unchecked：`执行了未检查的转换时的警告，例如当使用集合时没有用泛型 (Generics) 来指定集合保存的类型; 关闭编译器警告
 - `fallthrough：`当 switch 程序块直接通往下一种情况而没有 break 语句时的警告;
 - `path：`在类路径、源文件路径等中有不存在的路径时的警告;
 - `serial：`当在可序列化的类上缺少 serialVersionUID 定义时的警告;
 - `finally：`任何 finally 子句不能正常完成时的警告;
 - `all：`关于以上所有情况的警告。

### @SafeVarargs ###
**`@SafeVarargs`** 是 JDK 7 专门为抑制`堆污染`警告提供的。

### @FunctionalInterface (Java8新增) ###
**`@FunctionalInterface`** 是 Java8 中新增的函数式接口。Java8 规定：如果接口中只有一个抽象方法(可以包含多个 `default` 方法或多个 `static` 方法)，该接口称为函数式接口。

> **注意：**
> 1. **`value 特权：`**如果使用注解时只需要为 `value` 成员变量指定值，则使用注解时可以直接在该注解的括号中指定 `value` 值，而无需使用 `name=value` 的形式(如`@SuppressWarnings("unchecked")`)。
> 2. **`坚持使用 @Override 注解：`**如果在每个方法中使用 `@Override` 注解来声明要覆盖父类声明，编译器就可以替你防止大量的错误。

## JDK 元注解 ##
元注解(meta-annotation)是指注解的注解。元注解是 Java 定义的用于创建注解的工具，它们本身也是注解。在 `java.lang.annotation` 包下，JDK 提供了 `5` 个标准的元注解类型：`@Retention`、`@Target`、`@Inherited`、`@Documented`、`@Repeatable`。

### @Retention ###
**`@Retention：`**注解的保留策略。该注解指明了被它所注解的注解被保留的时间长短。

**`@Retention`** 包含一个名为 `value` 的成员变量，该 `value` 成员变量是 `RetentionPolicy`，`RetentionPolicy` 是枚举类型，值有如下几个：
 - **`RetentionPolicy.SOURCE：`**注解只在源码阶段保留，在编译器进行编译时它将被丢弃忽视，不记录在 class 文件中。
 - **`RetentionPolicy.CLASS：`**注解只被保留到编译进行的时候，它并不会被加载到 JVM 中。这是默认行为，所有没有用 @Retention 注解的注解，都会采用这种策略。
 - **`RetentionPolicy.RUNTIME：`**注解可以保留到程序运行的时候，它会被加载进入到 JVM 中，程序可以通过反射获取该注解的信息。

### @Target ###
**`@Target：`**注解的作用目标。该注解指定注解用于修饰哪些程序元素。

**`@Target`** 也包含一个名为 `value` 的成员变量，该 `value` 成员变量类型为 `ElementType[]`，`ElementType` 也为枚举类型，值有如下几个：
 - **`ElementType.TYPE：`** 修饰类型，比如接口、类、枚举、注解
 - **`ElementType.FIELD：`** 修饰属性，比如成员变量、枚举常量
 - **`ElementType.METHOD：`** 修饰方法
 - **`ElementType.PARAMETER：`**修饰方法内的参数
 - **`ElementType.CONSTRUCTOR：`**修饰构造方法
 - **`ElementType.LOCAL_VARIABLE：`**修饰局部变量
 - **`ElementType.ANNOTATION_TYPE：`**修饰注解
 - **`ElementType.PACKAGE：`**修饰包
 - **`ElementType.TYPE_PARAMETER：`**修饰类型参数(Java8 新增)
 - **`ElementType.TYPE_USE：`**修饰任何类型(Java8 新增)

### @Inherited ###
**`@Inherited：`**指定注解具有继承性。但是它并不是说注解本身可以继承，而是说如果一个超类被 @Inherited 注解过的注解进行注解的话，那么如果它的子类没有被任何注解应用的话，那么这个子类就继承了超类的注解。

### @Documented ###
**`@Documented：`**注解将被包含在 Javadoc 中。该注解的作用是在用 Javadoc 命令生成 API 文档时能够将注解中的元素包含到 Javadoc 中去。

### @Repeatable (Java8新增) ###
**`@Repeatable：`** 表示可重复注解。`@Repeatable` 是 Java 8 才加进来的，所以算是一个新的特性。

在实际应用中，可能会出现需要对同一个声明式或者类型加上相同的注解（包含不同的属性值）的情况。例如系统中除了管理员之外，还添加了超级管理员这一权限，对于某些只能由这两种角色调用的特定方法，可以使用可重复注解。

## 自定义注解 ##
使用 **`@interface`** 自定义注解时，自动继承了 **`java.lang.annotation.Annotation`** 接口，由编译程序自动完成其他细节。在定义注解时，不能继承其他的注解或接口。 `@interface` 用来声明一个注解，其中的每一个方法实际上是声明了一个配置参数。方法的名称就是参数的名称，返回值类型就是参数的类型(返回值类型只能是`基本类型`、`Class`、`String`、`Enum`)。可以通过 `default` 来声明参数的默认值。

定义注解格式：
```java
public @interface 注解名 {
    定义体
}
```

自定义注解就需要用到上面所介绍到的几种元注解，可以看出元注解就是用来注解其它注解。自定义注解和接口类似，只能定义方法。注解参数的可支持数据类型：
 - 所有基本数据类型(int、float、boolean、byte、double、char、long、short)
 - String 类型
 - Class 类型
 - Enum 类型
 - Annotation 类型
 - 以上所有类型的数组

定义新注解使用 **`@interface`** 关键字，其定义过程与定义接口非常类似，需要注意的是：Annotation 的成员变量在 Annotation 定义中是以无参的方法形式来声明的，其方法名和返回值类型定义了该成员变量的名字和类型，而且还可以使用 `default` 关键字为这个成员变量设定默认值。

自定义注解的示例如下：
```java
package annotation.custom;

import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD, ElementType.TYPE})
public @interface Tag {

    String name() default "undefined";

    String description();

}
```

Java 使用 Annotation 接口来代表程序元素前面的注解，该接口是所有 Annotation 类型的父接口。自定义的注解继承了 `Annotation` 这个接口，因此自定义注解中包含了 `Annotation` 接口中所有的方法：
```java
public interface Annotation {

    boolean equals(Object obj);

    int hashCode();

    String toString();

    Class<? extends Annotation> annotationType();
}
```

## 注解的提取 ##
Java 在 `java.lang.reflect` 包下新增了 `AnnotatedElement` 接口，该接口代表程序中可以接受注解的程序元素，该接口主要有如下几个实现类：
 - `AccessibleObject：`是 Field、Method 和 Constructor 对象的基类。它提供了将反射的对象标记为在使用时取消默认 Java 语言访问控制检查的能力。
   - `Executable：`是 Method 和 Constructor 对象的基类，提供了 Method 和 Constructor 的常用功能。
     - `Method：`提供关于类或接口上单独某个方法（以及如何访问该方法）的信息。所反映的方法可能是类方法或实例方法（包括抽象方法）。
     - `Constructor：`提供关于类的单个构造方法的信息以及对它的访问权限。
   - `Field：`提供有关类或接口的单个字段的信息，以及对它的动态访问权限。反射的字段可能是一个类（静态）字段或实例字段。
 - `Class：`表示正在运行的 Java 应用程序中的类和接口。枚举是一种类，注释是一种接口。每个数组属于被映射为 Class 对象的一个类，所有具有相同元素类型和维数的数组都共享该 Class 对象。基本的 Java 类型(boolean、byte、char、short、int、long、float 和 double)和关键字 void 也表示为 Class 对象。
 - `Package：`包含有关 Java 包的实现和规范的版本信息。通过用于加载类的 ClassLoader 实例，可以获取并获得此版本信息。通常，此信息存储在与类一起分发的清单中。
 - `Parameter：`提供有关方法参数的信息，包括其名称和修饰符(Java8 新增)。

`java.lang.reflect` 包下主要包含一些实现反射功能的工具类，实际上，`java.lang.reflect` 包所有提供的反射 API 扩充了读取运行时 Annotation 信息的能力。当一个 Annotation 类型被定义为运行时的 Annotation 后，该注解才能是运行时可见，当 class 文件被装载时被保存在 class 文件中的 Annotation 才会被虚拟机读取。

AnnotatedElement 接口是所有程序元素(Class、Method、Constructor、Field、Parameter、Package)的父接口，所以程序通过反射获取了某个类的 `AnnotatedElement` 对象之后，程序就可以调用该对象的如下方法来访问 `Annotation` 信息：

| 方法                                                                                | 描述                                                                                                                              |
|-------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| `boolean isAnnotationPresent(Class<? extends Annotation> annotationType)`           | 判断指定对象是否应用了某个注解，此方法主要用于方便地访问标记注释                                                                  |
| `Annotation[] getAnnotations()`                                                     | 返回作用于指定对象的所有注解，不存在则返回长度为 0 的数组                                                                         |
| `Annotation[] getDeclaredAnnotations()`                                             | 返回直接作用于指定对象的所有注解，不存在则返回长度为 0 的数组(此方法忽略继承的注解)                                               |
| `<T extends Annotation> T getAnnotation(Class<T> annotationClass)`                  | 返回指定类型的注解，不存在则返回 null                                                                                             |
| `<T extends Annotation> T getDeclaredAnnotation(Class<T> annotationClass)`          | 返回直接作用于指定对象的指定类型的注解，不存在则返回 null(此方法忽略继承的注解)                                                   |
| `<T extends Annotation> T[] getAnnotationsByType(Class<T> annotationClass)`         | 返回指定类型的注解，不存在则返回长度为 0 的数组，此方法检测其参数是否为可重复的注解类型                                           |
| `<T extends Annotation> T[] getDeclaredAnnotationsByType(Class<T> annotationClass)` | 返回直接作用于指定对象的指定类型的注解，不存在则返回长度为 0 的数组，此方法检测其参数是否为可重复的注解类型(此方法忽略继承的注解) |

> 只有当定义 Annotation 时使用了 @Retention(RetentionPolicy.RUNTIME) 修饰，JVM 才会在装载 class 文件时提取保存在 class 文件中的 Annotation，该 Annotation 才会在运行时可见。否则 class 文件的注解信息在执行过程中将不可用，从而也就不能从中得到任何和注解有关的数据。

## 注解处理器 ##
注解处理器是(Annotation Processor)是 javac 内置的一个用于在编译时扫描、编译和处理注解(Annotation)的工具。简单的说，在源代码编译阶段，通过注解处理器，我们可以获取源文件内注解(Annotation)相关内容。

在 Java 5 首次引入注解的时候，注解处理器的 API 还没有成熟，也没有标准化。处理注解需要一个名为 `apt(Annotation Processor Tool，注解处理器工具)`的独立的工具，以及包含在 `com.sum.mirror` 包中的 Mirror API。`apt` 需要使用 Mirror API 来自定义处理器。

从 Java 6 开始，注解处理器通过 JSR 269 已经标准化并被纳入到标准库中，APT 工具也被无缝集成到 Java 编译工具 javac 里面。Java 6 提供了一个已经实现通用功能的抽象类 `javax.annotation.processing.AbstractProcessor`，同时还提供了 `javax.lang.model`包。

### 注解处理器的用途 ###
由于注解处理器可以在程序编译阶段工作，所以开发者可以在编译期间通过注解处理器进行开发者需要的操作。比较常用的用法就是在编译期间获取相关注解数据，然后动态生成 `.java` 源文件(让机器帮开发者写代码)，通常是自动产生一些有规律性的重复代码，解决了手工编写重复代码的问题，大大提升编码效率。

### 注解处理器的介绍 ###
`AbstractProcessor` 是一个抽象类，该类实现了 `Processor` 接口。抽象类 `AbstractProcessor` 以及接口 `Processor` 都是位于包 [`javax.annotation.processing`](http://tool.oschina.net/apidocs/apidoc?api=jdk-zh) 中。该包中定义的所有类、接口都是与实现注解处理器相关的。如下表所示： 

| 类/接口                    | 描述                                                                                                                                                |
|----------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| `Completion`               | 某一注释的建议 completion                                                                                                                           |
| `Filer`                    | 此接口支持通过注释处理器创建新文件                                                                                                                  |
| `Messager`                 | Messager 提供注释处理器用来报告错误消息、警告和其他通知的方式                                                                                       |
| `ProcessingEnvironment`    | 注释处理工具框架将提供一个具有实现此接口的对象的注释 processor，因此 processor 可以使用该框架提供的设施来编写新文件、报告错误消息并查找其他实用工具 |
| `Processor`                | 注释 Processor 的接口                                                                                                                               |
| `RoundEnvironment` 	     | 注释处理工具框架将提供一个注释处理器和一个实现此接口的对象，这样处理器可以查询有关注释处理的 round 的信息                                           |
| `AbstractProcessor`        | 旨在用作最具体注释 processor 的便捷超类的抽象注释 processor                                                                                         |
| `Completions`              | 用来组合 Completion 对象的实用工具类                                                                                                                |
| `FilerException`  	     | 指示一个检测到试图打开某一文件的 Filer，该文件违反 Filer 提供的保证                                                                                 |
| `SupportedAnnotationTypes` | 用来指示注释处理器支持哪些注释类型的注释                                                                                                            |

### 自定义注解处理器 ###
实现一个自定义注解处理器需要有两个步骤，第一是实现 `Processor` 接口处理注解，第二是注册注解处理器。

#### 实现 Processor 接口 ####
通过实现 `Processor` 接口可以自定义注解处理器，可以采用更简单的方法通过继承 `AbstractProcessor` 类实现自定义注解处理器，并实现抽象方法 `process` 处理想要的功能。

`AbstractProcessor` 具有如下方法：

| 方法                                                                                                                                     | 描述                                                                                                              |
|------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| `void init(ProcessingEnvironment processingEnv)`                                                                                         | 用处理环境初始化 processor，方法是将 processingEnv 字段设置为 processingEnv 参数的值                              |
| `boolean isInitialized()`                                                                                                                | 如果此对象已被初始化，则返回 true，否则返回 false                                                                 |
| `Iterable<? extends Completion> getCompletions(Element element, AnnotationMirror annotation, ExecutableElement member, String userText)` | 返回一个空的 completion 迭代                                                                                      |
| `Set<String> getSupportedAnnotationTypes()`                                                                                              | 如果 processor 类是使用 SupportedAnnotationTypes 注释的，则返回一个不可修改的集合，该集合具有与注释相同的字符串集 |
| `Set<String> getSupportedOptions()`                                                                                                      | 如果 processor 类是使用 SupportedOptions 注释的，则返回一个不可修改的集合，该集合具有与注释相同的字符串集         |
| `SourceVersion getSupportedSourceVersion()`                                                                                              | 如果 processor 类是使用 SupportedSourceVersion 注释的，则返回注释中的源版本                                       |
| `boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv)`                                                     | 处理先前 round 产生的类型元素上的注释类型集，并返回这些注释是否由此 Processor 声明                                |

> **注意：**在 Java 6 及以上版本中，`getSupportedAnnotationTypes()` 方法和 `getSupportedSourceVersion()` 方法分别可以通过 `@SupportedAnnotationTypes` 注解和 `@SupportedSourceVersion` 注解来替换。


自定义注解处理器都需要继承于 `AbstractProcessor`，如下所示：
```java
/**
 * 自定义注解处理器
 */
public class CustomProcessor extends AbstractProcessor {

    private Filer mFiler;
    private Messager mMessager;
    private Elements mElementUtils;

    @Override
    public synchronized void init(ProcessingEnvironment processingEnvironment) {
        super.init(processingEnvironment);
        mFiler = processingEnvironment.getFiler();
        mMessager = processingEnvironment.getMessager();
        mElementUtils = processingEnvironment.getElementUtils();
    }

    @Override
    public Set<String> getSupportedAnnotationTypes() {
        Set<String> annotataions = new LinkedHashSet<String>();
        annotataions.add(Tag.class.getCanonicalName());
        return annotataions;
    }

    @Override
    public SourceVersion getSupportedSourceVersion() {
        return SourceVersion.latestSupported();
    }

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnvironment) {
        Set<? extends Element> tagElements = roundEnvironment.getElementsAnnotatedWith(Tag.class);
        for (Element element : tagElements) {
            // 1.获取包名
            PackageElement packageElement = mElementUtils.getPackageOf(element);
            String pkName = packageElement.getQualifiedName().toString();
            printMessage(String.format("package = %s", pkName));
            // 2.获取包装类类型
            TypeElement enclosingElement = (TypeElement) element.getEnclosingElement();
            String enclosingName = enclosingElement.getQualifiedName().toString();
            printMessage(String.format("enclosindClass = %s", enclosingName));
            // 3.获取注解的成员变量名
            String tagFiledName = element.getSimpleName().toString();
            // 4.获取注解的成员变量类型
            String tagFiledClassType = element.asType().toString();
            // 5.获取注解元数据
            Tag tag = element.getAnnotation(Tag.class);
            String name = tag.name();
            printMessage(String.format("%s %s = %s", tagFiledClassType, tagFiledName, name));
            // 6.生成文件
            createFile(enclosingElement, tagFiledClassType, tagFiledName, name);
            return true;
        }
        return false;
    }

    private void createFile(TypeElement enclosingElement, String tagFiledClassType, String tagFiledName, String name) {
        String pkName = mElementUtils.getPackageOf(enclosingElement).getQualifiedName().toString();
        try {
            JavaFileObject javaFileObject = mFiler.createSourceFile(pkName + ".Tag");
            Writer writer = javaFileObject.openWriter();
            writer.write(generateCode(pkName, tagFiledClassType, tagFiledName, name));
            writer.flush();
            writer.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private void printMessage(String msg) {
        mMessager.printMessage(Diagnostic.Kind.NOTE, msg);
    }

    private String generateCode(String pkName, String tagFiledClassType, String tagFiledName, String name) {
        StringBuilder builder = new StringBuilder();
        builder.append("package " + pkName + ";\n\n");
        builder.append("//Auto generated by apt,do not modify!!\n\n");
        builder.append("public class Tag { \n\n");
        builder.append("public static void main(String[] args){ \n");
        String info = String.format("%s %s = %s", tagFiledClassType, tagFiledName, name);
        builder.append("System.out.println(\"" + info + "\");\n");
        builder.append("}\n");
        builder.append("}");
        return builder.toString();
    }

}
```

在 `init()` 中可以获得如下引用：
 - **`Elements：`**一个用来处理 `Element` 的工具类。
 - **`Types：`**一个用来处理 `TypeMirror` 的工具类。
 - **`Filer：`**正如这个名字所示，使用 `Filer` 可以创建文件。

在注解处理过程中，扫描所有的 Java 源文件。源代码的每一个部分都是一个特定类型的 `Element`。换句话说：`Element` 代表程序的元素，例如包、类或者方法。每个 `Element` 代表一个静态的、语言级别的构件。

`Element` 代表的是源代码，它的子类有这些：
 - PackageElement：表示一个包程序元素。提供对有关包及其成员的信息的访问。
 - TypeElement：表示一个类或接口程序元素。提供对有关类型及其成员的信息的访问。注意，枚举类型是一种类，而注解类型是一种接口。
 - VariableElement：表示一个成员变量、枚举常量、方法或构造方法参数、局部变量或异常参数。
 - ExecutableElement：表示某个类或接口的方法、构造方法或初始化程序（静态或实例），包括注释类型元素。
 - TypeParameterElement：表示一般类、接口、方法或构造方法元素的形式类型参数。类型参数声明一个 TypeVariable。

#### 注册注解处理器 ####
注册注解处理器有两种方法：
 - 在当前项目中的 `resources/META-INF/services` 目录需要新建一个特殊的文件 `javax.annotation.processing.Processor`，文件里的内容就是声明你的处理器。 `javax.annotation.processing.Processor` 文件的内容是合法全称，多个处理器之间换行。
 - Google 提供了一个注册处理器的库，添加 [`com.google.auto.service:auto-service`](https://github.com/google/auto/tree/master/service) 依赖并在自定义注解处理器上添加 `@AutoService(Processor.class)`， `AutoService` 注解会自动在 `META-INF` 文件夹下生成 `javax.annotation.processing.Processor` 配置信息文件，该文件里就是实现该服务接口的具体实现类。

## 注解的使用场景 ##
注解是一系列元数据，它提供数据用来解释程序代码，但是注解并非是所解释的代码本身的一部分。注解对于代码的运行效果没有直接影响。

注解有许多使用场景：
 - 类属性自动赋值。
 - 验证对象属性完整性。
 - 代替配置文件功能，像 Spring 基于注解的配置。
 - 可以生成文档，像 Java 代码注释中的 @see、@param 等

注解有许多用处，主要如下：
 - 提供信息给编译器： 编译器可以利用注解来探测错误和警告信息
 - 编译阶段时的处理： 软件工具可以用来利用注解信息来生成代码、Html 文档或者做其它相应处理。
 - 运行时的处理： 某些注解可以在程序运行的时候接受代码的提取

> 使用 Annotation 修饰了类、方法、成员变量等之后，这些 Annotation 不会自己生效，必须由开发者提供相应的代码来提取并处理 Annotation 信息。这些处理提取和处理 Annotation 的代码统称为 APT(Annotation Processing Tool)。

