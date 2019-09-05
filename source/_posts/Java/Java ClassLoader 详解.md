---
title: Java ClassLoader 详解
categories: Java
tags:
  - Java
abbrlink: b7c8c167
date: 2019-05-02 19:16:25
---

Java 类加载器(Java ClassLoader)是 Java 运行时环境(Java Runtime Environment)的一部分，负责动态加载 Java 类到 Java 虚拟机的内存空间中。类通常是按需加载，即第一次使用该类时才加载。由于有了类加载器，Java运行时系统不需要知道文件与文件系统。

JVM 启动的时候，并不是一次性加载所有的类，而是根据需要动态去加载类，主要分为隐式加载和显式加载。
 - 隐式加载：程序代码中不通过调用 ClassLoader 来加载需要的类，而是通过 JVM 类自动加载需要的类到内存中。例如，当在类中继承或者引用某个类的时候，JVM 在解析当前这个类的时，发现引用的类不在内存中，那么就会自动将这些类加载到内存中。
 - 显式加载：代码中通过 Class.forName()、this.getClass.getClassLoader.LoadClass()、自定义类加载器中的 findClass() 方法等。

## 什么是类加载器 ##
Java 中的类加载器是加载 Java 类文件(*.class)的一个类。Java 源代码被 javac 编译器编译后以字节码的形式保存到类文件，JVM(Java 虚拟机)通过执行类文件里的字节码来执行 Java 程序。类加载器负责从文件系统、网络或其他资源中加载类文件。

Java 中使用的默认类加载器有以下三种：Bootstrap 类加载器、Extension 类加载器和 System/Application 类加载器。每个类加载器都有一个预定义的位置，它们在预定义的位置加载类文件。
 - **`BootStrap ClassLoader(引导类加载器)：`**主要加载 %JDK_HOME%\jre\lib 下的 rt.jar、resources.jar、charsets.jar 和 class 等 JDK 类文件，可以通 System.getProperty("sun.boot.class.path") 查看加载的路径。Bootstrap 类加载器是所有类加载器的父加载器，它没有任何父加载器。
 - **`Extension ClassLoader(扩展类加载器)：`**主要加载目录 %JDK_HOME%\jre\lib\ext 目录下的 jar 和 class 文件，可以通过 System.getProperty("java.ext.dirs") 查看加载类文件的路径。Extension 类加载器将加载类的请求先委托给它的父加载器，也就是 Bootstrap，如果没有成功加载的话，再从 jre/lib/ext 目录下或者 java.ext.dirs 系统属性定义的目录下加载类。Extension 加载器由 sun.misc.Launcher$ExtClassLoader 实现。
 - **`System ClassLoader(系统类加载器)：`**又叫作 Application 类加载器，它负责从 classpath 环境变量中加载某些应用相关的类，classpath 环境变量通常由 -classpath 或 -cp 命令行选项来定义，或者是 JAR 中的 Manifest 的 classpath 属性。Application 类加载器是 Extension 类加载器的子加载器，通过 sun.misc.Launcher$AppClassLoader 实现。

## 类加载器的关系 ##
Bootstrap 类加载器、Extension 类加载器和 System/Application 类加载器之间的关系如下图所示：
![Java类加载器的关系](https://henleylee.github.io/medias/java/class_loader_relation.png)

ExtClassLoader 和 AppClassLoder 继承 URLClassLoader，而 URLClassLoader 继承 ClassLoader，BoopStrap ClassLoder 是用 C/C++ 代码来实现的，并不继承自 java.lang.ClassLoader，它本身是虚拟机的一部分，并不是一个 Java 类。

JVM 加载的顺序：BoopStrap ClassLoder -> ExtClassLoader -> AppClassLoder，下面看一段源码：
```java
public class Launcher {

    private static Launcher launcher = new Launcher();
    private static String bootClassPath = System.getProperty("sun.boot.class.path");
    private ClassLoader loader;

    public static Launcher getLauncher() {
        return launcher;
    }

    public Launcher() {
        // Create the extension class loader
        ClassLoader extcl;
        try {
            extcl = ExtClassLoader.getExtClassLoader();
        } catch (IOException e) {
            throw new InternalError("Could not create extension class loader", e);
        }

        // Now create the class loader to use to launch the application
        try {
            loader = AppClassLoader.getAppClassLoader(extcl);
        } catch (IOException e) {
            throw new InternalError("Could not create application class loader", e);
        }

        Thread.currentThread().setContextClassLoader(loader);
    }

    /*
     * Returns the class loader used to launch the main application.
     */
    public ClassLoader getClassLoader() {
        return loader;
    }

    /*
     * The class loader used for loading installed extensions.
     */
    static class ExtClassLoader extends URLClassLoader {}

    /**
     * The class loader used for loading from java.class.path.
     * runs in a restricted security context.
     */
    static class AppClassLoader extends URLClassLoader {}

```

从上面的源码中我们看到：Launcher 初始化的时候创建了 ExtClassLoader 以及 AppClassLoader，并将 ExtClassLoader 实例传入到 AppClassLoader 中。虽然上面的源码中没见到创建 BoopStrap ClassLoader，但是程序一开始就执行了 System.getProperty("sun.boot.class.path")。

> AppClassLoader 的父加载器为 ExtClassLoader，ExtClassLoader 的父加载器为 null，BoopStrap ClassLoader 为顶级加载器。

## 类加载器的工作原理 ##
Java 类加载器的工作原理基于三个机制：委托、可见性和单一性。
 - **`委托机制：`**是指将加载一个类的请求交给父类加载器，如果这个父类加载器不能够找到或者加载这个类，那么再加载它。
 - **`可见性原理：`**是指子类加载器可以看见所有的父类加载器加载的类，而父类加载器看不到子类加载器加载的类。
 - **`单一性原理：`**是指仅加载一个类一次，父加载器加载过的类不能被子加载器加载第二次，这是由委托机制确保子类加载器不会再次加载父类加载器加载过的类。

正确理解类加载器能够帮你解决 java.lang.NoClassDefFoundError 和 java.lang.ClassNotFoundException，因为它们和类的加载相关。

## 类加载器的双亲委派机制 ##
ClassLoader 使用双亲委派机制来加载 class 文件，双亲委派机制的流程是这样的：当 JVM 要加载一个类的时候，
1. 首先会到自定义加载器中查找，看是否已经加载过，如果已经加载过，则返回字节码；
2. 如果自定义加载器没有加载过，则询问上一层加载器(AppClassLoader)是否已经加载过；
3. 如果没有加载过，则询问上一层加载器(ExtClassLoader)是否已经加载过；
4. 如果没有加载过，则继续询问上一层加载(BoopStrap ClassLoader)是否已经加载过；
5. 如果 BoopStrap ClassLoader 依然没有加载过，则到自己指定类加载路径下("sun.boot.class.path")查看是否有该类的字节码，有则返回，没有通知下一层加载器 ExtClassLoader 到自己指定的类加载路径下("java.ext.dirs")查看；
6. 依次类推，最后到自定义类加载器指定的路径还没有找到Test.class字节码，则抛出异常ClassNotFoundException。

JVM 要加载一个类的流程如下图所示：
![类加载器的双亲委派机制](https://henleylee.github.io/medias/java/class_loader_process.png)


> 某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父类加载器，依次递归，如果父类加载器可以完成类加载任务，就成功返回；只有父类加载器无法完成此加载任务时，才自己去加载。

## 类的加载过程 ##
类的加载过程会使用到 findLoadedClass()、loadClass()、findClass() 等方法，ClassLoader 的 loadClass() 方法源码如下：
```java
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        // 首先，检查类是否已经加载
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            try {
                if (parent != null) {
                    // 父加载器不为空，调用父加载器的loadClass()方法
                    c = parent.loadClass(name, false);
                } else {
                    // 父加载器为空则，调用 Bootstrap ClassLoader
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }

            if (c == null) {
                // 如果仍然没有找到该类，则调用findClass()方法以找到该类
                c = findClass(name);
            }
        }
        return c;
}
```

## 自定义类加载器 ##
要实现自定义类加载器需要先继承 ClassLoader，ClassLoader 类是一个抽象类，负责加载 class 的对象。自定义 ClassLoader 中至少需要了解其中的三个的方法：loadClass()、findClass()、defineClass()。
```java
public Class<?> loadClass(String name) throws ClassNotFoundException {
    return loadClass(name, false);
}

protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
            
}

protected Class<?> findClass(String name) throws ClassNotFoundException {
    throw new ClassNotFoundException(name);
}

protected final Class<?> defineClass(String name, byte[] b, int off, int len) throws ClassFormatError {
    throw new UnsupportedOperationException("can't load this type of class file");
}
```

 - **`loadClass()：`**JVM 在加载类的时候，都是通过 ClassLoader 的 loadClass() 方法来加载 class 的，loadClass() 方法使用双亲委派模式。如果要改变双亲委派模式，可以修改 loadClass() 方法来改变 class 的加载方式。
 - **`findClass()：`**ClassLoader 通过 findClass() 方法来加载类。自定义类加载器实现这个方法来加载需要的类，比如指定路径下的文件、字节流等。
 - **`definedClass()：`**将定义的字节码文件经过字节数组流解密之后，将该字节流数组生成字节码文件，也就是该类文件的类名 .class。通常用在重写 findClass() 方法中，返回一个 Class 对象。

### 实现步骤 ###
1. 自定义的ClassLoader通过继承ClassLoader来实现，也可以使用URLClassLoader更简单。
2. 如果需要改写类的加载过程最好覆盖 findClass() 而不是 loadClass()，loadClass() 是为了保持 JDK 1.2 之前的兼容，使用 findClass() 能保证不会违背双亲委派模式。

### 实现原理 ###
 - 创建自定义类加载器对象时，默认(或显式指定)其父加载器；
 - 显式调用继承来的 loadClass() 方法来加载类，此时在 loadClass() 方法内部将执行双亲委派的逻辑，如果父类加载没能成功加载类，则调用重写的 findClass() 方法来实现自定义类加载器加载类的逻辑：
  - 将要加载的类的 .class 文件中的内容输入到 byte[] 数组中；
  - 调用父加载器的 defineClass() 方法完成真正的类加载。

### 代码示例 ###
```java
public class CustomClassLoader extends ClassLoader {

    private String dirPath;

    public CustomClassLoader(String dirPath) {
        this.dirPath = dirPath;
    }

    public CustomClassLoader(ClassLoader parent, String dirPath) {
        super(parent);
        this.dirPath = dirPath;
    }

    @Override
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        // 查找这个类是否加载了
        Class<?> cls = findLoadedClass(name);
        if (cls == null) {
            // 获取到父加载器
            ClassLoader parent = this.getParent();
            try {
                // 委派给父加载器加载
                cls = parent.loadClass(name);
            } catch (ClassNotFoundException e) {
                // ignore
            }

            if (cls == null) {
                cls = findClass(name);
            }
        }
        return cls;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        Class cls = null;
        try {
            String classPath = dirPath + "/" + name.replace('.', '/') + ".class";
            byte[] data = getClassFileBytes(classPath);
            if (data == null) {
                throw new ClassNotFoundException();
            }
            cls = defineClass(name, data, 0, data.length);
            if (cls == null) {
                throw new ClassFormatError();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return cls;
    }

    private byte[] getClassFileBytes(String classFile) throws IOException {
        FileInputStream fis = new FileInputStream(classFile);
        FileChannel fileChannel = fis.getChannel();
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        WritableByteChannel outC = Channels.newChannel(baos);
        ByteBuffer buffer = ByteBuffer.allocateDirect(1024);
        while (true) {
            int i = fileChannel.read(buffer);
            if (i == 0 || i == -1) {
                break;
            }
            buffer.flip();
            outC.write(buffer);
            buffer.clear();
        }
        fis.close();
        return baos.toByteArray();
    }

}
```

