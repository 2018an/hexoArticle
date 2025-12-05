---
title: Spring Boot 2.x 启动流程深度剖析：从入口类开始源码追踪
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Boot
---

## Spring Boot应用启动流程深度剖析

关于Spring Boot的应用开发，我们已经探讨了许多实用主题。今天，让我们深入其内部机制，通过源码解析来探索这个框架为何能如此简化开发流程。

本文基于Spring Boot 2.0.3版本展开分析，阅读前需要具备Java和Spring框架的基础知识。如果你还不了解Spring
Boot的基本概念，建议先学习相关入门教程。

### 应用启动入口的奥秘

标准Spring Boot应用的入口类通常如下所示：

```java

@SpringBootApplication
public class ApplicationLauncher {

    public static void main(String[] args) {
        SpringApplication.run(ApplicationLauncher.class, args);
    }
}
```

这是Spring Boot应用的标准启动模式。入口类通常位于项目的根包路径下，包含一个main方法。`@SpringBootApplication`注解用于激活Spring
Boot的各项特性，而`SpringApplication.run()`方法则是应用启动的起点。

深入查看`run`方法的调用链：

```java
public static ConfigurableApplicationContext run(Class<?> primarySource,
                                                 String... args) {
    return run(new Class<?>[]{primarySource}, args);
}

public static ConfigurableApplicationContext run(Class<?>[] primarySources,
                                                 String[] args) {
    return new SpringApplication(primarySources).run(args);
}
```

参数说明：

- `primarySource`：应用的主配置类
- `args`：传递给应用程序的命令行参数

启动流程分为两个核心阶段：首先使用主配置类创建`SpringApplication`实例，然后调用该实例的`run`方法。接下来我们将分步深入分析。

## SpringApplication实例的构建过程

![](img/18-7-2-69983860.jpg)

跟随源码进入`SpringApplication`构造器：

```java
public SpringApplication(Class<?>... primarySources) {
    this(null, primarySources);
}

public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    // 1. 资源加载器初始化为null
    this.resourceLoader = resourceLoader;

    // 2. 验证主配置类非空
    Assert.notNull(primarySources, "主配置类不能为null");

    // 3. 初始化主配置类集合并去重
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));

    // 4. 推断当前Web应用类型
    this.webApplicationType = deduceWebApplicationType();

    // 5. 设置应用上下文初始化器
    setInitializers((Collection) getSpringFactoriesInstances(
            ApplicationContextInitializer.class));

    // 6. 设置应用事件监听器          
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));

    // 7. 推断主启动类
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

整个构造过程包含以下7个关键步骤：

**1. 资源加载器初始化**
当前场景中资源加载器设置为null。

**2. 主配置类验证**
确保传入的主配置类参数不为null，否则抛出异常。

**3. 主配置类集合初始化**
将传入的多个主配置类转换为有序集合，避免重复。

**4. Web应用类型推断**
根据类路径中存在的特定类来判断应用类型：

```java
private WebApplicationType deduceWebApplicationType() {
    if (ClassUtils.isPresent(REACTIVE_WEB_ENVIRONMENT_CLASS, null)
            && !ClassUtils.isPresent(MVC_WEB_ENVIRONMENT_CLASS, null)) {
        return WebApplicationType.REACTIVE;
    }
    for (String className : WEB_ENVIRONMENT_CLASSES) {
        if (!ClassUtils.isPresent(className, null)) {
            return WebApplicationType.NONE;
        }
    }
    return WebApplicationType.SERVLET;
}
```

应用类型分为三种：非Web应用、Servlet Web应用、响应式Web应用。

**5. 应用上下文初始化器配置**
通过Spring的工厂加载机制从`META-INF/spring.factories`文件中加载并实例化所有`ApplicationContextInitializer`实现。

**6. 应用事件监听器配置**
同样通过工厂机制加载并实例化所有`ApplicationListener`实现。

**7. 主启动类推断**
通过分析当前调用栈，查找包含main方法的类作为主启动类。

## SpringApplication实例的运行过程

![](img/18-7-2-34936854.jpg)

完成实例构建后，调用其`run`方法启动应用：

```java
public ConfigurableApplicationContext run(String... args) {
    // 1. 启动性能计时器
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();

    // 2. 初始化应用上下文和异常报告器集合
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();

    // 3. 配置Headless模式
    configureHeadlessProperty();

    // 4. 创建并启动运行监听器
    SpringApplicationRunListeners listeners = getRunListeners(args);
    listeners.starting();

    try {
        // 5. 初始化应用参数
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);

        // 6. 准备运行环境
        ConfigurableEnvironment environment = prepareEnvironment(listeners,
                applicationArguments);
        configureIgnoreBeanInfo(environment);

        // 7. 创建启动横幅
        Banner printedBanner = printBanner(environment);

        // 8. 创建应用上下文
        context = createApplicationContext();

        // 9. 初始化异常报告器
        exceptionReporters = getSpringFactoriesInstances(
                SpringBootExceptionReporter.class,
                new Class[]{ConfigurableApplicationContext.class}, context);

        // 10. 准备应用上下文
        prepareContext(context, environment, listeners, applicationArguments,
                printedBanner);

        // 11. 刷新应用上下文
        refreshContext(context);

        // 12. 刷新后置处理
        afterRefresh(context, applicationArguments);

        // 13. 停止计时器
        stopWatch.stop();

        // 14. 记录启动日志
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass)
                    .logStarted(getApplicationLog(), stopWatch);
        }

        // 15. 发布应用启动完成事件
        listeners.started(context);

        // 16. 执行Runner组件
        callRunners(context, applicationArguments);
    } catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        // 17. 发布应用运行就绪事件
        listeners.running(context);
    } catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }

    // 18. 返回应用上下文
    return context;
}
```

整个运行过程可概括为18个关键步骤，其中最核心的包括：

**环境准备阶段**

- 创建并配置运行环境
- 根据应用类型创建对应的应用上下文
- 加载所有配置的初始化和监听组件

**上下文初始化阶段**

- 绑定环境配置到上下文
- 执行所有应用上下文初始化器
- 加载所有Bean定义和资源

**应用启动阶段**

- 刷新应用上下文（触发Bean创建和依赖注入）
- 执行所有Runner组件
- 发布系列启动事件通知监听器

**启动完成阶段**

- 记录启动性能指标
- 返回完全初始化的应用上下文

## 启动流程的技术精髓

Spring Boot的启动流程体现了几个重要的设计思想：

**1. 约定优于配置**
通过类路径扫描和条件化配置，自动推断应用类型和所需组件，极大减少了显式配置。

**2. 扩展点丰富**
提供了多个扩展接口（如`ApplicationContextInitializer`、`ApplicationListener`、`SpringApplicationRunListener`
），允许开发者在启动过程的关键节点插入自定义逻辑。

**3. 事件驱动架构**
通过发布系列启动事件（starting、environmentPrepared、contextPrepared、started、running等），实现了松耦合的组件通信机制。

**4. 工厂模式应用**
大量使用Spring的`SpringFactoriesLoader`机制，通过`META-INF/spring.factories`文件实现组件的自动发现和加载。

理解Spring Boot的启动流程，不仅有助于深入掌握框架原理，更能在实际开发中更有效地解决启动相关问题，设计出更优雅的应用架构。虽然源码分析过程充满挑战，但收获的理解深度是无可替代的。