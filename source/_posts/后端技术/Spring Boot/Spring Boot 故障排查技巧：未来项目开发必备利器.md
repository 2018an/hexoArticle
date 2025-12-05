---
title: Spring Boot 故障排查技巧：未来项目开发必备利器
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Boot
---

## 当 Spring Boot 项目无法 Debug 时，我是这样解决的

最近在本地启动 Spring Boot 应用时，发现断点怎么都进不去，明明代码没动，却突然无法调试了。仔细一查才发现，原来是 Spring Boot
版本升级之后，配套的 Maven 插件命令和参数也悄悄变了——这确实是一个容易踩的坑。

所以，今天就把完整的设置过程重新梳理一遍，如果你也遇到类似问题，可以跟着一步步来。

### 问题出在哪儿？

通常，我们使用 Spring Boot Maven 插件运行 `mvn spring-boot:run` 时，应用默认是以 **fork 进程** 的方式启动的。这意味着 IDE
里的调试器无法直接附加到该进程上，因此你打的断点自然就不会生效。

官方文档里其实已经给出了提示：

> 默认情况下，`run` 目标在一个分支进程中运行你的应用。如果需要调试，你应该添加必要的 JVM 参数来启用远程调试。

下面我们就直接进入正题，看看具体怎么做。


### 第一步：给 JVM 注入“调试灵魂”

核心思路，就是让 Spring Boot Maven 插件在启动时，带上支持远程调试的 JVM 参数。

**方法一：在 `pom.xml` 中固定配置**

在项目的 `spring-boot-maven-plugin` 插件配置区域，加入 `<jvmArguments>` 参数。

```xml

<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>2.6.3</version> <!-- 请改为你的实际版本 -->
    <configuration>
        <jvmArguments>
            -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005
        </jvmArguments>
    </configuration>
</plugin>
```

**方法二：在命令行中动态指定**

如果不想修改 `pom.xml`，也可以在运行命令时直接通过参数传递：

```bash
mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005"
```

![](img/20191030171505.png)

>
最新参数格式或注意事项，建议随时查阅 [Spring Boot 官方文档](https://docs.spring.io/spring-boot/docs/current/maven-plugin/examples/run-debug.html)。

### 第二步：在 IDE 中配置远程调试（Remote）

应用启动后会监听指定的端口（例如上面的 `5005`），我们需要让 IDE 主动连接上去。

以 IntelliJ IDEA 为例：

1. 打开 `Run/Debug Configurations`。
2. 点击 `+` 号，选择 `Remote`。
3. 关键只需配置两项：
    - **Host**: `localhost`（本地运行时）
    - **Port**: `5005`（与 JVM 参数中的 `address` 一致）

![](img/20191030170633.png)


### 第三步：启动并开始调试

这一步的顺序很重要，请严格按照以下流程：

1. **首先**，启动你的 Spring Boot 项目。配置了 JVM 参数后，启动日志会显示类似
   `Listening for transport dt_socket at address: 5005` 的信息，并且**进程会暂停**，等待调试器连接。
   ![](img/20191030174448.png)

2. **接着**，在 IDE 中，以 `Debug` 模式启动刚才配置好的 `Remote`。
   ![](img/20191030174710.png)

3. **当调试器连接成功后**，Spring Boot 应用会继续启动，输出正常的日志。此时，你在 IDE 中设置的断点就能够被正确识别并进入了。


### 最后说两句

这个方法本质上是一种 **“远程调试”** 技术，只不过调试对象是本地启动的另一个 JVM
进程。它的最大好处是通用性强，不仅能解决本地调试问题，未来如果需要调试测试环境或远程服务器上部署的 Spring Boot 应用，只需要调整
`Host` 和防火墙端口即可。

当然，对于纯粹的本地开发来说，步骤确实稍显繁琐。但在某些版本升级或复杂依赖环境下，这却是那个最直接、最可靠的调试手段。希望这篇更新后的指南能帮你重新夺回对代码的“控制权”。