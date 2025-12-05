---
title: Spring Cloud Eureka注册中心的安全加固策略与实施
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Cloud
---

## 强化Eureka注册中心的安全性：集成登录认证机制

默认情况下，Eureka注册中心的管理控制台无需任何身份验证即可直接访问，同时任何微服务也都可以自由注册到该中心。这种开放状态存在显著的安全隐患。本章将指导你如何为Eureka注册中心添加登录认证功能，从而构建一个受保护的安全访问环境。

### 第一步：引入Spring Security安全框架

在项目配置文件中添加Spring Security依赖：

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### 第二步：配置认证凭据信息

在`application.yml`配置文件中设置登录所需的用户名和密码：

```yaml
spring:
  security:
    user:
      name: javastack
      password: javastack
```

### 第三步：更新服务注册地址配置

需要在Eureka客户端配置的`defaultZone`中添加身份验证信息，格式为：`http://用户名:密码@主机:端口/eureka/`

具体配置示例：

```yaml
defaultZone: http://javastack:javastack@eureka1:8761/eureka/,
  http://javastack:javastack@eureka2:8762/eureka/
```

### 第四步：调整安全策略以支持Eureka注册

启用Spring Security后，默认会开启CSRF（跨站请求伪造）防护机制，这可能导致服务实例注册异常，如下图所示的`unavailable-replicas`
问题：

![](img/20190328114313.png)

参考Spring Cloud官方文档中关于保护Eureka服务器的章节，我们需要针对Eureka相关端点禁用CSRF防护：

```java

@EnableWebSecurity
class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().ignoringAntMatchers("/eureka/**");
        super.configure(http);
    }
}
```

完成上述配置后，访问Eureka控制台时将首先跳转到登录页面，只有通过身份验证才能查看注册中心信息，如下图所示：

![](img/20190328145822.png)

通过这一系列安全加固措施，你的Eureka注册中心将获得基本的访问控制保护，有效防止未经授权的访问和注册行为，为微服务架构增添一道安全防线。