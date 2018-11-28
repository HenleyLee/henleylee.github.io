---
title: Java 反射详解
date: 2018-11-26 18:30:28
categories: Java
tags:
  - Java
  - 反射
---

## 反射的概述 ##
### 什么是反射？ ###
Java 反射(`Reflection`)机制就是在运行状态中，对于`任意`一个类，都能够获取到这个类的所有属性和方法。对于`任意`一个对象，都能够调用它的`任意`一个方法和属性(包括私有的方法和属性)。这种动态获取的信息以及动态调用对象的方法的功能就称为 Java 语言的反射机制。通俗点讲，通过反射，该类对我们来说是完全透明的，想要获取任何东西都可以。Java 程序中一般的对象的类型都是在编译期就确定下来的，而 Java 反射机制可以动态地创建对象并调用其属性，这样的对象的类型在编译期是未知的。所以我们可以通过反射机制直接创建对象，即使这个对象的类型在编译期是未知的。

反射的核心是 JVM 在**`运行时`**才动态加载类或调用方法/访问属性，它不需要事先(写代码的时候或编译期)知道运行对象是谁。反射机制就是通过 `java.lang.Class` 类来实现的，在 Java 中，`Object` 类是所有类的根类，而 `Class` 类就是描述 Java 类的类。

> **注**：`Class` 本身就是一个类，`Class` 就是这个类的名称(注意首字母是大写)，所以 `Object` 也包括 `Class` 类。`public class Demo {}`，这里的 `class` 是作为关键字，来表明 `Demo` 是一个类。

### 反射的主要功能 ###
Java 反射框架主要提供以下功能：
 - 在运行时判断任意一个对象所属的类；
 - 在运行时构造任意一个类的对象；
 - 在运行时判断任意一个类所具有的成员变量和方法(通过反射甚至可以调用 private 方法)；
 - 在运行时调用任意一个对象的方法；
 - 修改构造函数、方法、属性的可见性。

### 反射的主要用途 ###
**反射最重要的用途就是开发各种通用框架**。很多框架(比如 Spring)都是配置化的(比如通过 XML 文件配置 JavaBean、Action 之类的)，为了保证框架的通用性，它们可能需要根据配置文件加载不同的对象或类，调用不同的方法，这个时候就必须用到反射——运行时动态加载需要加载的对象。对与框架开发人员来说，反射虽小但作用非常大，它是各种容器实现的核心。

## 反射的使用 ##
反射机制中会用到一些类，在了解反射是如何使用之前，先介绍一下这些类。

| 类 	      | 说明                                                                                          |
|-------------|-----------------------------------------------------------------------------------------------|
| Class       | 在反射中表示内存中的一个 Java 类，Class 可以代表的实例类型包括，类和接口、基本数据类型、数组  |
| Object      | Java 中所有类的超类                                                                           |
| Constructor | 封装了类的构造函数的属性信息，包括访问权限和动态调用信息                                      |
| Field       | 提供类或接口的成员变量属性信息，包括访问权限和动态修改                                        |
| Method      | 提供类或接口的方法属性信息，包括访问权限和动态调用信息                                        |
| Modifier    | 封装了修饰属性，public、protected、static、final、synchronized、abstract 等                   |

声明一个接口和一个简单的具体类作为示例代码：
声明一个接口：
```java
package com.custom;

public interface IAddress {
	void address();
}
```

具体的类，注意类各个方法的修饰属性。
```java
package com.custom;

public class Person implements IAddress {

	private int age;
	public String name;

	public Person() {
	}

	public Person(int age, String name) {
		this.age = age;
		this.name = name;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public Person setData(int age, String name) {
		this.age = age;
		this.name = name;
		return this;
	}

	@Override
	public void address() {
		System.out.println("I am from China.");
	}

	@Override
	public String toString() {
		return "Person [age=" + age + ", name=" + name + "]";
	}

}
```

### 获取 Class 对象 ###
想要使用反射机制，就必须要先获取到该类的字节码文件对象(.class)，通过字节码文件对象，就能够通过该类中的方法获取到我们想要的所有信息(方法，属性，类名，父类名，实现的所有接口等等)，每一个类对应着一个字节码文件也就对应着一个 `Class` 类型的对象，也就是字节码文件对象。

反射的各种功能都需要通过 `Class` 对象来实现，因此，需要知道如何获取 `Class`对象，主要有以下三种方式：
 - **`调用某个对象的 getClass() 方法：`**通过类的实例获取该类的字节码文件对象，该类处于`创建对象阶段`。
 - **`直接获取某个类的 class：`**当类被加载成 `.class` 文件时，此时该类变成了 `.class`，在获取该字节码文件对象，也就是获取自己，该类处于`字节码阶段`。
 - **`使用 Class.forName() 的静态方法：`**通过 `Class` 类中的静态方法 `forName()`，可以通过类或接口的名称(一个字符串或完全限定名)来获取对应的的字节码文件对象。此时该类还是`源文件阶段`，并没有变为字节码文件。

代码示例：
```java
package com.custom;

public class Test {

	public static void main(String[] args) throws ClassNotFoundException {
		// 调用某个对象的 getClass() 方法
		Person person = new Person();
		Class<? extends Person> clazz1 = person.getClass();

		// 直接获取某个类的 class
		Class<Person> clazz2 = Person.class;

		// 使用 Class.forName() 的静态方法
		Class<?> clazz3 = Class.forName("com.custom.Person");
	}

}
```

> 注意：在运行期间，一个类，只有一个 `Class` 对象产生。

### 判断是否为某个类的实例 ###

一般地，可以使用 `instanceof` 关键字来判断是否为某个类的实例。同时也可以借助反射中 `Class` 对象的 `isInstance()` 方法来判断是否为某个类的实例，它是一个 `Native` 方法：
```java
public native boolean isInstance(Object obj);
```

### 基类或者接口 ###
可以通过反射，来获取一个类的基类或者实现的接口，使用 `getSuperclass()` 获取该类的基类，使用 `getInterfaces()` 获取该类实现的接口。

代码示例：
```java
package com.custom;

public class Test {

	public static void main(String[] args) throws ClassNotFoundException {
		Class clazz = Class.forName("com.custom.Person");
		Class superClazz = clazz.getSuperclass();
		System.out.println("该类的父类：");
		System.out.println(superClazz.getName());
		
		System.out.println();
		System.out.println("该类实现的接口：");
		Class[] interfaces = clazz.getInterfaces();
		for (Class clz : interfaces) {
			System.out.println(clz.getName());
		}
	}

}
```

### 创建实例 ###
通过反射来生成对象主要有两种方式：
 - 使用 Class 对象的 newInstance() 方法
 - 通过 Class 对象获取指定的 Constructor 对象，再调用 Constructor 对象的 newInstance() 方法

代码示例：
```java
package com.custom;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

public class Test {

	public static void main(String[] args) throws InstantiationException, IllegalAccessException, NoSuchMethodException, SecurityException, IllegalArgumentException, InvocationTargetException {
		// 使用 Class 对象的 newInstance() 方法
		Class<?> clazz1 = Person.class;
		Object obj1 = clazz1.newInstance();
		((Person) obj1).setData(16, "Shelly");
		System.out.println(obj1);

		// 通过 Class 对象获取指定的 Constructor 对象，再调用 Constructor 对象的 newInstance() 方法
		Class<?> clazz2 = Person.class;
		Constructor<?> constructor = clazz2.getConstructor(int.class, String.class);
		Object obj2 = constructor.newInstance(18, "Shirley");
		System.out.println(obj2);
	}

}
```

> **注**：第二种方法可以用指定的构造器构造类的实例。

### 获取方法 ###
获取某个 Class 对象的方法集合，主要有以下几个方法：
 - **`Method[] getMethods()：`**返回一个包含此 Class 对象所表示的类或接口的所有公共方法所反映 Method 对象的数组，包括由类或接口声明的那些从超类和超接口继承的那些方法。
 - **`Method[] getDeclaredMethods()：`**返回一个包含此 Class 对象所表示的类或接口的所有已声明方法所反映 Method 对象的数组，包括 public、protected、default(包)访问和 private 方法，但不包括继承的方法。
 - **`Method getMethod(String name, Class<?>... parameterTypes)：`**返回一个此 Class 对象所表示的类或接口的指定公共成员方法所反映的 Method 对象。
 - **`Method getDeclaredMethod(String name, Class<?>... parameterTypes)：`**返回一个此 Class 对象表示的类或接口的指定声明方法所反映的 Method 对象。

代码示例：

```java
package com.custom;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.lang.reflect.Modifier;

public class Test {

	public static void main(String[] args) throws IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
        Class<?> clazz = Person.class;

        System.out.println("通过getMethods()方法获取该类的方法：");
        Method[] methods = clazz.getMethods();
        for (Method method : methods) {
            System.out.println(Modifier.toString(method.getModifiers()) + " " + method.getReturnType() + " " + method.getName());
        }

        System.out.println();
        System.out.println("通过getDeclaredMethods()方法获取该类的方法：");
        Method[] declaredMethods = clazz.getDeclaredMethods();
        for (Method declaredMethod : declaredMethods) {
            System.out.println(Modifier.toString(declaredMethod.getModifiers()) + " " + declaredMethod.getReturnType() + " " + declaredMethod.getName());
        }

        System.out.println();
        System.out.println("通过getMethod()方法获取该类的方法：");
        Method method = clazz.getMethod("toString");
        System.out.println(Modifier.toString(method.getModifiers()) + " " + method.getReturnType() + " " + method.getName());

        System.out.println();
        System.out.println("通过getDeclaredMethod()方法获取该类的方法：");
        Method declaredMethod = clazz.getDeclaredMethod("address");
        System.out.println(Modifier.toString(declaredMethod.getModifiers()) + " " + declaredMethod.getReturnType() + " " + declaredMethod.getName());
    }

}
```

> **注**：通过`getMethods()`获取的方法可以获取到父类的方法,比如`java.lang.Object`下定义的各个方法。

### 获取构造方法 ###
获取某个 Class 对象的构造方法，主要有以下几个方法：
 - **`Constructor<?>[] getConstructors()：`**返回一个包含此 Class 对象所表示的类的所有公共构造函数的所反映的 Constructor 对象的数组。
 - **`Constructor<?>[] getDeclaredConstructors()：`**返回一个包含此 Class 对象表示的类声明的所有构造函数所反映的 Constructor 对象的数组。
 - **`Constructor<T> getConstructor(Class<?>... parameterTypes)：`**返回一个此 Class 对象所表示的类的指定公共构造函数所反映的 Constructor 对象。
 - **`Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes)：`**返回一个此 Class 对象表示的类或接口的指定构造函数所反映的 Constructor 对象。

通过 `Constructor` 对象创建一个对象实例，主要是通过 `Class` 类的 `getConstructor` 方法得到 `Constructor` 类的一个实例，而 `Constructor` 类有一个 `newInstance` 方法可以创建一个对象实例：
```java
public T newInstance(Object ... initargs)
```

> **注**：此方法可以根据传入的参数来调用对应的 `Constructor` 创建对象实例。

### 获取成员变量 ###
获取某个 Class 对象的成员变量，主要有以下几个方法：
 - **`Field[] getFields()：`**返回一个包含此 Class 对象所表示的类或接口的所有可访问公共字段所反映的 Field 对象的数组。
 - **`Field[] getDeclaredFields()：`**返回一个包含此 Class 对象所表示的类或接口声明的所有字段所反映的 Field 对象的数组。
 - **`Field getField(String name)：`**返回一个此 Class 对象表示的类或接口的指定公共成员字段所反映的 Field 对象。
 - **`Field getDeclaredField(String name)：`**返回一个此 Class 对象表示的类或接口的指定声明字段所反映的 Field 对象。

### 调用方法 ###
当我们从类中获取了一个方法后，我们就可以用 `invoke()` 方法来调用这个方法。`invoke()` 方法的原型为:
```java
public Object invoke(Object obj, Object... args) throws IllegalAccessException, IllegalArgumentException, InvocationTargetException
```

代码示例：
```java
package com.custom;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Test {

	public static void main(String[] args) throws IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
		Class<?> clazz = Person.class;
		// 创建 Person 的实例
		Object obj = clazz.newInstance();
		// 获取 Person 类的 setData 方法
		Method method = clazz.getMethod("setData", int.class, String.class);
		// 调用 method 对应的方法 => setData(16, "Shelly")
		Object result = method.invoke(obj, 16, "Shelly");
		System.out.println(result);
	}

}
```

### 利用反射创建数组 ###
数组在 Java 里是比较特殊的一种类型，它可以赋值给一个 `Object Reference`。下面我们看一看利用反射创建数组的例子：
```java
package com.custom;

import java.lang.reflect.Array;

public class Test {

	public static void main(String[] args) throws ClassNotFoundException {
		// 使用`java.lang.reflect.Array`反射创建长度为25的字符串数组.
		Class<?> clazz = Class.forName("java.lang.String");
		Object array = Array.newInstance(clazz, 10);
		// 往数组里添加内容
		Array.set(array, 0, "Hello");
		Array.set(array, 1, "Java");
		Array.set(array, 2, "Kotlin");
		Array.set(array, 3, "Android");
		// 获取某一项的内容
		System.out.println(Array.get(array, 3));
	}

}
```

其中的 `Array` 类为 `java.lang.reflect.Array` 类。我们通过 `Array.newInstance()` 创建数组对象，而 `newArray` 方法是一个 `native` 方法，它的原型是：
```java
public static Object newInstance(Class<?> componentType, int length) throws NegativeArraySizeException {
    return newArray(componentType, length);
}

private static native Object newArray(Class<?> componentType, int length) throws NegativeArraySizeException;
```

## 使用反射获取信息 ##
Class 类提供了大量的实例方法来获取该 Class 对象所对应的详细信息，Class 类大致包含如下方法，其中每个方法都包含多个重载版本，因此我们只是做简单的介绍，详细请参考[JDK 文档](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html)。

### 获取类内信息 ###
 - 构造器: `Constructor<T> getConstructor(Class<?>... parameterTypes)`
 - 包含的方法: `Method getMethod(String name, Class<?>... parameterTypes)`
 - 包含的属性: `Field getField(String name)`
 - 包含的Annotation: `<A extends Annotation> A getAnnotation(Class<A> annotationClass)`
 - 内部类: `Class<?>[] getDeclaredClasses()`
 - 外部类: `Class<?> getDeclaringClass()`
 - 所实现的接口: `Class<?>[] getInterfaces()`
 - 修饰符: `int getModifiers()`
 - 所在包: `Package getPackage()`
 - 类名: `String getName()`
 - 简称: `String getSimpleName()`

### 判断类本身信息的方法 ###
 - 是否注解类型: `boolean isAnnotation()`
 - 是否使用了该Annotation修饰: `boolean isAnnotationPresent(Class<? extends Annotation> annotationClass)`
 - 是否匿名类: `boolean isAnonymousClass()`
 - 是否数组: `boolean isArray()`
 - 是否枚举: `boolean isEnum()`
 - 是否原始类型: `boolean isPrimitive()`
 - 是否接口: `boolean isInterface()`
 - obj 是否是该 Class 的实例: `boolean isInstance(Object obj)`

### 使用反射获取泛型信息 ###
为了通过反射操作泛型以迎合实际开发的需要，Java 新增了 `java.lang.reflect.ParameterizedType`、`java.lang.reflect.GenericArrayType`、`java.lang.reflect.TypeVariable`、`java.lang.reflect.WildcardType` 几种类型来代表不能归一到 Class 类型但是又和原始类型同样重要的类型。
 - `ParameterizedType`: 一种参数化类型，比如Collection<String>
 - `GenericArrayType`: 一种元素类型是参数化类型或者类型变量的数组类型
 - `TypeVariable`: 各种类型变量的公共接口
 - `WildcardType`: 一种通配符类型表达式，如`?`、`? extends Number`、`? super Integer`

代码示例：

```java
public class Client {

	private Map<String, Object> objectMap;

	public void test(Map<String, User> map, String string) {
	}

	public Map<User, Bean> test() {
		return null;
	}

	/**
	 * 测试属性类型
	 *
	 * @throws NoSuchFieldException
	 */
	@Test
	public void testFieldType() throws NoSuchFieldException {
		Field field = Client.class.getDeclaredField("objectMap");
		Type gType = field.getGenericType();
		// 打印type与generic type的区别
		System.out.println(field.getType());
		System.out.println(gType);
		System.out.println("**************");
		if (gType instanceof ParameterizedType) {
			ParameterizedType pType = (ParameterizedType) gType;
			Type[] types = pType.getActualTypeArguments();
			for (Type type : types) {
				System.out.println(type.toString());
			}
		}
	}

	/**
	 * 测试参数类型
	 *
	 * @throws NoSuchMethodException
	 */
	@Test
	public void testParamType() throws NoSuchMethodException {
		Method testMethod = Client.class.getMethod("test", Map.class, String.class);
		Type[] parameterTypes = testMethod.getGenericParameterTypes();
		for (Type type : parameterTypes) {
			System.out.println("type -> " + type);
			if (type instanceof ParameterizedType) {
				Type[] actualTypes = ((ParameterizedType) type).getActualTypeArguments();
				for (Type actualType : actualTypes) {
					System.out.println("\tactual type -> " + actualType);
				}
			}
		}
	}

	/**
	 * 测试返回值类型
	 *
	 * @throws NoSuchMethodException
	 */
	@Test
	public void testReturnType() throws NoSuchMethodException {
		Method testMethod = Client.class.getMethod("test");
		Type returnType = testMethod.getGenericReturnType();
		System.out.println("return type -> " + returnType);

		if (returnType instanceof ParameterizedType) {
			Type[] actualTypes = ((ParameterizedType) returnType).getActualTypeArguments();
			for (Type actualType : actualTypes) {
				System.out.println("\tactual type -> " + actualType);
			}
		}
	}
}
```

## 反射的一些注意事项 ##
由于反射会额外消耗一定的系统资源，因此如果不需要动态地创建一个对象，那么就不需要用反射。

另外，反射调用方法时可以忽略权限检查，因此可能会破坏封装性而导致安全问题。

