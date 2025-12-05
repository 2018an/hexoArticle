---
title: Dubbo与Spring Boot的强强联合
date: 2025-10-29 17:30:25
category: 后端
tags: Dubbo
---
![image](https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/u=3636281043,3305223769&fm=27&gp=0.jpg)

Dubbo和Spring Boot都是业界公认的优秀框架，如今这两大技术即将实现深度整合。为了降低Dubbo的使用门槛，阿里巴巴Dubbo团队即将推出基于Spring Boot的官方版本，这将大幅简化分布式开发的复杂度，同时提供企业级特性支持，包括安全机制、健康检查、外部化配置等功能。

如果对Dubbo还不太熟悉，建议先访问Dubbo官网（http://dubbo.io）了解其基本概念。

**接下来让我们看看Dubbo与Spring Boot的具体集成方案！**

熟悉Dubbo的开发者都知道，在分布式架构中存在两个关键角色：服务提供者和服务消费者。

#### 服务提供者实现方案

**1、定义服务接口：DemoService**

```java
public interface DemoService {
    String sayHello(String name);
}
```
2、实现服务提供者

```java
@Service(
        version = "1.0.0",
        application = "${dubbo.application.id}",
        protocol = "${dubbo.protocol.id}",
        registry = "${dubbo.registry.id}"
)
public class DefaultDemoService implements DemoService {
    public String sayHello(String name) {
        return "Hello, " + name + " (from Spring Boot)";
    }
}
```
需要注意的是，服务提供者通过@Service注解进行声明，相关配置参数在配置文件中定义。

3、配置文件设置

properties
# Spring Boot应用配置
spring.application.name = dubbo-provider-demo
server.port = 9090
management.port = 9091

# Dubbo组件扫描包路径
dubbo.scan.basePackages = com.alibaba.boot.dubbo.demo.provider.service

# Dubbo应用配置
dubbo.application.id = dubbo-provider-demo
dubbo.application.name = dubbo-provider-demo

# Dubbo协议配置
dubbo.protocol.id = dubbo
dubbo.protocol.name = dubbo
dubbo.protocol.port = 12345

# Dubbo注册中心配置
dubbo.registry.id = my-registry
dubbo.registry.address = N/A
4、服务提供者启动类

```java
@SpringBootApplication
public class DubboProviderDemo {
    public static void main(String[] args) {
        SpringApplication.run(DubboProviderDemo.class, args);
    }
}
```
更多服务提供者示例代码参考：https://github.com/dubbo/dubbo-spring-boot-project/tree/master/dubbo-spring-boot-samples/dubbo-spring-boot-sample-provider

服务消费者实现方案
服务消费者负责调用服务提供者发布的服务接口。

消费者需要通过注入方式获取服务提供者接口的Spring Bean实例。

1、定义服务消费者

```java
@RestController
public class DemoConsumerController {
    
    @Reference(version = "1.0.0",
            application = "${dubbo.application.id}",
            url = "dubbo://localhost:12345")
    private DemoService demoService;

    @RequestMapping("/sayHello")
    public String sayHello(@RequestParam String name) {
        return demoService.sayHello(name);
    }
}
```
@Reference注解用于注入远程服务的Spring Bean实例，相关配置同样在配置文件中定义。

2、消费者配置文件

properties
# Spring Boot应用配置
spring.application.name = dubbo-consumer-demo
server.port = 8080
management.port = 8081

# Dubbo应用配置
dubbo.application.id = dubbo-consumer-demo
dubbo.application.name = dubbo-consumer-demo

# Dubbo协议配置
dubbo.protocol.id = dubbo
dubbo.protocol.name = dubbo
dubbo.protocol.port = 12345
3、消费者启动类

```java
@SpringBootApplication(scanBasePackages = "com.alibaba.boot.dubbo.demo.consumer.controller")
public class DubboConsumerDemo {
    public static void main(String[] args) {
        SpringApplication.run(DubboConsumerDemo.class, args);
    }
}
```
更多消费者示例代码参考：https://github.com/dubbo/dubbo-spring-boot-project/blob/master/dubbo-spring-boot-samples/dubbo-spring-boot-sample-consumer

按照顺序先启动服务提供者，再启动服务消费者，通过访问消费者控制器即可调用远程服务。

更多Spring Boot集成特性请参考官方文档：

项目地址：https://github.com/dubbo/dubbo-spring-boot-project

从以上示例可以看出，Dubbo与Spring Boot的集成使用非常简便，Spring Boot确实极大地提升了开发效率。目前该项目尚未正式发布，我们将持续关注进展，及时为大家带来最新消息。
