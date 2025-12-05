---
title: 基于Greenwich版本的Spring Cloud Eureka高可用集群部署指南
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Cloud
---

## 构建高可用的Spring Cloud服务发现中枢：Eureka集群实战指南

在Spring Cloud微服务生态中，服务注册中心是架构的核心枢纽。虽然Consul、Zookeeper、ETCD等多种技术都能胜任此角色，但基于Netflix
Eureka二次封装的Spring Cloud Eureka以其成熟稳定、与Spring生态无缝集成的特性，成为众多企业的首选方案。它如同微服务世界的“电话簿”，让每个服务都能找到彼此。

本文将带你从零开始，构建一个真正具备生产级高可用能力的Eureka注册中心集群。单机模式我们就不探讨了，毕竟在生产环境中，高可用才是硬道理。

### 第一步：快速初始化Eureka Server项目骨架

访问Spring官方项目生成器，按照下图所示选择相应配置，特别留意勾选Eureka Server依赖项，系统将自动生成项目基础代码。

> https://start.spring.io/

![](img/微信截图_20190327180739.png)
![](img/微信截图_20190327180726.png)

生成的核心Maven配置文件关键部分如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
    </parent>

    <groupId>cn.javastack</groupId>
    <artifactId>spring-cloud-eureka-server-cluster</artifactId>
    <version>1.0.0</version>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
    </properties>

    <dependencies>
        <!-- Eureka服务端核心依赖 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>

    <!-- Spring Cloud依赖版本管理 -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### 第二步：激活Eureka Server功能

在应用启动类上添加专用注解，开启注册中心服务端能力：

```java

@EnableEurekaServer
@SpringBootApplication
public class RegistryCenterApplication {
    public static void main(String[] args) {
        SpringApplication.run(RegistryCenterApplication.class, args);
    }
}
```

`@EnableEurekaServer`这个注解就如同一个开关，告诉Spring Cloud框架：“我将扮演服务注册中心的角色”。

### 第三步：配置高可用Eureka集群

在`application.yml`中配置集群参数，这里我们搭建一个双节点互备的高可用集群：

```yaml
# 公共基础配置
spring:
  application:
    name: service-registry-cluster

eureka:
  instance:
    prefer-ip-address: false
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
    lease-expiration-duration-in-seconds: 30      # 心跳超时时间
    lease-renewal-interval-in-seconds: 5         # 心跳发送间隔
  server:
    enable-self-preservation: true              # 开启自我保护
    eviction-interval-timer-in-ms: 5000         # 清理失效节点间隔
  client:
    register-with-eureka: true                  # 向集群注册自己
    fetch-registry: true                        # 获取注册表
    serviceUrl:
      defaultZone: http://eureka-node1:8761/eureka/,http://eureka-node2:8762/eureka/

# 降低Eureka内部日志级别
logging.level.com.netflix.eureka: OFF
logging.level.com.netflix.discovery: OFF

# --- 节点1配置 ---
spring:
  profiles: node1
server:
  port: 8761
eureka.instance.hostname: eureka-node1

# --- 节点2配置 ---
spring:
  profiles: node2
server:
  port: 8762
eureka.instance.hostname: eureka-node2
```

**关键注意事项：**

1. 每个Eureka节点都需要向集群中的其他节点注册自己，形成相互备份
2. 切勿使用`localhost`作为主机名，否则集群无法正确识别节点
3. 需要在系统hosts文件中添加映射：
   ```
   127.0.0.1 localhost eureka-node1 eureka-node2
   ```

如果配置不当，你可能会在Eureka控制台的`unavailable-replicas`区域看到节点，这明确表示配置存在问题。

### 第四步：启动并验证Eureka集群

使用不同配置profile启动两个Eureka节点：

```bash
# 启动第一个节点
mvn spring-boot:run -Dspring-boot.run.profiles=node1

# 启动第二个节点
mvn spring-boot:run -Dspring-boot.run.profiles=node2
```

访问以下地址查看集群状态：

> http://localhost:8761/
> http://localhost:8762/

![](img/20190328103743.png)

如图所示，两个Eureka节点已成功互相识别并形成集群。每个节点的控制台都会显示另一个节点的注册信息，标志着高可用注册中心集群搭建完成。

通过这种集群部署方式，即使单个注册中心节点发生故障，整个微服务体系依然能够正常运行，服务发现功能不会中断，真正实现了注册中心的高可用性。这种架构为后续的微服务规模化部署奠定了坚实的基础。