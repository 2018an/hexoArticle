---
title: 构建安全的Spring Cloud微服务间通信连接
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Cloud
---

## 概述

为保障微服务间通信的安全，可以在 Spring Cloud 应用中启用 HTTP Basic 认证机制。这是一种简单有效的身份验证方式，能为服务访问增加一道基础防护。

## 步骤一：引入安全依赖模块

首先，需要在项目的 Maven 构建配置文件（pom.xml）中，添加 Spring Boot Security 的依赖。

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

引入此依赖后，应用便自动开启了基础安全防护。默认情况下，系统会创建一个用户名为 `"user"`
的账户，并生成一个随机的登录密码。该密码会在服务启动时，输出到控制台日志中，请注意查看。

## 步骤二：配置自定义登录凭据

使用随机生成的密码通常不便于记忆和管理。我们可以通过配置文件，指定固定的用户名和密码。

在 `application.yml` 或 `application.properties` 中添加如下配置即可。

```yaml
security:
  user:
    name: admin
    password: admin123456
```

完成此配置后，任何对该服务的访问请求都需要提供正确的用户名和密码进行认证。若认证失败，服务将返回 HTTP 401 状态码及错误信息。

```json
{
  "timestamp": 1502689874556,
  "status": 401,
  "error": "Unauthorized",
  "message": "Bad credentials",
  "path": "/test/save"
}
```

## 步骤三：配置服务间的安全访问

1. **访问受保护的服务注册中心**
   当注册中心（如Eureka Server）开启了安全认证，客户端在配置注册地址时，需要将用户名和密码包含在URL中。
   格式为：`username:password@ipaddress`

2. **配置 Feign 客户端的认证信息**
   对于使用 Feign 进行声明的服务调用，需要创建一个配置类，为其注入基本的认证拦截器。

   **服务接口定义示例：**
   ```java
   @FeignClient(name = "SERVICE", configuration = FeignAuthConfig.class)
   public interface OrderService extends OrderAPI {
   }
   ```

   **对应的 Feign 认证配置类：**
   ```java
   @Configuration
   public class FeignAuthConfig {
       @Bean
       public BasicAuthRequestInterceptor basicAuthRequestInterceptor() {
           return new BasicAuthRequestInterceptor("admin", "admin123456");
       }
   }
   ```
   通过以上配置，该 Feign 客户端在发起请求时，会自动在请求头中添加认证信息。