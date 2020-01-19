---
title: 深入理解 Gradle 框架之二：依赖实现分析
categories: Gradle
tags:
  - Gradle
  - Plugin
abbrlink: 5fe63be9
date: 2019-06-26 18:36:29
---

本文转载自：[https://mp.weixin.qq.com/s/WQCqUaYDPHDIjUHlxjRIkw](https://mp.weixin.qq.com/s/WQCqUaYDPHDIjUHlxjRIkw)


## 引言 ##
大家在日常开发中，见过最多的可能就是下面3种依赖声明：
 - implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
 - implementation project(':applemodule')
 - implementation fileTree(dir:'libs', include:['*.jar'])

业务复杂一点的，可能会涉及类似下面这种：
 - implementation project(path: ':applemdoule')
 - implementation project(path: ':applemodule', configuration: 'configA')

那这些不同的依赖声明，到底有何不同呢？以及它们内部的机制是怎样的？

## 从 implemenation 说起 ##
像如下代码是如何起作用的呢？
```gradle
implemenation 'com.android.support:appcompat-v7:25.1.0'
```

按照 `Groovy` 的语法，这里要执行的是 `DependencyHandler` 的 `implementation()` 方法，参数则为 `'com.android.support:appcompat-v7:25.1.0'`。
可是我们可以看到，`DependencyHandler` 中并没有 `implementation()` 这个方法，那么这是怎么回事呢？

### MethodMissing ###
这其实涉及到 `Groovy` 语言的一个重要特性：`methodMissing`，这个特性允许在运行时 `catch` 对于未定义方法的调用。

`Gradle` 对这个特性进行了封装，一个类要想使用这个特性，只要实现 `MixIn` 接口即可，这个接口如下：
```java
/**
 * A decorated domain object type may optionally implement this interface to dynamically expose methods in addition to those declared statically on the type.
 *
 * Note that when a type implements this interface, dynamic Groovy dispatch will not be used to discover opaque methods. That is, methods such as methodMissing() will be ignored.
 */
public interface MethodMixIn {
    MethodAccess getAdditionalMethods();
}
```

其中 `MethodAccess` 接口如下：
```java
/**
 * Provides dynamic access to methods of some object.
 */
public interface MethodAccess {
    /**
     * Returns true when this object is known to have a method with the given name that accepts the given arguments.
     *
     * <p>Note that not every method is known. Some methods may require an attempt invoke it in order for them to be discovered.</p>
     */
    boolean hasMethod(String name, Object... arguments);

    /**
     * Invokes the method with the given name and arguments.
     */
    DynamicInvokeResult tryInvokeMethod(String name, Object... arguments);
}
```

也就是说，对于 `DependeancyHandler` 中未定义的方法(如 `implementation()` 方法)，只要 `hasMeethod()` 返回 `true`，就最终会调用到 `MethodAccess` 的实现者的 `tryInvokeMethod()` 方法中，其中 `name` 为 `configuration` 名称，`argusments` 就是 `'com.android.support:appcompat-v7:25.1.0'` 这个参数。

那 `DependencyHandler` 接口的实现者 `DefaultDependencyHandler` 是如何实现 `MethodMixIn` 这个接口的呢？
```java
    @Override
    public MethodAccess getAdditionalMethods() {
        return dynamicMethods;
    }
```

非常简单，就是直接返回 `dynamicMethods` 这个成员，而 `dynamicMethods` 的赋值在 `DefaultDependencyHandler` 的构造方法中，如下：
```java
    public DefaultDependencyHandler(ConfigurationContainer configurationContainer,
                                    DependencyFactory dependencyFactory,
                                    ProjectFinder projectFinder,
                                    DependencyConstraintHandler dependencyConstraintHandler,
                                    ComponentMetadataHandler componentMetadataHandler,
                                    ComponentModuleMetadataHandler componentModuleMetadataHandler,
                                    ArtifactResolutionQueryFactory resolutionQueryFactory,
                                    AttributesSchema attributesSchema,
                                    VariantTransformRegistry transforms,
                                    Factory<ArtifactTypeContainer> artifactTypeContainer,
                                    NamedObjectInstantiator namedObjectInstantiator,
                                    PlatformSupport platformSupport) {
        this.configurationContainer = configurationContainer;
        this.dependencyFactory = dependencyFactory;
        this.projectFinder = projectFinder;
        this.dependencyConstraintHandler = dependencyConstraintHandler;
        this.componentMetadataHandler = componentMetadataHandler;
        this.componentModuleMetadataHandler = componentModuleMetadataHandler;
        this.resolutionQueryFactory = resolutionQueryFactory;
        this.attributesSchema = attributesSchema;
        this.transforms = transforms;
        this.artifactTypeContainer = artifactTypeContainer;
        this.namedObjectInstantiator = namedObjectInstantiator;
        this.platformSupport = platformSupport;
        configureSchema();
        dynamicMethods = new DynamicAddDependencyMethods(configurationContainer, new DirectDependencyAdder());
    }
```

而 `DynamicAddDependencyMethods` 类定义如下：
```java
class DynamicAddDependencyMethods implements MethodAccess {
    private ConfigurationContainer configurationContainer;
    private DependencyAdder dependencyAdder;

    DynamicAddDependencyMethods(ConfigurationContainer configurationContainer, DependencyAdder dependencyAdder) {
        this.configurationContainer = configurationContainer;
        this.dependencyAdder = dependencyAdder;
    }

    @Override
    public boolean hasMethod(String name, Object... arguments) {
        return arguments.length != 0 && configurationContainer.findByName(name) != null;
    }

    @Override
    public DynamicInvokeResult tryInvokeMethod(String name, Object... arguments) {
        if (arguments.length == 0) {
            return DynamicInvokeResult.notFound();
        }
        Configuration configuration = configurationContainer.findByName(name);
        if (configuration == null) {
            return DynamicInvokeResult.notFound();
        }

        List<?> normalizedArgs = CollectionUtils.flattenCollections(arguments);
        if (normalizedArgs.size() == 2 && normalizedArgs.get(1) instanceof Closure) {
            return DynamicInvokeResult.found(dependencyAdder.add(configuration, normalizedArgs.get(0), (Closure) normalizedArgs.get(1)));
        } else if (normalizedArgs.size() == 1) {
            return DynamicInvokeResult.found(dependencyAdder.add(configuration, normalizedArgs.get(0), null));
        } else {
            for (Object arg : normalizedArgs) {
                dependencyAdder.add(configuration, arg, null);
            }
            return DynamicInvokeResult.found();
        }
    }

    interface DependencyAdder<T> {
        T add(Configuration configuration, Object dependencyNotation, Closure configureAction);
    }
}
```

注意到它是实现了 `MethodAccess` 这个接口的，首先看它的 `hasMethod()` 方法，很简单，返回 `true`的条件是：
 - 参数长度不为0
 - configuration 必须是已经定义过的

然后再看 `tryInvokeMethod()`，它会先通过 `configurationsContainer` 找到对应的 `configuration`，然后分如下几种情况：
 - 参数个数为2，并且第2个参数是 Closure
 - 参数个数为1
 - 其他情形

不过不管哪种情形，都会先调用 `dependencyAdder` 的 `add()` 方法，而 `dependencyAdder` 是 `DefaultDependencyHandler.DirectDependencyAdder` 对象，其 `add()`方法如下：
```java
private class DirectDependencyAdder implements DynamicAddDependencyMethods.DependencyAdder<Dependency> {

    @Override
    public Dependency add(Configuration configuration, Object dependencyNotation, @Nullable Closure configureAction) {
        return doAdd(configuration, dependencyNotation, configureAction);
    }
}
```

可见，其实是调用外部类 `DefaultDependencyHandler` 的 `doAdd()` 方法，在下一小节分析该方法。

### DefaultDependencyHandler.doAdd() 方法分析 ###
`DefaultDependencyHandler` 的 `doAdd()` 方法如下：
```java
    private Dependency doAdd(Configuration configuration, Object dependencyNotation, Closure configureClosure) {
        if (dependencyNotation instanceof Configuration) {
            Configuration other = (Configuration) dependencyNotation;
            if (!configurationContainer.contains(other)) {
                throw new UnsupportedOperationException("Currently you can only declare dependencies on configurations from the same project.");
            }
            configuration.extendsFrom(other);
            return null;
        }

        Dependency dependency = create(dependencyNotation, configureClosure);
        configuration.getDependencies().add(dependency);
        return dependency;
    }
```

可见，这里会先判断 `dependencyNotation` 是否为 `Configuration`，如果是的话，就让当前的 `configuration` 继承自 `other` 这个 `configuration`，而继承的意思就是，后续所有添加到 `other` 的依赖，也会添加到当前这个 `configuration` 中。

为什么还要考虑参数中的 `dependencyNotation` 是否为 `Configuration` 的情形呢？

其实就是考虑到有诸如 `implementation project(path: ':applemodule', configuration: 'configA')` 这样的依赖声明。

至于依赖的创建过程，在下一节进行分析。

## 依赖创建过程分析 ##
`DefaultDependencyHandler` 的 `create()` 方法如下：
```java
    @Override
    public Dependency create(Object dependencyNotation) {
        return create(dependencyNotation, null);
    }

    @Override
    public Dependency create(Object dependencyNotation, Closure configureClosure) {
        Dependency dependency = dependencyFactory.createDependency(dependencyNotation);
        return ConfigureUtil.configure(configureClosure, dependency);
    }
```

其中的 `dependencyFactory` 为 `DefaultDependencyFactory` 对象，其 `createDependency()` 方法如下:
```java
    @Override
    public Dependency createDependency(Object dependencyNotation) {
        return dependencyNotationParser.parseNotation(dependencyNotation);
    }
```
可见，它是直接调用 `dependencyNotationParser` 这个解析器对于 `dependencyNotation` 进行解析。其中的 `dependencyNotationParser` 是实现了接口 `NotationParser<Object, Dependency>` 接口的对象。

为了找出这里的 `dependencyNotationParser` 到底是哪个类的实例，查看 `DependencyManagementBuildScopeServices` 中 `DefaultDependencyFactory` 的创建，如下：
```java
    DependencyFactory createDependencyFactory(
        Instantiator instantiator,
        ProjectAccessListener projectAccessListener,
        StartParameter startParameter,
        ClassPathRegistry classPathRegistry,
        CurrentGradleInstallation currentGradleInstallation,
        FileLookup fileLookup,
        RuntimeShadedJarFactory runtimeShadedJarFactory,
        ImmutableAttributesFactory attributesFactory,
        SimpleMapInterner stringInterner) {
        NotationParser<Object, Capability> capabilityNotationParser = new CapabilityNotationParserFactory(false).create();
        DefaultProjectDependencyFactory factory = new DefaultProjectDependencyFactory(
            projectAccessListener, instantiator, startParameter.isBuildProjectDependencies(), capabilityNotationParser, attributesFactory);
        ProjectDependencyFactory projectDependencyFactory = new ProjectDependencyFactory(factory);

        return new DefaultDependencyFactory(
            DependencyNotationParser.parser(instantiator, factory, classPathRegistry, fileLookup, runtimeShadedJarFactory, currentGradleInstallation, stringInterner),
                DependencyConstraintNotationParser.parser(instantiator, factory, stringInterner),
                new ClientModuleNotationParserFactory(instantiator, stringInterner).create(),
                capabilityNotationParser, projectDependencyFactory,
            attributesFactory);
    }
```
可见，它是通过 `DependencyNotationParser.parser()` 方法创建的，该方法如下:
```java
public class DependencyNotationParser {
    public static NotationParser<Object, Dependency> parser(Instantiator instantiator, DefaultProjectDependencyFactory dependencyFactory, ClassPathRegistry classPathRegistry, FileLookup fileLookup, RuntimeShadedJarFactory runtimeShadedJarFactory, CurrentGradleInstallation currentGradleInstallation, Interner<String> stringInterner) {
        return NotationParserBuilder
            .toType(Dependency.class)
            .fromCharSequence(new DependencyStringNotationConverter<DefaultExternalModuleDependency>(instantiator, DefaultExternalModuleDependency.class, stringInterner))
            .converter(new DependencyMapNotationConverter<DefaultExternalModuleDependency>(instantiator, DefaultExternalModuleDependency.class))
            .fromType(FileCollection.class, new DependencyFilesNotationConverter(instantiator))
            .fromType(Project.class, new DependencyProjectNotationConverter(dependencyFactory))
            .fromType(DependencyFactory.ClassPathNotation.class, new DependencyClassPathNotationConverter(instantiator, classPathRegistry, fileLookup.getFileResolver(), runtimeShadedJarFactory, currentGradleInstallation))
            .invalidNotationMessage("Comprehensive documentation on dependency notations is available in DSL reference for DependencyHandler type.")
            .toComposite();
    }
}
```

这个方法其实很好理解，它其实是创建了多个实现了接口 `NotationConverter` 的对象，然后将这些转换器都添加在一起，构成一个综合的转换器。
其中，
`DependencyStringNotationConverter` 负责将字符串类型的 `notation`转换为 `DefaultExternalModuleDependency`，也就是对应 `implementation 'com.android.support:appcompat-v7:25.1.0'` 这样的声明；
`DependencyFilesNotationConverter` 将 `FileCollection` 转换为 `SelfResolvingDependency`，也就是对应 `implementation fileTree(dir:'libs', include:['*.jar'])` 这样的声明；
`DependencyProjectNotationConverter` 将 `Project` 转换为 `ProjectDependency`，对应 `implementation project(":applemodule")` 这样的情形；
`DependencyClasspathNotationConverter` 将 `ClasspathNotation` 转换为 `SelfResolvingDependency`。

到这里，就知道类似 `compile 'com.android.support:appcompat-v7:25.1.0'`、`implementation project(':applemodule')` 这样的声明，其实是被不同的转换器，转换成了 `SelfResolvingDependency` 或者 `ProjectDependency`。

这里可以看出，除了 `project` 依赖之外，其他都转换成 `SelfResolvingDependency`，所谓的 `SelfResolvingDependency` 其实是可以自解析的依赖，独立于 `repository`。

`ProjectDependency` 则不然，它与依赖于 `repository` 的，下面就分析 `ProjectDependency` 的独特之处。

## DependencyHandler.project() 方法分析 ##
### ProjectDependency 的创建过程 ###
`DependencyHandler.project()` 方法是为了添加 `project` 依赖，而 `DefaultDependencyHandler.project()` 方法如下：
```java
    @Override
    public Dependency project(Map<String, ?> notation) {
        return dependencyFactory.createProjectDependencyFromMap(projectFinder, notation);
    }
```

其中 `dependencyFactory` 为 `DefaultDependencyFactory` 对象，其 `createProjectDependencyFromMap()` 方法如下：
```java
    @Override
    public ProjectDependency createProjectDependencyFromMap(ProjectFinder projectFinder, Map<? extends String, ? extends Object> map) {
        return projectDependencyFactory.createFromMap(projectFinder, map);
    }
```

其中的 `projectDependencyFactory` 为 `ProjectDependencyFactory` 对象，其 `createFromMap()` 方法如下：
```java
    public ProjectDependency createFromMap(ProjectFinder projectFinder, Map<? extends String, ?> map) {
        return NotationParserBuilder.toType(ProjectDependency.class)
                .converter(new ProjectDependencyMapNotationConverter(projectFinder, factory)).toComposite().parseNotation(map);
    }
```

可见，它其实是依靠 `ProjectDependencyMapNotationConverter` 这个转换器实现将 `project` 转换为 `ProjectDependency` 的，而 `ProjectDependencyMapNotationConverter` 的定义非常简单：
```java
static class ProjectDependencyMapNotationConverter extends MapNotationConverter<ProjectDependency> {

    private final ProjectFinder projectFinder;
    private final DefaultProjectDependencyFactory factory;

    public ProjectDependencyMapNotationConverter(ProjectFinder projectFinder, DefaultProjectDependencyFactory factory) {
        this.projectFinder = projectFinder;
        this.factory = factory;
    }

    protected ProjectDependency parseMap(@MapKey("path") String path, @Optional @MapKey("configuration") String configuration) {
        return factory.create(projectFinder.getProject(path), configuration);
    }

    @Override
    public void describe(DiagnosticsVisitor visitor) {
        visitor.candidate("Map with mandatory 'path' and optional 'configuration' key").example("[path: ':someProj', configuration: 'someConf']");
    }
}
```

显然，就是先通过 `projectFinder` 找到相应的 `Project`，然后通过 `factory` 创建 `ProjectDependency`，其中的 `factory` 为 `DefaultProjectDependencyFactory`，其定义如下：
```java
public class DefaultProjectDependencyFactory {
    private final ProjectAccessListener projectAccessListener;
    private final Instantiator instantiator;
    private final boolean buildProjectDependencies;
    private final NotationParser<Object, Capability> capabilityNotationParser;
    private final ImmutableAttributesFactory attributesFactory;

    public DefaultProjectDependencyFactory(ProjectAccessListener projectAccessListener, Instantiator instantiator, boolean buildProjectDependencies, NotationParser<Object, Capability> capabilityNotationParser, ImmutableAttributesFactory attributesFactory) {
        this.projectAccessListener = projectAccessListener;
        this.instantiator = instantiator;
        this.buildProjectDependencies = buildProjectDependencies;
        this.capabilityNotationParser = capabilityNotationParser;
        this.attributesFactory = attributesFactory;
    }

    public ProjectDependency create(ProjectInternal project, String configuration) {
        DefaultProjectDependency projectDependency = instantiator.newInstance(DefaultProjectDependency.class, project, configuration, projectAccessListener, buildProjectDependencies);
        projectDependency.setAttributesFactory(attributesFactory);
        projectDependency.setCapabilityNotationParser(capabilityNotationParser);
        return projectDependency;
    }

    public ProjectDependency create(Project project) {
        return instantiator.newInstance(DefaultProjectDependency.class, project, projectAccessListener, buildProjectDependencies);
    }
}
```
显然，就是根据传入的 `project` 和 `configuration` 名称，创建 `DefaultProjectDependency` 对象。

### project 依赖到底是如何体现的 ###
其实与 `configuration` 息息相关。

注意 `DefaultProjectDependency` 中的 `getBuildDependencies()` 方法：
```java
    @Override
    public TaskDependencyInternal getBuildDependencies() {
        return new TaskDependencyImpl();
    }
```

`TaskDependencyImpl` 是一个内部类，其定义如下：
```java
private class TaskDependencyImpl extends AbstractTaskDependency {
    @Override
    public void visitDependencies(TaskDependencyResolveContext context) {
        if (!buildProjectDependencies) {
            return;
        }
        projectAccessListener.beforeResolvingProjectDependency(dependencyProject);

        Configuration configuration = findProjectConfiguration();
        context.add(configuration);
        context.add(configuration.getAllArtifacts());
    }
}
```
其中 `findProjectConfiguration()`方法如下：
```java
    @Override
    public Configuration findProjectConfiguration() {
        ConfigurationContainer dependencyConfigurations = getDependencyProject().getConfigurations();
        String declaredConfiguration = getTargetConfiguration();
        Configuration selectedConfiguration = dependencyConfigurations.getByName(GUtil.elvis(declaredConfiguration, Dependency.DEFAULT_CONFIGURATION));
        if (!selectedConfiguration.isCanBeConsumed()) {
            throw new ConfigurationNotConsumableException(dependencyProject.getDisplayName(), selectedConfiguration.getName());
        }
        return selectedConfiguration;
    }
```

这个方法的含义是，如果依赖 `project` 时指定了 `configuration`(比如 `implementation project(":applemodule")` 时的 `implementation`)，那就获取 `implementation` 这个 `configuration`，如果没有，那就使用 `default` 这个 `configuration`。

再回到 `TaskDependencyImpl` 类中，注意如下两个调用：
```java
    context.add(configuration);
    context.add(configuration.getAllArtifacts());
```
这两个语句的真实含义如下：
1. `Configuration` 实现了 `FileCollection` 接口，而 `FileCollection` 继承自 `Buildable`，所以 `context.add(configuration);` 是将其作为一个 `Buildable` 对象添加进去。其中 `configuration` 是 `DefaultConfiguration` 对象，它实现了 `getBuildDependencies()` 方法，如下：
```java
    public TaskDependency getBuildDependencies() {
        this.assertIsResolvable();
        return this.intrinsicFiles.getBuildDependencies();
    }
```

2. `context.add(configuration.getAllArtifacts());`这个，则是因为 `configuration.getAllArtifacts()` 获得的是 `DefaultPublishArtifactSet` 对象，而 `DefaultPublishArtifactSet` 也实现了 `Buildable` 接口，其 `getBuildDependencies()` 方法如下：
```java
    public TaskDependency getBuildDependencies() {
        return this.builtBy;
    }
```
其中的 `builtBy` 是内部类 `ArtifactsTaskDependency` 对象，而 `ArtifactsTaskDependency` 定义如下：
```java
private class ArtifactsTaskDependency extends AbstractTaskDependency {
    @Override
    public void visitDependencies(TaskDependencyResolveContext context) {
        for (PublishArtifact publishArtifact : DefaultPublishArtifactSet.this) {
            context.add(publishArtifact);
        }
    }
}
```
可见，这里是直接将 `PublishArtifact` 对象添加到 `context` 中，而 `PublishArtifact`中含有编译依赖。

关于 `artifacts` 以及 `PublishArtifact` 的知识点，会在第3篇中进行详细分析，敬请期待。

## 总结 ##
经过本文的分析，可得出如下结论：
 - `DependencyHandler` 是没有 `implementation()`、`api()`、`compile()` 这些方法的，是通过 `MethodMissing` 机制，间接地调用 `DependencyHandler` 的实现 `DefaultDependencyHandler` 的 `add()` 方法将依赖添加进去的；
 - 如果 `dependencyNotation` 中含有 `configuration`(如configA)，则让当前的 `configuration`(如configB)继承这个 `configuration`，意思就是后续所有添加到 `configA` 的依赖，都会添加到 `configB` 中；
 - 不同的依赖声明，其实是由不同的转换器进行转换的，比如 `DependencyStringNotationConverter` 负责将类似 `"org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"` 这样的依赖声明转换为依赖，`DependencyProjectNotationConverter` 负责将 `project(':applemodule')` 这样的依赖声明转换为依赖；
 - 除了 `project` 依赖之外，其他的依赖最终都转换为 `SelfResolvingDependency`，即可自解析的依赖；
 - `project` 依赖的本质是 `artifacts` 依赖。

## 致谢 ##
[字节跳动技术团队](https://mp.weixin.qq.com/s/WQCqUaYDPHDIjUHlxjRIkw)
