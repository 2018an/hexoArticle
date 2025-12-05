---
title: Spring Cloud 配置中心客户端读取配置
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Cloud
---

## 接入外部配置管理服务

在微服务架构中，通常需要将应用程序的配置信息集中管理。通过让微服务客户端连接到统一的配置中心，可以实现外部化配置的集中读取与动态更新。

## 添加必要的功能模块

首先，在项目的依赖管理文件（如pom.xml）中，引入以下必要的组件。

```xml

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.retry</groupId>
        <artifactId>spring-retry</artifactId>
    </dependency>
</dependencies>
```

**依赖说明：**

- `spring-cloud-starter-config`：这是配置中心客户端核心依赖，使服务能够从远程配置服务器获取配置。
- `spring-boot-starter-aop` 与 `spring-retry`
  ：这两个依赖为客户端提供了增强的容错能力。它们支持在连接配置中心失败时进行快速失败（fail-fast）判断，并自动执行重试逻辑，提高系统启动的可靠性。

## 创建主程序入口类

在项目源代码的根包路径下，创建应用的主启动类，并添加必要的注解来启用服务发现功能。

```java

@EnableDiscoveryClient
@SpringBootApplication
public class ServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServiceApplication.class, args);
    }
}
```

## 配置客户端参数

微服务客户端需要一个特殊的引导配置文件来初始化与配置中心的连接。**请注意，此配置必须置于 `bootstrap.yml`
（或 `bootstrap.properties`）中，在 `application.yml` 中配置将无法生效。**

**`bootstrap.yml` 配置示例：**

```yaml
spring:
  application:
    name: config-client
  cloud:
    config:
      # 如果配置中心启用了安全认证，需填写用户名和密码
      # username:
      # password:
      name: ${git.application}      # 对应远程配置文件的名称前缀
      profile: ${git.profile}       # 指定激活的环境配置（如dev, prod）
      label: ${git.label}           # 指定Git仓库的分支（如master）
      fail-fast: true               # 启用快速失败：连接失败则立即报错，而非使用本地缓存
      retry:
        initial-interval: 2000      # 首次重试的间隔时间（毫秒）
        max-attempts: 5             # 最大重试次数
      discovery:
        enabled: true               # 通过服务发现（如Eureka）查找配置中心
        service-id: config-center   # 配置中心在注册中心的服务名

eureka:
  client:
    serviceUrl:
      defaultZone: ${register-center.urls} # 注册中心的地址
```

**`application.yml` 配置示例（主要用于定义应用自身属性）：**

```yaml
spring:
  profiles:
    active: config-client1 # 激活profile，决定使用哪一组端口等配置

eureka:
  instance:
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ipAddress}:${server.port}
    lease-expiration-duration-in-seconds: ${lease-expiration-duration-in-seconds}
    lease-renewal-interval-in-seconds: ${lease-renewal-interval-in-seconds}

---
spring:
  profiles: config-client1
server:
  port: ${config-client1.server.port}

---
spring:
  profiles: config-client2
server:
  port: ${config-client2.server.port}
```

## 通过Maven管理环境变量

使用Maven资源过滤机制，将环境相关的变量（如Git分支、环境标识）从外部属性文件注入。

在 `filter-dev.properties` 等环境配置文件中，可以这样定义：

```
...
# Git仓库配置相关参数
git.application=application
git.profile=dev
git.label=master
...
```

## 在代码中获取远程配置

客户端应用启动并成功连接到配置中心后，即可像读取本地配置一样，使用 `@Value` 注解注入远程配置文件中的属性。

```java

@RestController
public class TestController {
    @Value("${username}") // 此值来自远程配置中心
    private String username;
    // ... 其他代码
}
```

除了 `@Value` 注解，还可以使用 `@ConfigurationProperties` 等方式进行绑定，具体方法可参考 Spring Boot 配置管理的相关文档。

## 启动微服务实例

最后，通过指定不同的 Profile 来启动多个服务实例。它们都将从同一个配置中心获取各自的配置信息。

启动命令示例（在项目根目录执行）：

```
spring-boot:run -Drun.profiles=config-client1 -P dev
spring-boot:run -Drun.profiles=config-client2 -P dev
```

启动后，这两个服务实例会向注册中心注册自己，并从名为 `config-center` 的配置中心服务拉取 `application-dev.yml`
等配置文件完成自身的配置初始化。