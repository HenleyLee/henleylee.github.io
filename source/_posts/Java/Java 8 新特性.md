---
title: Java 8 新特性
categories: Java
tags:
  - Java
  - JDK
abbrlink: 409be0bd
date: 2018-11-12 18:45:38
---

Java 8 (又称为 jdk 1.8) 是 Java 语言开发的一个主要版本。Oracle 公司于 2014 年 3 月 18 日发布 Java 8 ，它支持函数式编程，新的 JavaScript 引擎，新的日期 API，新的 Stream API 等。

## Java 8 的新特性 ##
Java 8 新增了非常多的特性，主要特性包含以下几个：
 - **`Lambda 表达式`** − Lambda允许把函数作为一个方法的参数（函数作为参数传递进方法中。
 - **`方法引用`** − 方法引用提供了非常有用的语法，可以直接引用已有 Java 类或对象（实例）的方法或构造器。与 lambda 联合使用，方法引用可以使语言的构造更紧凑简洁，减少冗余代码。
 - **`默认方法`** − 默认方法就是一个在接口里面有了一个实现的方法。
 - **`新工具`** − 新的编译工具，如：Nashorn 引擎 jjs、 类依赖分析器 jdeps。
 - **`Stream API`** −新添加的 Stream API（java.util.stream） 把真正的函数式编程风格引入到 Java 中。
 - **`Date Time API`** − 加强对日期与时间的处理。
 - **`Optional 类`** − Optional 类已经成为 Java 8 类库的一部分，用来解决空指针异常。
 - **`Nashorn, JavaScript 引擎`** − Java 8 提供了一个新的 Nashorn javascript 引擎，它允许我们在 JVM 上运行特定的 javascript 应用。

Java 8 包含语言、编译器、库、工具和JVM等方面的十多个新特性。

更多的新特性可以参阅官网：[What's New in JDK 8](https://www.oracle.com/technetwork/java/javase/8-whats-new-2157071.html)
JDK 8 下载地址：[Java 8 Downloads](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

### Java语言的新特性 ###
#### Lambda 表达式和函数式接口 ####
Lambda 表达式（也称为闭包）是 Java 8 中最大和最令人期待的语言改变。它允许我们将函数当成参数传递给某个方法，或者把代码本身当作数据处理：[函数式开发者](https://www.javacodegeeks.com/2014/03/functional-programming-with-java-8-lambda-expressions-monads.html)非常熟悉这些概念。

很多 JVM 平台上的语言（[Groovy](https://www.javacodegeeks.com/category/jvm-languages/groovy)、[Scala](https://www.javacodegeeks.com/category/jvm-languages/scala)等）从诞生之日就支持 Lambda 表达式，但是 Java 开发者没有选择，只能使用匿名内部类代替 Lambda 表达式。

Lambda 的设计耗费了很多时间和很大的社区力量，最终找到一种折中的实现方案，可以实现简洁而紧凑的语言结构。

最简单的 Lambda 表达式可由`逗号分隔的参数列表`、`->符号`和`语句块`组成，例如：
```java
Arrays.asList("a", "b", "d").forEach(e -> System.out.println(e));
```

在上面这个代码中的参数 e 的类型是由编译器推理得出的，你也可以显式指定该参数的类型，例如：
```java
Arrays.asList("a", "b", "d").forEach((String e) -> System.out.println(e));
```

如果 Lambda 表达式需要更复杂的语句块，则可以使用花括号将该语句块括起来，类似于 Java 中的函数体，例如：
```java
Arrays.asList("a", "b", "d").forEach(e -> {
        System.out.print(e);
        System.out.print(e);
});
```
Lambda 表达式可以引用类成员和局部变量（会将这些变量隐式得转换成 `final` 的），例如下列两个代码块的效果完全相同：
```java
String separator = ",";
Arrays.asList("a", "b", "d").forEach(
        (String e) -> System.out.print(e + separator));
```
和
```java
final String separator = ",";
Arrays.asList("a", "b", "d").forEach(
        (String e) -> System.out.print(e + separator));
```

Lambda 表达式有返回值，返回值的类型也由编译器推理得出。如果 Lambda 表达式中的语句块只有一行，则可以不用使用 `return` 语句，下列两个代码片段效果相同：
```java
Arrays.asList("a", "b", "d").sort((e1, e2) -> e1.compareTo(e2));
```
和
```java
Arrays.asList("a", "b", "d").sort((e1, e2) -> {
    int result = e1.compareTo(e2);
    return result;
});
```

Lambda 的设计者们为了让现有的功能与 Lambda 表达式良好兼容，考虑了很多方法，于是产生了[函数接口](https://www.javacodegeeks.com/2013/03/introduction-to-functional-interfaces-a-concept-recreated-in-java-8.html)这个概念。函数接口指的是只有一个函数的接口，这样的接口可以隐式转换为 Lambda 表达式。`java.lang.Runnable` 和 `java.util.concurrent.Callable` 是函数式接口的最佳例子。在实践中，函数式接口非常脆弱：只要某个开发者在该接口中添加一个函数，则该接口就不再是函数式接口进而导致编译失败。为了克服这种代码层面的脆弱性，并显式说明某个接口是函数式接口，Java 8 提供了一个特殊的注解 `@FunctionalInterface`（Java 库中的所有相关接口都已经带有这个注解了），举个简单的函数式接口的定义：
```java
@FunctionalInterface
public interface Functional {
    void method();
}
```

不过有一点需要注意，默认方法和静态方法不会破坏函数式接口的定义，因此如下的代码是合法的。
```java
@FunctionalInterface
public interface FunctionalDefaultMethods {
    void method();

    default void defaultMethod() {
    }
}
```

Lambda 表达式作为 Java 8 的最大卖点，它有潜力吸引更多的开发者加入到 JVM 平台，并在纯 Java 编程中使用函数式编程的概念。如果你需要了解更多 Lambda 表达式的细节，可以参考[官方文档](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)。

#### 接口的默认方法和静态方法 ####
Java 8 使用两个新概念扩展了接口的含义：默认方法和静态方法。默认方法使得接口有点类似 traits，不过要实现的目标不一样。默认方法使得开发者可以在 不破坏二进制兼容性的前提下，往现存接口中添加新的方法，即不强制那些实现了该接口的类也同时实现这个新加的方法。

默认方法和抽象方法之间的区别在于抽象方法需要实现，而默认方法不需要。接口提供的默认方法会被接口的实现类继承或者覆写，例子代码如下：
```java
private interface Defaulable {
    // Interfaces now allow default methods, the implementer may or
    // may not implement (override) them.
    default String notRequired() {
        return "Default implementation";
    }
}
```
```java
private static class DefaultableImpl implements Defaulable {
}
```
```java
private static class OverridableImpl implements Defaulable {
    @Override
    public String notRequired() {
        return "Overridden implementation";
    }
}
```

`Defaulable` 接口使用关键字 `default` 定义了一个默认方法 `notRequired()`。`DefaultableImpl` 类实现了这个接口，同时默认继承了这个接口中的默认方法；`OverridableImpl` 类也实现了这个接口，但覆写了该接口的默认方法，并提供了一个不同的实现。

Java 8 带来的另一个有趣的特性是在接口中可以定义静态方法，例子代码如下：
```java
private interface DefaulableFactory {
    // Interfaces now allow static methods
    static Defaulable create(Supplier<Defaulable> supplier) {
        return supplier.get();
    }
}
```

下面的代码片段整合了默认方法和静态方法的使用场景：
```java
public static void main(String[] args) {
    Defaulable defaulable = DefaulableFactory.create(DefaultableImpl::new);
    System.out.println(defaulable.notRequired());

    defaulable = DefaulableFactory.create(OverridableImpl::new);
    System.out.println(defaulable.notRequired());
}
```

这段代码的输出结果如下：
```shell
Default implementation
Overridden implementation
```

由于 JVM 上的默认方法的实现在字节码层面提供了支持，因此效率非常高。默认方法允许在不打破现有继承体系的基础上改进接口。该特性在官方库中的应用是：给 `java.util.Collection` 接口添加新方法，如 `stream()`、`parallelStream()`、`forEach()` 和 `removeIf()` 等等。

尽管默认方法有这么多好处，但在实际开发中应该谨慎使用：在复杂的继承体系中，默认方法可能引起歧义和编译错误。如果你想了解更多细节，可以参考[官方文档](https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html)。

#### 方法引用 ####
方法引用使得开发者可以直接引用现存的方法、Java 类的构造方法或者实例对象。方法引用和 Lambda 表达式配合使用，使得 Java 类的构造方法看起来紧凑而简洁，没有很多复杂的模板代码。

西门的例子中，**Car** 类是不同方法引用的例子，可以帮助读者区分四种类型的方法引用：
```java
public static class Car {
    public static Car create(final Supplier<Car> supplier) {
        return supplier.get();
    }

    public static void collide(final Car car) {
        System.out.println("Collided " + car.toString());
    }

    public void follow(final Car another) {
        System.out.println("Following the " + another.toString());
    }

    public void repair() {
        System.out.println("Repaired " + this.toString());
    }
}
```

第一种方法引用的类型是构造器引用，语法是 `Class::new`，或者更一般的形式：`Class<T>::new`。注意：这个构造器没有参数。
```java
final Car car = Car.create(Car::new);
final List<Car> cars = Arrays.asList(car);
```

第二种方法引用的类型是静态方法引用，语法是 `Class::static_method`。注意：这个方法接受一个 Car 类型的参数。
```java
cars.forEach(Car::collide);
```

第三种方法引用的类型是某个类的成员方法的引用，语法是 `Class::method`，注意，这个方法没有定义入参：
```java
cars.forEach(Car::repair);
```

第四种方法引用的类型是某个实例对象的成员方法的引用，语法是 `instance::method`。注意：这个方法接受一个 Car 类型的参数：
```java
final Car police = Car.create(Car::new);
cars.forEach(police::follow);
```

运行上述例子，可以在控制台看到如下输出（Car实例可能不同）：
```shell
Collided com.javacodegeeks.java8.method.references.MethodReferences$Car@7a81197d
Repaired com.javacodegeeks.java8.method.references.MethodReferences$Car@7a81197d
Following the com.javacodegeeks.java8.method.references.MethodReferences$Car@7a81197d
```

如果想了解和学习更详细的内容，可以参考[官方文档](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)。

#### 重复注解 ####
自从 Java 5 中引入[注解](https://www.javacodegeeks.com/2012/08/java-annotations-explored-explained.html)以来，这个特性开始变得非常流行，并在各个框架和项目中被广泛使用。不过，注解有一个很大的限制是：在同一个地方不能多次使用同一个注解。Java 8 打破了这个限制，引入了重复注解的概念，允许在同一个地方多次使用同一个注解。

在 Java 8 中使用 `@Repeatable` 注解定义重复注解，实际上，这并不是语言层面的改进，而是编译器做的一个 trick，底层的技术仍然相同。可以利用下面的代码说明：
```java
package com.javacodegeeks.java8.repeatable.annotations;

import java.lang.annotation.ElementType;
import java.lang.annotation.Repeatable;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

public class RepeatingAnnotations {

    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface Filters {
        Filter[] value();
    }

    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Repeatable(Filters.class)
    public @interface Filter {
        String value();
    }

    @Filter("filter1")
    @Filter("filter2")
    public interface Filterable {
    }

    public static void main(String[] args) {
        for (Filter filter : Filterable.class.getAnnotationsByType(Filter.class)) {
            System.out.println(filter.value());
        }
    }

}
```

正如我们所见，这里的 `Filter` 类使用 `@Repeatable(Filters.class)` 注解修饰，而 `Filters` 是存放 `Filter` 注解的容器，编译器尽量对开发者屏蔽这些细节。这样，`Filterable` 接口可以用两个 `Filter` 注解注释（这里并没有提到任何关于Filters的信息）。

另外，反射 API 提供了一个新的方法：`getAnnotationsByType()`，可以返回某个类型的重复注解，例如 Filterable.class.getAnnoation(Filters.class) 将返回两个 Filter 实例，输出到控制台的内容如下所示：
```shell
filter1
filter2
```

如果你希望了解更多内容，可以参考[官方文档](https://docs.oracle.com/javase/tutorial/java/annotations/repeating.html)。

#### 更好的类型推断 ####
Java 8 编译器在类型推断方面有很大的提升，在很多场景下编译器可以推导出某个参数的数据类型，从而使得代码更为简洁。例子代码如下：
```java
package com.javacodegeeks.java8.type.inference;

public class Value<T> {
    public static <T> T defaultValue() {
        return null;
    }

    public T getOrDefault(T value, T defaultValue) {
        return (value != null) ? value : defaultValue;
    }
}
```

下列代码是 `Value<String>` 类型的应用：
```java
package com.javacodegeeks.java8.type.inference;

public class TypeInference {
    public static void main(String[] args) {
        final Value<String> value = new Value<>();
        value.getOrDefault("22", Value.defaultValue());
    }
}
```

参数 `Value.defaultValue()` 的类型由编译器推导得出，不需要显式指明。在 Java 7 中这段代码会有编译错误，除非使用 `Value.<String>defaultValue()`。

#### 拓宽注解的应用场景 ####
Java 8 拓宽了注解的应用场景。现在，注解几乎可以使用在任何元素上：局部变量、接口类型、超类和接口实现类，甚至可以用在函数的异常定义上。下面是一些例子：
```java
package com.javacodegeeks.java8.annotations;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.util.ArrayList;
import java.util.Collection;

public class Annotations {
    @Retention(RetentionPolicy.RUNTIME)
    @Target({ElementType.TYPE_USE, ElementType.TYPE_PARAMETER})
    public @interface NonEmpty {
    }

    public static class Holder<@NonEmpty T> extends @NonEmpty Object {
        public void method() throws @NonEmpty Exception {
        }
    }

    @SuppressWarnings("unused")
    public static void main(String[] args) {
        final Holder<String> holder = new @NonEmpty Holder<String>();
        @NonEmpty Collection<@NonEmpty String> strings = new ArrayList<>();
    }
}
```

`ElementType.TYPE_USER` 和 `ElementType.TYPE_PARAMETER` 是 Java 8 新增的两个注解，用于描述注解的使用场景。Java 语言也做了对应的改变，以识别这些新增的注解。

### Java 编译器的新特性 ###
#### 参数名称 ####
为了在运行时获得 Java 程序中方法的参数名称，老一辈的 Java 程序员必须使用不同方法，例如 [Paranamer liberary](https://github.com/paul-hammant/paranamer)。Java 8 终于将这个特性规范化，在语言层面（使用反射API和Parameter.getName()方法）和字节码层面（使用新的javac编译器以及-parameters参数）提供支持。
```java
package com.javacodegeeks.java8.parameter.names;

import java.lang.reflect.Method;
import java.lang.reflect.Parameter;

public class ParameterNames {
    public static void main(String[] args) throws Exception {
        Method method = ParameterNames.class.getMethod("main", String[].class);
        for (final Parameter parameter : method.getParameters()) {
            System.out.println("Parameter: " + parameter.getName());
        }
    }
}
```

在 Java 8 中这个特性是默认关闭的，因此如果不带 `-parameters` 参数编译上述代码并运行，则会输出如下结果：
```shell
Parameter: arg0
```

如果带 `-parameters` 参数，则会输出如下结果（正确的结果）：
```shell
Parameter: args
```

如果你使用 Maven 进行项目管理，则可以在 `maven-compiler-plugin` 编译器的配置项中配置 `-parameters` 参数：
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.1</version>
    <configuration>
        <compilerArgument>-parameters</compilerArgument>
        <source>1.8</source>
        <target>1.8</target>
    </configuration>
</plugin>
```

### Java 官方库的新特性 ###
Java 8 增加了很多新的工具类（date/time类），并扩展了现存的工具类，以支持现代的并发编程、函数式编程等。
#### Optional ####
Java 应用中最常见的 bug 就是[空值异常](https://examples.javacodegeeks.com/java-basics/exceptions/java-lang-nullpointerexception-how-to-handle-null-pointer-exception/)。在 Java 8 之前，[Google Guava](https://github.com/google/guava) 引入了 `Optionals` 类来解决 `NullPointerException`，从而避免源码被各种 `null` 检查污染，以便开发者写出更加整洁的代码。Java 8 也将 `Optional` 加入了官方库。

`Optional` 仅仅是一个容器：存放 `T` 类型的值或者 `null`，它提供了一些有用的方法来避免显式的 `null` 检查，可以参考 [Java 8 官方文档](https://docs.oracle.com/javase/8/docs/api/)了解更多细节。

接下来看一点使用 `Optional` 的例子：可能为空的值或者某个类型的值：
```java
Optional<String> fullName = Optional.ofNullable(null);
System.out.println("Full Name is set? " + fullName.isPresent());
System.out.println("Full Name: " + fullName.orElseGet(() -> "[none]"));
System.out.println(fullName.map(s -> "Hey " + s + "!").orElse("Hey Stranger!"));
```

如果 `Optional` 实例持有一个非空值，则 `isPresent()` 方法返回 true，否则返回 false；`orElseGet()` 方法，`Optional` 实例持有 `null`，则可以接受一个 lambda 表达式生成的默认值；`map()` 方法可以将现有的 `Opetional` 实例的值转换成新的值；`orElse()` 方法与 `orElseGet()` 方法类似，但是在持有null的时候返回传入的默认值。

上述代码的输出结果如下：
```shell
Full Name is set? false
Full Name: [none]
Hey Stranger!
```

再看下另一个简单的例子：
```java
Optional<String> firstName = Optional.of("Tom");
System.out.println("First Name is set? " + firstName.isPresent());
System.out.println("First Name: " + firstName.orElseGet(() -> "[none]"));
System.out.println(firstName.map(s -> "Hey " + s + "!").orElse("Hey Stranger!"));
System.out.println();
```

这个例子的输出是：
```shell
First Name is set? true
First Name: Tom
Hey Tom!
```
如果想了解更多的细节，请参考[官方文档](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)。

#### Streams API ####
新增的 [Stream API](https://www.javacodegeeks.com/2014/05/the-effects-of-programming-with-java-8-streams-on-algorithm-performance.html)（java.util.stream）将生成环境的函数式编程引入了 Java 库中。这是目前为止最大的一次对 Java 库的完善，以便开发者能够写出更加有效、更加简洁和紧凑的代码。

Steam API 极大得简化了集合操作（后面我们会看到不止是集合），首先看下这个叫 Task 的类：
```java
public class Streams {

    private enum Status {
        OPEN, CLOSED
    }

    private static final class Task {
        private final Status status;
        private final Integer points;

        Task(final Status status, final Integer points) {
            this.status = status;
            this.points = points;
        }

        public Integer getPoints() {
            return points;
        }

        public Status getStatus() {
            return status;
        }

        @Override
        public String toString() {
            return String.format("[%s, %d]", status, points);
        }
    }

}
```

Task 类有一个分数（或伪复杂度）的概念，另外还有两种状态：OPEN 或者 CLOSED。现在假设有一个 task 集合：
```java
final Collection<Task> tasks = Arrays.asList(
        new Task(Status.OPEN, 5),
        new Task(Status.OPEN, 13),
        new Task(Status.CLOSED, 8)
);
```

首先看一个问题：在这个 task 集合中一共有多少个 OPEN 状态的点？在 Java 8 之前，要解决这个问题，则需要使用 `foreach` 循环遍历 task 集合；但是在 Java 8 中可以利用 `steams` 解决：包括一系列元素的列表，并且支持顺序和并行处理。
```java
// Calculate total points of all active tasks using sum()
final long totalPointsOfOpenTasks = tasks
        .stream()
        .filter(task -> task.getStatus() == Status.OPEN)
        .mapToInt(Task::getPoints)
        .sum();

System.out.println("Total points: " + totalPointsOfOpenTasks);
```

运行这个方法的控制台输出是：
```shell
Total points: 18
```

这里有很多知识点值得说。首先，tasks 集合被转换成 `steam` 表示；其次，在 steam 上的 `filter` 操作会过滤掉所有 CLOSED 的 task；第三，`mapToInt` 操作基于每个 task 实例的 `Task::getPoints` 方法将 task 流转换成 Integer 集合；最后，通过 `sum` 方法计算总和，得出最后的结果。

在学习下一个例子之前，还需要记住一些 steams（[点此更多细节](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html#StreamOps)）的知识点。Steam 之上的操作可分为中间操作和晚期操作。

中间操作会返回一个新的 steam —— 执行一个中间操作（例如filter）并不会执行实际的过滤操作，而是创建一个新的 steam，并将原 steam 中符合条件的元素放入新创建的 steam。

晚期操作（例如 `forEach` 或者 `sum`），会遍历 steam 并得出结果或者附带结果；在执行晚期操作之后，steam 处理线已经处理完毕，就不能使用了。在几乎所有情况下，晚期操作都是立刻对 steam 进行遍历。

steam 的另一个价值是创造性地支持并行处理（parallel processing）。对于上述的 tasks 集合，我们可以用下面的代码计算所有任务的点数之和：
```java
// Calculate total points of all tasks
final double totalPoints = tasks
        .stream()
        .parallel()
        .map(task -> task.getPoints()) // or map( Task::getPoints )
        .reduce(0, Integer::sum);

System.out.println("Total points (all tasks): " + totalPoints);
```

这里我们使用 `parallel` 方法并行处理所有的 task，并使用 `reduce` 方法计算最终的结果。控制台输出如下：
```shell
Total points（all tasks）: 26.0
```

对于一个集合，经常需要根据某些条件对其中的元素分组。利用 steam 提供的 API 可以很快完成这类任务，代码如下：
```java
// Group tasks by their status
final Map<Status, List<Task>> map = tasks
        .stream()
        .collect(Collectors.groupingBy(Task::getStatus));

System.out.println(map);
```

控制台的输出如下：
```shell
{CLOSED=[[CLOSED, 8]], OPEN=[[OPEN, 5], [OPEN, 13]]}
```

最后一个关于 tasks 集合的例子问题是：如何计算集合中每个任务的点数在集合中所占的比重，具体处理的代码如下：
```java
// Calculate the weight of each tasks (as percent of total points) 
final Collection<String> result = tasks
        .stream()                                      // Stream< String >
        .mapToInt(Task::getPoints)                     // IntStream
        .asLongStream()                                // LongStream
        .mapToDouble(points -> points / totalPoints)   // DoubleStream
        .boxed()                                       // Stream< Double >
        .mapToLong(weigth -> (long) (weigth * 100))    // LongStream
        .mapToObj(percentage -> percentage + "%")      // Stream< String>
        .collect(Collectors.toList());                 // List< String >

System.out.println(result);
```

控制台输出结果如下：
```shell
[19%, 50%, 30%]
```

最后，正如之前所说，Steam API 不仅可以作用于 Java 集合，传统的 IO 操作（从文件或者网络一行一行得读取数据）可以受益于 steam 处理，这里有一个小例子：
```java
final Path path = new File(filename).toPath();
try (Stream<String> lines = Files.lines(path, StandardCharsets.UTF_8)) {
        lines.onClose(() -> System.out.println("Done!")).forEach(System.out::println);
}
```

Stream 的方法 `onClose` 返回一个等价的有额外句柄的 Stream，当 Stream 的 `close()` 方法被调用的时候这个句柄会被执行。Stream API、Lambda 表达式还有接口默认方法和静态方法支持的方法引用，是 Java 8 对软件开发的现代范式的响应。

#### Date/Time API ####
Java 8 引入了[新的 Date-Time API(JSR 310)](https://jcp.org/en/jsr/detail?id=310)来改进时间、日期的处理。时间和日期的管理一直是最令 Java 开发者痛苦的问题。`java.util.Date` 和后来的 `java.util.Calendar` 一直没有解决这个问题（甚至令开发者更加迷茫）。

因为上面这些原因，诞生了第三方库 [Joda-Time](https://www.joda.org/joda-time/)，可以替代 Java 的时间管理 API。Java 8 中新的时间和日期管理 API 深受 Joda-Time 影响，并吸收了很多 Joda-Time 的精华。新的 java.time 包包含了所有关于日期、时间、时区、Instant（跟日期类似但是精确到纳秒）、duration（持续时间）和时钟操作的类。新设计的API认真考虑了这些类的不变性（从 java.util.Calendar 吸取的教训），如果某个实例需要修改，则返回一个新的对象。

我们接下来看看 java.time 包中的关键类和各自的使用例子。
首先，`Clock` 类使用时区来返回当前的纳秒时间和日期。`Clock` 可以替代 `System.currentTimeMillis()` 和 `TimeZone.getDefault()`。
```java
// Get the system clock as UTC offset 
final Clock clock = Clock.systemUTC();
System.out.println(clock.instant());
System.out.println(clock.millis());
```

这个例子的输出结果是：
```shell
2014-04-12T15:19:29.282Z
1397315969360
```

第二，关注下 `LocalDate` 和 `LocalTime` 类。`LocalDate` 仅仅包含 ISO-8601 日历系统中的日期部分；`LocalTime` 则仅仅包含该日历系统中的时间部分。这两个类的对象都可以使用 `Clock` 对象构建得到。
```java
// Get the local date and local time
final LocalDate date = LocalDate.now();
final LocalDate dateFromClock = LocalDate.now(clock);

System.out.println(date);
System.out.println(dateFromClock);

// Get the local date and local time
final LocalTime time = LocalTime.now();
final LocalTime timeFromClock = LocalTime.now(clock);

System.out.println(time);
System.out.println(timeFromClock);
```

上述例子的输出结果如下：
```shell
2014-04-12
2014-04-12
11:25:54.568
15:25:54.568
```

`LocalDateTime` 类包含了 `LocalDate` 和 `LocalTime` 的信息，但是不包含 ISO-8601 日历系统中的时区信息。这里有一些[关于 LocalDate 和 LocalTime 的例子](https://www.javacodegeeks.com/2014/04/java-8-date-time-api-tutorial-localdatetime.html)：
```java
// Get the local date/time
final LocalDateTime datetime = LocalDateTime.now();
final LocalDateTime datetimeFromClock = LocalDateTime.now(clock);

System.out.println(datetime);
System.out.println(datetimeFromClock);
```

上述这个例子的输出结果如下：
```shell
2014-04-12T11:37:52.309
2014-04-12T15:37:52.309
```

如果你需要特定时区的 data/time 信息，则可以使用 `ZoneDateTime`，它保存有 ISO-8601 日期系统的日期和时间，而且有时区信息。下面是一些使用不同时区的例子：
```java
// Get the zoned date/time
final ZonedDateTime zonedDatetime = ZonedDateTime.now();
final ZonedDateTime zonedDatetimeFromClock = ZonedDateTime.now(clock);
final ZonedDateTime zonedDatetimeFromZone = ZonedDateTime.now(ZoneId.of("America/Los_Angeles"));

System.out.println(zonedDatetime);
System.out.println(zonedDatetimeFromClock);
System.out.println(zonedDatetimeFromZone);
```

这个例子的输出结果是：
```shell
2014-04-12T11:47:01.017-04:00[America/New_York]
2014-04-12T15:47:01.017Z
2014-04-12T08:47:01.017-07:00[America/Los_Angeles]
```

最后看下 `Duration` 类，它持有的时间精确到秒和纳秒。这使得我们可以很容易得计算两个日期之间的不同，例子代码如下：
```java
// Get duration between two dates
final LocalDateTime from = LocalDateTime.of(2014, Month.APRIL, 16, 0, 0, 0);
final LocalDateTime to = LocalDateTime.of(2015, Month.APRIL, 16, 23, 59, 59);

final Duration duration = Duration.between(from, to);
System.out.println("Duration in days: " + duration.toDays());
System.out.println("Duration in hours: " + duration.toHours());
```

这个例子用于计算2014年4月16日和2015年4月16日之间的天数和小时数，输出结果如下：
```shell
Duration in days: 365
Duration in hours: 8783
```

对于 Java 8 的新日期时间的总体印象还是比较积极的，一部分是因为 Joda-Time 的积极影响，另一部分是因为官方终于听取了开发人员的需求。如果希望了解更多细节，可以参考[官方文档](https://docs.oracle.com/javase/tutorial/datetime/index.html)。

#### Nashorn JavaScript 引擎 ####
Java 8 提供了新的 [Nashorn JavaScript 引擎](https://www.javacodegeeks.com/2014/02/java-8-compiling-lambda-expressions-in-the-new-nashorn-js-engine.html)，使得我们可以在 JVM 上开发和运行 JS 应用。Nashorn JavaScript 引擎是 javax.script.ScriptEngine 的另一个实现版本，这类 Script 引擎遵循相同的规则，允许 Java 和 JavaScript 交互使用，例子代码如下：
```java
ScriptEngineManager manager = new ScriptEngineManager();
ScriptEngine engine = manager.getEngineByName("JavaScript");

System.out.println(engine.getClass().getName());
System.out.println("Result:" + engine.eval("function f() { return 1; }; f() + 1;"));
```

这个代码的输出结果如下：
```shell
jdk.nashorn.api.scripting.NashornScriptEngine
Result: 2
```

#### Base64 ####
[对 Base64 编码的支持](https://www.javacodegeeks.com/2014/04/base64-in-java-8-its-not-too-late-to-join-in-the-fun.html)已经被加入到 Java 8 官方库中，这样不需要使用第三方库就可以进行 Base64 编码，例子代码如下：
```java
package com.javacodegeeks.java8.base64;

import java.nio.charset.StandardCharsets;
import java.util.Base64;

public class Base64s {
    public static void main(String[] args) {
        final String text = "Base64 finally in Java 8!";

        final String encoded = Base64
                .getEncoder()
                .encodeToString(text.getBytes(StandardCharsets.UTF_8));
        System.out.println(encoded);

        final String decoded = new String(
                Base64.getDecoder().decode(encoded),
                StandardCharsets.UTF_8);
        System.out.println(decoded);
    }
}
```

这个例子的输出结果如下：
```shell
QmFzZTY0IGZpbmFsbHkgaW4gSmF2YSA4IQ==
Base64 finally in Java 8!
```

新的 Base64 API 也支持 URL 和 MINE 的编码解码(`Base64.getUrlEncoder()` / `Base64.getUrlDecoder()`, `Base64.getMimeEncoder()` / `Base64.getMimeDecoder()`)。

BASE64不是用来加密的，是BASE64编码后的字符串，全部都是由标准键盘上面的常规字符组成，这样编码后的字符串在网关之间传递不会产生UNICODE字符串不能识别或者丢失的现象。你再仔细研究下EMAIL就会发现其实EMAIL就是用base64编码过后再发送的。然后接收的时候再还原。

#### 并行数组 ####
Java 8 版本新增了很多新的方法，用于支持并行数组处理。最重要的方法是 `parallelSort()`，可以显著加快多核机器上的数组排序。下面的例子论证了 `parallexXxx` 系列的方法：
```java
package com.javacodegeeks.java8.parallel.arrays;

import java.util.Arrays;
import java.util.concurrent.ThreadLocalRandom;

public class ParallelArrays {
    public static void main(String[] args) {
        long[] arrayOfLong = new long[20000];

        Arrays.parallelSetAll(arrayOfLong,
                index -> ThreadLocalRandom.current().nextInt(1000000));
        Arrays.stream(arrayOfLong).limit(10).forEach(
                i -> System.out.print(i + " "));
        System.out.println();

        Arrays.parallelSort(arrayOfLong);
        Arrays.stream(arrayOfLong).limit(10).forEach(
                i -> System.out.print(i + " "));
        System.out.println();
    }
}
```

上述这些代码使用 `parallelSetAll()` 方法生成20000个随机数，然后使用 `parallelSort()` 方法进行排序。这个程序会输出乱序数组和排序数组的前10个元素。上述例子的代码输出的结果是：
```shell
Unsorted: 591217 891976 443951 424479 766825 351964 242997 642839 119108 552378 
Sorted: 39 220 263 268 325 607 655 678 723 793
```

#### 并发性 ####
基于新增的 lambda 表达式和 steam 特性，为 Java 8 中为 `java.util.concurrent.ConcurrentHashMap` 类添加了新的方法来支持聚焦操作；另外，也为 `java.util.concurrentForkJoinPool` 类添加了新的方法来支持通用线程池操作（更多内容可以参考我们的[并发编程课程](academy.javacodegeeks.com/course/java-concurrency-essentials/)）。

Java 8 还添加了新的 `java.util.concurrent.locks.StampedLock` 类，用于支持基于容量的锁——该锁有三个模型用于支持读写操作（可以把这个锁当做是 `java.util.concurrent.locks.ReadWriteLock` 的替代者）。

在 `java.util.concurrent.atomic` 包中也新增了不少工具类，列举如下：
 - DoubleAccumulator
 - DoubleAdder
 - LongAccumulator
 - LongAdder

### 新的 Java 工具 ###
Java 8 提供了一些新的命令行工具，这部分会讲解一些对开发者最有用的工具。

#### Nashorn 引擎：jjs ####
`jjs` 是一个基于标准 Nashorn 引擎的命令行工具，可以接受 js 源码并执行。例如，我们写一个 `func.js` 文件，内容如下：
```javascript
function f() {
     return 1;
};

print(f() + 1);
```

可以在命令行中执行这个命令：jjs func.js，控制台输出结果是：
```shell
2
```

如果需要了解细节，可以参考[官方文档](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jjs.html)。

#### 类依赖分析器：jdeps ####
`jdeps` 是一个相当棒的命令行工具，它可以展示包层级和类层级的 Java 类依赖关系，它以 `.class` 文件、目录或者 JAR 文件为输入，然后会把依赖关系输出到控制台。

我们可以利用 jedps 分析下 Spring Framework 库，为了让结果少一点，仅仅分析一个 JAR 文件：`org.springframework.core-3.0.5.RELEASE.jar`。
```shell
jdeps org.springframework.core-3.0.5.RELEASE.jar
```

这个命令会输出很多结果，我们仅看下其中的一部分：依赖关系按照包分组，如果在 classpath 上找不到依赖，则显示 "not found"。
```shell
org.springframework.core-3.0.5.RELEASE.jar -> C:\Program Files\Java\jdk1.8.0\jre\lib\rt.jar
   org.springframework.core (org.springframework.core-3.0.5.RELEASE.jar)
      -> java.io                                            
      -> java.lang                                          
      -> java.lang.annotation                               
      -> java.lang.ref                                      
      -> java.lang.reflect                                  
      -> java.util                                          
      -> java.util.concurrent                               
      -> org.apache.commons.logging                         not found
      -> org.springframework.asm                            not found
      -> org.springframework.asm.commons                    not found
   org.springframework.core.annotation (org.springframework.core-3.0.5.RELEASE.jar)
      -> java.lang                                          
      -> java.lang.annotation                               
      -> java.lang.reflect                                  
      -> java.util
```

更多的细节可以参考[官方文档](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jdeps.html)。

### JVM的新特性 ###
使用 [Metaspace](https://www.javacodegeeks.com/2013/02/java-8-from-permgen-to-metaspace.html)（[JEP 122](http://openjdk.java.net/jeps/122)）代替持久代（`PermGen space`）。在 JVM 参数方面，使用 `-XX:MetaSpaceSize` 和 `-XX:MaxMetaspaceSize` 代替原来的 `-XX:PermSize` 和 `-XX:MaxPermSize`。

## 参考资源 ##
 - 参考 [Java 8 官方文档](https://docs.oracle.com/javase/8/)，了解 Java 8 的更多内容。
 - 参考 [Java 8 API 文档](https://docs.oracle.com/javase/8/docs/api/index.html)，了解 Java 8 API 的细节。 
 - 参考 [What's New in JDK 8](https://www.oracle.com/technetwork/java/javase/8-whats-new-2157071.html)，了解 Java 8 的新特性。
 - 参考 [Java 8 Release Notes](https://www.oracle.com/technetwork/java/javase/8u-relnotes-2225394.html)，了解 Java 8 的更新说明。
