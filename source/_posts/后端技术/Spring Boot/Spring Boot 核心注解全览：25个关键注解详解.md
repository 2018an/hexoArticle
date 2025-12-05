---
title: Spring Boot 核心注解全览：25个关键注解详解
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Boot
---

在实践Spring Boot一段时间后，你是否真正熟悉了它的注解体系？今天，我将为你系统梳理Spring Boot中最为关键的25个注解，助你深入理解框架的精髓。

## Spring Boot核心注解全解析

**1. @SpringBootApplication**
作为Spring Boot应用的基石，这个注解标注于主类之上，标志着这是一个Spring Boot应用，负责激活框架的所有核心能力。实质上，它是
`@SpringBootConfiguration`、`@EnableAutoConfiguration`和`@ComponentScan`三个注解的复合体，必要时可用这三个注解替代它。

**2. @EnableAutoConfiguration**
开启自动配置功能的关键注解。启用后，Spring Boot能够依据当前类路径下的依赖自动配置相应的Spring
Bean。例如，当检测到Mybatis相关JAR包时，便会触发`MybatisAutoConfiguration`来初始化Mybatis所需的各类Bean。

**3. @Configuration**
Spring 3.0引入的注解，用于完全替代传统的`applicationContext.xml`配置文件。所有原先在XML中完成的Bean定义和配置，均可转移到被此注解标注的类中实现。

**4. @SpringBootConfiguration**
这是`@Configuration`注解的特定形式，专用于标识Spring Boot的配置类，旨在为框架未来的扩展提供便利。

**5. @ComponentScan**
Spring 3.1加入的注解，取代了XML配置中的`<context:component-scan>`标签。它的作用是自动扫描指定包路径下的组件（如被
`@Component`、`@Service`等标注的类），并将它们注册为Spring容器管理的Bean。

**6. @Conditional**
Spring 4.0新增的条件化配置注解。用于标注一个Spring Bean或配置类，仅当满足指定条件时，相关的配置才会生效。

**7. @ConditionalOnBean**
与`@Conditional`组合使用。仅当Spring容器中存在指定的Bean时，才启用配置。

**8. @ConditionalOnMissingBean**
与`@ConditionalOnBean`逻辑相反。仅当容器中不存在指定的Bean时，才启用配置。

**9. @ConditionalOnClass**
与`@Conditional`组合使用。仅当类路径上存在指定的Class时，才启用配置。

**10. @ConditionalOnMissingClass**
与`@ConditionalOnClass`逻辑相反。仅当类路径上不存在指定的Class时，才启用配置。

**11. @ConditionalOnWebApplication**
与`@Conditional`组合使用。仅当当前应用是WEB类型项目时才启用配置。支持三种项目类型：

- `ANY`：任何Web应用均匹配
- `SERVLET`：仅基于Servlet的Web应用匹配
- `REACTIVE`：仅基于响应式的Web应用匹配

**12. @ConditionalOnNotWebApplication**
与`@ConditionalOnWebApplication`逻辑相反。仅当当前项目不是WEB项目时才启用配置。

**13. @ConditionalOnProperty**
与`@Conditional`组合使用。仅当配置文件中指定的属性拥有特定值时，才启用配置。

**14. @ConditionalOnExpression**
与`@Conditional`组合使用。仅当指定的SpEL表达式求值结果为`true`时，才启用配置。

**15. @ConditionalOnJava**
与`@Conditional`组合使用。仅当运行应用的JVM版本在指定范围内时，才启用配置。

**16. @ConditionalOnResource**
与`@Conditional`组合使用。仅当类路径下存在指定的资源文件时，才启用配置。

**17. @ConditionalOnJndi**
与`@Conditional`组合使用。仅当指定的JNDI（Java命名和目录接口）资源存在时，才启用配置。

**18. @ConditionalOnCloudPlatform**
与`@Conditional`组合使用。仅当运行在指定的云平台（如Cloud Foundry, Kubernetes）时，才启用配置。

**19. @ConditionalOnSingleCandidate**
与`@Conditional`组合使用。仅当容器中指定Class类型的Bean只有一个实例，或者存在多个但已明确指定了首选Bean时，才启用配置。

**20. @ConfigurationProperties**
用于将外部配置文件（如`.properties`或`.yml`文件）中的属性批量注入到Bean中。可用在`@Configuration`标注的类或`@Bean`标注的方法上。


**21. @EnableConfigurationProperties**
通常与`@ConfigurationProperties`配合使用，用于启用对`@ConfigurationProperties` Bean的支持，使其能够被自动注入。

**22. @AutoConfigureAfter**
用于自动配置类上，声明当前配置类需要在另一个指定的自动配置类完成配置之后再执行。
例如，Mybatis的自动配置需要在数据源配置完成之后：

```java

@AutoConfigureAfter(DataSourceAutoConfiguration.class)
public class MybatisAutoConfiguration {
    // ...
}
```

**23. @AutoConfigureBefore**
与`@AutoConfigureAfter`作用相反，声明当前配置类需要在另一个指定的自动配置类之前执行。

**24. @Import**
Spring 3.0引入的注解，用于导入一个或多个被`@Configuration`标注的配置类。在Spring Boot中广泛用于模块化配置。

**25. @ImportResource**
同样是Spring 3.0引入的注解，用于导入一个或多个传统的Spring XML配置文件。这对于Spring Boot项目需要集成或迁移遗留的、难以用Java
Config重构的配置非常有用。