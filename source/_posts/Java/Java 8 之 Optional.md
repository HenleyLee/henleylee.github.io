---
title: Java 8 之 Optional
date: 2018-11-29 19:45:22
categories: Java
tags:
  - Java
---

## 概述 ##
Java 应用中最常见的 bug 就是[空值异常](https://examples.javacodegeeks.com/java-basics/exceptions/java-lang-nullpointerexception-how-to-handle-null-pointer-exception/)。在 Java 8 之前，[Google Guava](https://github.com/google/guava) 引入了 `Optionals` 类来解决 `NullPointerException`，从而避免源码被各种 `null` 检查污染，以便开发者写出更加整洁的代码。Java 8 也将 `Optional` 加入了官方库。

`Optional` 仅仅是一个容器：存放 `T` 类型的值或者 `null`，它提供了一些有用的方法来避免显式的 `null` 检查，可以参考 [Java 8 官方文档](https://docs.oracle.com/javase/8/docs/api/)了解更多细节。

## Optional 的方法 ##
`java.util.Optional` 类提供了很多有用的方法：

| 方法                                                                           | 描述                                                                                                                                           |
|--------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| `static <T> Optional<T> empty()`                                                | 返回空的 Optional 实例                                                                                                                         |
| `static <T> Optional<T> of(T value)`                                           | 返回一个描述指定非 null 值的 Optional 实例                                                                                                     |
| `static <T> Optional<T> ofNullable(T value)`                                   | 如果指定的值为非空，则返回具有指定值的 Optional 实例，返回空的 Optional 实例                                                                   |
| `T get()`                                                                      | 如果在这个 Optional 中存在一个非 null 值，则返回该值，否则抛出 NoSuchElementException                                                          |
| `boolean isPresent()`                                                          | 如果存在值，则返回true，否则返回false                                                                                                          |
| `void ifPresent(Consumer<? super T> consumer)`                                 | 如果值存在，则使用该值调用指定的 consumer，否则不做任何事情                                                                                    |
| `Optional<T> filter(Predicate<? super T> predicate)`                           | 如果值存在，并且这个值匹配给定的 predicate，则返回描述该值的 Optional 实例，否则返回一个空的 Optional 实例                                     |
| `<U> Optional<U> map(Function<? super T, ? extends U> mapper)`                 | 如果值存在，则使用该值调用指定的 mapper；如果返回值不为 null，则返回描述返回值的 Optional 实例，否则返回一个空的 Optional 实例                 |
| `<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper)`             | 如果值存在，则使用该值调用指定的 mapper；如果返回值不为 null，则返回基于 Optional 的描述返回值的 Optional 实例，否则返回一个空的 Optional 实例 |
| `T orElse(T other)`                                                            | 如果值存在，则返回该值，否则返回 other                                                                                                         |
| `T orElseGet(Supplier<? extends T> other)`                                     | 如果值存在，则返回该值，调用 other 并返回该调用的结果                                                                                          |
| `<X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier)` | 如果值存在，则返回该值，否则抛出由 Supplier 提供的异常                                                                                         |
| `boolean equals(Object obj)`                                                   | 判断其他对象是否等于 Optional                                                                                                                  |
| `int hashCode()`                                                               | 如果值存在，则返回该值的哈希值；如果值不存在，则返回 0                                                                                         |
| `String toString()`                                                            | 返回此 Optional 的非空字符串表示形式，适用于调试                                                                                               |

> Java 8 除了提供了 `Optional` 类之外，还提供了 `OptionalInt`、`OptionalLong`、`OptionalDouble`，他们的使用方法和 `Optional` 类似。

## Optional 示例 ##
可以通过以下实例来更好的了解 `Optional` 类的使用：
```java
public class OptionalTest {

    public static void main(String[] args) {
        User user1 = null;
        User user2 = new User("Aaron", 16);
        User user3 = new User("Cherry", 18);

        Optional<User> optional1 = Optional.ofNullable(user1);
        Optional<User> optional2 = Optional.of(user2);
        Optional<User> optional3 = Optional.of(user3);

        System.out.println("Whether the value of optional1 exists : " + optional1.isPresent());
        System.out.println("Whether the value of optional2 exists : " + optional2.isPresent());

        optional3.ifPresent(user -> System.out.println("The name of one is "+ user3.name+ ", and the age is " + user3.age));

        System.out.println("The value of user2 : " + optional2.get());
        System.out.println("The value of user1 or user3 : " + optional1.orElse(user3));
        System.out.println("The value of user2 or user3 : " + optional2.orElse(user3));

        System.out.println("The name of user3 : " + optional3.map(user -> user.name).get());

        Optional<User> optional4 = optional3.flatMap(user -> Optional.of(new User(user.name, user.age)));
        System.out.println("The value of optional4 : " + optional4.get());

        System.out.println("Whether optional3 and optional4 are equal : " + optional3.equals(optional4));
    }

    private static class User {

        private String name;
        private int age;

        public User(String name, int age) {
            this.name = name;
            this.age = age;
        }

        @Override
        public String toString() {
            return "User{" + "name='" + name + '\'' + ", age=" + age + '}';
        }
    }

}
```
