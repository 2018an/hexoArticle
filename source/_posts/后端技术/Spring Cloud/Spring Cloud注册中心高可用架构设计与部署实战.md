---
title: Spring Cloud注册中心高可用架构设计与部署实战
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Cloud
---

## 服务注册中心的选型与搭建

在 Spring Cloud 微服务架构中，服务注册与发现是实现服务治理的核心环节。常见的注册中心实现方案包括
Eureka、Consul、Zookeeper、ETCD 等。本文推荐使用 Spring Cloud Eureka 来构建注册中心，它在 Netflix Eureka
的基础上进行了深度集成与封装，为分布式系统提供了完整的服务治理能力。整个微服务体系中所有服务的注册、发现与管理，都将通过此中心节点来协调完成。

## 第一步：添加 Eureka 服务端依赖

在已有的 Spring Cloud 项目依赖基础上，引入 Eureka Server 的专用依赖包。

```xml

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka-server</artifactId>
    </dependency>
</dependencies>
```

## 第二步：创建应用启动类，激活服务端功能

在项目的根包路径下，创建主启动类，并添加关键注解以启用注册中心服务。

```java

@EnableEurekaServer
@SpringBootApplication
public class RegisterApplication {
    public static void main(String[] args) {
        new SpringApplicationBuilder(RegisterApplication.class)
                .bannerMode(Banner.Mode.LOG)
                .run(args);
    }
}
```

其中，`@EnableEurekaServer` 注解用于声明当前应用作为 Eureka 服务端运行。

## 第三步：配置 Eureka 服务参数

在 `application.yml` 配置文件中，进行如下详细设置，此处以配置一个高可用集群为例。

```yaml
spring:
  application:
    name: service-registry-center
  profiles:
    active: register-center1

eureka:
  instance:
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ipAddress}:${server.port}
    lease-expiration-duration-in-seconds: ${lease-expiration-duration-in-seconds}
    lease-renewal-interval-in-seconds: ${lease-renewal-interval-in-seconds}
  server:
    enable-self-preservation: ${enable-self-preservation}
    eviction-interval-timer-in-ms: ${eviction-interval-timer-in-ms}
  client:
    register-with-eureka: true
    fetch-registry: true
    serviceUrl:
      defaultZone: ${register-center.urls}

---
spring:
  profiles: register-center1
server:
  port: ${register-center1.server.port}

---
spring:
  profiles: register-center2
server:
  port: ${register-center2.server.port}
```

以上配置定义了一个包含两个节点（`register-center1` 和 `register-center2`
）的高可用集群。每个节点都将自己注册到对方，从而实现相互备份与状态同步。此模式可以轻松扩展为多个节点。

## 第四步：通过 Maven 资源过滤管理环境配置

配置中使用 `${}` 占位符引用的变量，将通过 Maven 资源过滤机制在打包时被替换。这便于为不同环境（如开发、测试、生产）加载相应的配置参数。

例如，针对开发环境的 `filter-dev.properties` 配置文件内容可参考如下：

```
# 服务器地址
register-center1.server.ip=192.168.1.22
register-center2.server.ip=192.168.1.23
register-center.urls=http://${register-center1.server.ip}:${register-center1.server.port}/eureka/,http://${register-center2.server.ip}:${register-center2.server.port}/eureka/

# 服务端口
register-center1.server.port=7001
register-center2.server.port=7002

# Eureka 核心参数
enable-self-preservation=false
eviction-interval-timer-in-ms=5000
lease-expiration-duration-in-seconds=20
lease-renewal-interval-in-seconds=6
```

## 第五步：关键配置项说明

以下对 Spring Cloud Eureka 相关的核心配置进行解释（关于 Spring Boot 通用配置请参考其他资料）：

- **`spring.application.name`**：指定应用在注册中心显示的服务名称。
- **`spring.cloud.client.ipAddress`**：自动获取应用所在主机的 IP 地址。
- **`eureka.instance.prefer-ip-address`**：设置为 `true` 表示优先使用 IP 地址进行注册与发现。在生产环境中，这通常比使用主机名更可靠。
- **`eureka.instance.instance-id`**：定义实例在注册中心的唯一标识符。
- **`eureka.instance.lease-expiration-duration-in-seconds`**：定义 Eureka
  服务端在最后一次收到心跳后，需要等待多少秒才会将此实例从注册表中移除并停止向其路由流量。设置时间过长可能导致流量被路由到已下线的实例；设置过短则可能因为短暂的网络波动导致健康实例被误剔除。该值必须大于
  `lease-renewal-interval-in-seconds` 的设置。
- **`eureka.instance.lease-renewal-interval-in-seconds`**：定义 Eureka
  客户端向服务端发送心跳以证明自己存活的间隔时间（秒）。若服务端在此间隔指定的时间窗口内未收到心跳，则会触发实例移除流程。注意，即使实例发送心跳，如果其健康检查回调（HealthCheckCallback）指示自身状态为不可用，流量依然不会被路由到该实例。
- **`eureka.server.enable-self-preservation`**：是否启用服务端的自我保护模式。当网络分区故障发生时，该模式可以防止因大量实例心跳超时而错误地注销健康服务。
- **`eureka.server.eviction-interval-timer-in-ms`**：服务端清理无效实例的后台任务执行间隔（毫秒）。默认值为 60000 毫秒（即 1
  分钟）。
- **`eureka.client.register-with-eureka`**：是否将本实例自身的信息注册到 Eureka 服务器以供其他服务发现。在某些特定场景下（如仅作为查询客户端的应用），可设置为
  `false`。
- **`eureka.client.fetch-registry`**：客户端是否从 Eureka 服务器获取服务注册表的副本并缓存到本地。
- **`eureka.client.serviceUrl.defaultZone`**：指定 Eureka 服务器集群的地址列表，以逗号分隔。

## 第六步：启动高可用注册中心集群

完成以上配置后，即可分别启动两个注册中心节点。通过指定不同的 Profile 来激活对应的端口配置。

启动命令示例（在项目根目录下执行）：

```
spring-boot:run -Drun.profiles=register-center1 -P dev
spring-boot:run -Drun.profiles=register-center2 -P dev
```

至此，一个由两个节点构成的 Eureka Server 高可用集群便部署完成。微服务中的各个客户端应用可以配置连接到该集群地址，实现服务的注册与发现。