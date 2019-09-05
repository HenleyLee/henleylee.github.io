---
title: 深入理解 Gradle 框架之一：Plugin、Extension、buildSrc
categories: Gradle
tags:
  - Gradle
  - Plugin
abbrlink: babf38d8
date: 2019-06-10 18:30:55
---

本文转载自：[https://mp.weixin.qq.com/s/mDCTtQZb6mhWOFAvLYKBSg](https://mp.weixin.qq.com/s/mDCTtQZb6mhWOFAvLYKBSg)

## 缘起 ##
从2018年下半年开始，因为工作需要，开始深入了解 `Android Gradle plugin` 和 `Gradle` 框架，在看完 `Android Gradle plugin` 3.1.x 和 3.2.x 版本的源码之后，发现目前开源的几乎所有插件化框架，因为没有理解 `Android Gradle plugin` 的原理，打包代码的实现都非常混乱，导致的结果就是很难随着 `Android Gradle plugin` 的升级而快速升级，所以目前几乎所有开源的插件化项目都因不适应新的 Gradle 版本问题而不可用。

另一方面，随着项目的迭代，引入越来越多的 `Gradle plugin`，其中对于 `Transform` 的滥用是导致项目编译速度越来越慢的最重要的一个原因了，然而，实际上，对于 `Transform` 的使用是有很大的优化空间的。

加上目前不管是中文还是英文，几乎所有这方面的文章都停留在基础使用的阶段，真正深入分析原理的几乎没有。

所以一直在酝酿写一个 Gradle 系列的文章，一方面让大家了解 `Android Gradle plugin` 的原理（尽管各个大版本之间有差别，然而大版本内基本是一脉相承），另一方面是在介绍原理的过程中，也会加入一些我觉得是 Best Practice 的 Demo。

或许通过这个系列，能够引导大家都去改进自己实现的 Plugin，最终能够更快更好地实现自己的编译时功能。

p.s：因为基础使用的文章已经有太多了，对于这一块我基本上就是一笔带过，不会花太多笔墨。


## 系列说明 ##
在讲解整个系列之前，先看一下 Gradle 的架构是怎样的，如下图所示：
![Gradle架构](https://henleylee.github.io/medias/gradle/gradle_architecture.webp)
 - 在最下层的是底层 `Gradle 框架`，它主要提供一些基础服务，如 task 的依赖，有向无环图的构建等
 - 上面的则是 Google 编译工具团队的 `Android Gradle plugin 框架`，它主要是在 Gradle 框架的基础上，创建了很多与 Android 项目打包有关的 task 及 artifacts
 - 最上面的则是开发者`自定义的 Plugin`，一般是在 Android Gradle plugin 提供的 task 的基础上，插入一些自定义的 task，或者是增加 Transform 进行编译时代码注入

为了简单起见，我将底层的 Gradle 框架和 Android Gradle plugin 框架统称为 Gradle 框架，整个系列文章其实分析的就是底层 Gradle 框架和 Android Gradle plugin 框架的原理，其中侧重点在 Android Gradle plugin 框架，因为这与我们日常编译息息相关，也是收益最大的部分。

这是深入理解 Gradle 框架系列的第一篇。整个系列共分为9篇，文章列表如下：
 - 第1篇：入门文章，主要讲解 Gradle Plugin 以及 Extension 的多种用法，以及 buildSrc 及 gradle 插件调试的方法。
 - 第2篇：从 dependencies 出发，阐述 DependencyHandler 的原理
 - 第3篇：关于 gradle configuration 的预备知识介绍，以及 artifacts 的发布流程
 - 第4篇：artifacts 的获取流程
 - 第5篇：从 TaskManager 出发，分析如 ApplicationTaskManager，LibraryTaskManager 中各主要的 Task，最后给出当前版本的编译流程图
 - 第6篇：比较 3.2.1 相比 3.1.2 中架构的变化
 - 第7篇：关于 Gradle Transform
 - 第8篇：从打包的角度讲解 app bundles 的原理
 - 第9篇：分析资源编译的流程，特别是 aapt2 的编译流程

我们会陆续更新，敬请期待。

## Plugin ##
### 语言选择 ###
其实只要是 `JVM` 语言，都可以用来写插件，比如 Android Gradle plugin 团队，在 3.2.0 之前一直是用 `Java` 编写 `Gradle` 插件。

国内很多开源项目都是用 `Groovy` 编写的，`Groovy` 的优势是书写方便，而且其闭包写法非常灵活，然而 `Groovy` 的缺点也非常明显，最大的一点不好就是 `IDE` 对其的支持非常不好，不仅仅是语法高亮没做好，还有导航跳转都极为有限，比如 `build.gradle` 中的方法跳转不到其定义处。

当然，我自己长期使用 `Groovy` 下来，也发现了它的一些缺点，比如 each 这个闭包，在运行时竟然会出现找不到其成员的情况。

以及出现开发者自定义的成员与其默认成员（`Groovy` 中会为每个类增加一些默认成员）名称重合时，不能给出有效的提示，当然，这个问题我不确定是 `IDE` 的问题还是 `Groovy` 自身的编译器实现不够完善的问题。

其实到目前为止，使用 `Kotlin` 进行插件开发是最好的选择，有如下两个原因:
 - `Kotlin` 的语法糖完全不输于 Groovy，可以有效提高开发效率
 - `Kotlin` 是 JetBrains 自家的，IDE 对其的支持更完善

可能正是这个原因，`Google` 编译工具组从 `3.2.0` 开始，新增的插件全部都是用 `Kotlin` 编写的。

### 插件名与 Plugin 的关系 ###
比如我们常用的 `apply plugin: 'com.android.library'`，其实是对应的 `AppPlugin`， 其声明在源码的 `META-INF` 中，如下图所示：
![插件名与Plugin的关系](https://henleylee.github.io/medias/gradle/gradle_plugin_relation.webp)

可以看到，不仅仅有 `com.android.appliation`，还有我们经常用到的 `com.android.library`，以及 `com.android.feature`、`com.android.dynamic-feature`。

以 `com.android.application.properties` 为例，其内容如下：
```gradle
implementation-class=com.android.build.gradle.AppPlugin
```
其含义很清楚了，就表示 `com.android.application` 对应的插件实现类是 `com.android.build.gradle.AppPlugin` 这个类。

其他的类似，就不一一列举了。

### 定义插件的方法 ###
要定义一个 Gradle Plugin，则要实现 `org.gradle.api.Plugin` 接口，该接口如下：
```java
/**
 * <p>A <code>Plugin</code> represents an extension to Gradle. A plugin applies some configuration to a target object.
 * Usually, this target object is a {@link org.gradle.api.Project}, but plugins can be applied to any type of
 * objects.</p>
 *
 * @param <T> The type of object which this plugin can configure.
 */
public interface Plugin<T> {
    /**
     * Apply this plugin to the given target object.
     *
     * @param target The target object
     */
    void apply(T target);
}
```

以我们经常用的 `AppPlugin` 和 `LibraryPlugin`，其继承关系如下：
![AppPlugin和LibraryPlugin的继承关系](https://henleylee.github.io/medias/gradle/gradle_plugin_extends.webp)
> 注意，这是 3.2.0 之前的继承关系，在 3.2.0 之后，略微有些调整。

可以看到，`LibraryPlugin` 和 `AppPlugin` 都继承自 `BasePlugin`， 而 `BasePlugin` 实现了 `Plugin` 接口，如下：
```java
/** Base class for all Android plugins */
public abstract class BasePlugin<E extends BaseExtension2>
        implements Plugin<Project>, ToolingRegistryProvider {

    @VisibleForTesting
    public static final GradleVersion GRADLE_MIN_VERSION =
            GradleVersion.parse(SdkConstants.GRADLE_MINIMUM_VERSION);

    private BaseExtension extension;

    private VariantManager variantManager;
    
    ...

}
```
这里继承的层级多一层的原因是，有很多共同的逻辑可以抽出来放到 `BasePlugin` 中，然而大多数时候，我们可能没有这么复杂的关系，所以直接实现 `Plugin` 这个接口即可。

## Extension ##
`Extension` 其实可以理解成 Java 中的 Java bean，它的作用也是类似的，即获取输入数据，然后在插件中使用。

最简单的 `Extension` 为例，比如我定义一个名为 `Student` 的 `Extension`，其定义如下：
```groovy
class Student {
    String name
    int age
    boolean isMale
}
```

然后在 Plugin 的 `apply()` 方法中，添加这个 `Extension`，不然编译时会出现找不到的情形：
```groovy
project.extensions.create("student", Student.class)
```

这样，我们就可以在 `build.gradle` 中使用名为 `student` 的 `Extension`了，如下：
```gradle
student {
    name 'Mike'
    age 18
    isMale true
}
```
注意，这个名称要与创建 `Extension` 时的名称一致。

而获取它的方式也很简单：
```groovy
Student studen = project.extensions.getByType(Student.class)
```

嵌套的 `Extension` 类似，不再赘述。

如果 `Extension` 中要包含固定数量的配置项，那很简单，类似下面这样就可以：
```groovy
class Fruit {
    int count
    Fruit(Project project) {
        project.extensions.create("apple", Apple, "apple")
        project.extension.create("banana", Banana, "banana")
    }
}
```

其配置如下：
```gradle
fruit {
    count 3
    apple {
        name 'Big Apple'
        weight 580f
    }

    banana {
        name 'Yellow Banana'
        size 19f
    }
}
```

下面要说的是包含不定数量的配置项的 `Extension`，就需要用到 `NamedDomainObjectContainer`，比如我们常用的编译配置中的 `productFlavors`，就是一个典型的包含不定数量的配置项的 `Extension`。

但是，如果我们不进行特殊处理，而是直接使用 `NamedDomainObjectContainer` 的话，就会发现这个配置项都要用 `=` 赋值，类似下面这样。

接着使用 `Student`，如果我需要在某个配置项中添加不定项个 `Student` 输入，其添加方式如下：
```groovy
NamedDomainObjectContainer<Student> studentContainer = project.container(Student)
project.extensions.add('team', studentContainer)
```

然而，此时其配置只能如下：
```gradle
team {
    John {
        age = 18
        isMale = true
    }
    Daisy {
        age = 17
        isMale = false
    }
}
```
注意，这里不需要 `name` 了，因为 John 和 Daisy 就是 `name` 了。

可是，这不科学呀，`Groovy` 的语法不是可以省略么？就比如 `productFlavors` 这样：
![productFlavors语法](https://henleylee.github.io/medias/gradle/gradle_product_flavors.webp)

要达到这样的效果其实并不难，只要做好以下两点：
 - `item Extension` 的定义中必须有 `name` 这个属性，因为在 `Factory` 中会在创建时为这个名称的属性赋值。定义如下：
```groovy
class Cat {
    String name

    String from
    float weight
}
```

 - 需要定义一个实现了 `NamedDomainObjectFactory` 接口的类，这个类的构造方法中必须有 `instantiator` 这个参数，如下：
```groovy
class CatExtFactory implements NamedDomainObjectFactory<Cat>{
    private Instantiator instantiator
    
    CatExtFactory(Instantiator instantiator){
        this.instantiator=instantiator
    }
    
    @Override
    Cat create(String name){
        return instantiator.newInstance(Cat.class, name)
    }
}
```

此时，gradle配置文件中就可以类似这样写了：
```gradle
animal {
    count 58

    dog {
        from 'America'
        isMale false
    }

    catConfig {
        chinaCat {
            from 'China'
            weight 2900.8f
        }

        birman {
            from 'Burma'
            weight 5600.51f
        }

        shangHaiCat {
            from 'Shanghai'
            weight 3900.56f
        }

        beijingCat {
            from 'Beijing'
            weight 4500.09f
        }
    }
}
```

## Plugin Transform ##
`Transform` 是 Android Gradle plugin 团队提供给开发者使用的一个抽象类，它的作用是提供接口让开发者可以在源文件编译成为 `class` 文件之后，`dex` 之前进行字节码层面的修改。

借助 `javaassist`，`ASM` 这样的字节码处理工具，可在自定义的 `Transform` 中进行代码的插入，修改，替换，甚至是新建类与方法。

像美团点评的 `Robust`，以及我开源的 `Andromeda` 项目中，都有在 `Transform` 中插入代码的示例。

如下是一个自定义 `Transform` 实现：
```java
public class AllenCompTransform extends Transform {

    private Project project;
    private IComponentProvider provider

    public AllenCompTransform(Project project,IComponentProvider componentProvider) {
        this.project = project;
        this.provider=componentProvider
    }

    @Override
    public String getName() {
        return "AllenCompTransform";
    }

    @Override
    public Set<QualifiedContent.ContentType> getInputTypes() {
        return TransformManager.CONTENT_CLASS;
    }

    @Override
    public Set<? super QualifiedContent.Scope> getScopes() {
        return TransformManager.SCOPE_FULL_PROJECT;
    }

    @Override
    public boolean isIncremental() {
        return false;
    }

    @Override
    public void transform(TransformInvocation transformInvocation) throws TransformException, InterruptedException, IOException {

        long startTime = System.currentTimeMillis();

        transformInvocation.getOutputProvider().deleteAll();
        File jarFile = transformInvocation.getOutputProvider().getContentLocation("main", getOutputTypes(), getScopes(), Format.JAR);
        if (!jarFile.getParentFile().exists()) {
            jarFile.getParentFile().mkdirs()
        }
        if (jarFile.exists()) {
            jarFile.delete();
        }

        ClassPool classPool = new ClassPool()
        project.android.bootClasspath.each{
            classPool.appendClassPath((String)it.absolutePath)
        }

        def box=ConvertUtils.toCtClasses(transformInvocation.getInputs(),classPool)

        CodeWeaver codeWeaver=new AsmWeaver(provider.getAllActivities(),provider.getAllServices(),provider.getAllReceivers())
        codeWeaver.insertCode(box,jarFile)

        System.out.println("AllenCompTransform cost "+(System.currentTimeMillis()-startTime)+" ms")
    }
}
```

## Gradle 插件的发布 ##
绝大多数 Gradle 插件，我们可能都是只要在公司内部使用，那么只要使用公司内部的 maven 仓库即可，即配置并运用 maven 插件，然后执行其 `upload` task 即可。这个很简单，不再赘述。

## 特殊的 buildSrc ##
前面说过 Gradle 插件的发布，那如果我们在插件的代码编写阶段，总不能修改一点点代码，就发布一个版本，然后重新运用吧？

有人可能会说，那就不发布到 maven 仓库，而是发布到本地仓库呗，然而这样至多发布时节省一点点时间，仍然太麻烦。

幸好有 `buildSrc`！

在 `buildSrc` 中定义的插件，可以直接在其他 `module` 中运用，而且是类似这种运用方式：
```gradle
apply plugin: wang.imallen.blog.comp.MainPlugin
```

即直接 `apply` 具体的类，而不是其发布名称，这样的话，不管做什么修改，都能马上体现，而不需要等到重新发布版本。

## Gradle插件的调试 ##
以调试 `:app:assembleRelease` 这个 task 为例，其实很简单，分如下两步即可：
 - 新建 `remote target`
 - 在命令行输入 `./gradlew --no-daemon -Dorg.gradle.debug=true :app:assembleRelease`
 - 之后选择刚刚创建的 `remote target`，然后点击调试按钮即可

## 致谢 ##
[字节跳动技术团队](https://mp.weixin.qq.com/s/mDCTtQZb6mhWOFAvLYKBSg)

