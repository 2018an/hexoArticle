---
title: Spring Boot 主类与框架脉络解析
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Boot
---

在基于 Spring Boot 的项目中，配置管理是一个关键环节。项目通常会区分开发、测试与生产等多套环境，每套环境都有其独立的配置需求。Spring
Boot 让开发者能够借助同一套代码，在不同环境中灵活切换配置，大幅简化了部署与维护工作。

**Spring Boot 支持多种配置来源，以下将基于 Spring Boot 2.x 版本展开说明。**

常见的配置加载方式包括：

1. Properties 配置文件；
2. YAML 配置文件；
3. 操作系统环境变量；
4. 命令行传入参数；
5. 其他外部化配置源。

**在应用程序中，可以通过以下几种方式读取这些配置值：**

- 通过 `@Value` 注解直接将属性值注入到字段，它依赖于 Spring 的 `Environment` 抽象；
- 使用 `@ConfigurationProperties` 注解将一组配置属性批量绑定到 Java Bean 对象；
- 直接注入 `Environment` 对象，然后调用其 `getProperty` 方法获取指定属性。

Spring Boot 采用一种可扩展的 `PropertySource` 机制来管理配置源，并允许在优先级高的源中覆盖低优先级的属性值。

**以下是配置源的加载优先级顺序（由高到低）：**

```
1、Devtools 全局配置参数（仅在开发工具激活时生效）；
2、测试类中 @TestPropertySource 注解指定的属性；
3、测试类中 @SpringBootTest 注解的 properties 属性；
4、命令行参数，例如：java -jar app.jar --server.port=8081；
5、通过 SPRING_APPLICATION_JSON 传递的 JSON 格式参数，如：java -Dspring.application.json='{"server":{"port":9090}}' -jar app.jar；
6、ServletConfig 初始化参数；
7、ServletContext 初始化参数；
8、通过 JNDI 获取的属性（例如 java:comp/env/...）；
9、Java 系统属性（System.getProperties()）；
10、操作系统环境变量；
11、随机数属性源，仅用于 random.* 属性的生成；
12、位于 JAR 包外、带环境标识的配置文件（如 application-dev.yml）；
13、位于 JAR 包内、带环境标识的配置文件；
14、位于 JAR 包外的通用配置文件（application.yml 或 application.properties）；
15、位于 JAR 包内的通用配置文件；
16、通过 @Configuration 类上的 @PropertySource 注解导入的属性文件；
17、通过 SpringApplication.setDefaultProperties() 设置的默认属性。
```

**优先级数字越小，其配置生效的权重越高，高优先级的配置值会覆盖低优先级的配置。**

下面通过一个简单示例，验证上述配置加载的优先级。

1. 在应用启动时设置一个 Java 系统属性。

```java

@Bean
public CommandLineRunner commandLineRunner() {
    return args -> {
        System.setProperty("name", "system-property-value");
    };
}
```

2. 在 application.properties 中定义同名属性。

```
name = value-from-application-properties
```

3. 在 application-dev.properties 中也定义该属性。

```
name = value-from-application-dev-properties
```

4. 编写测试类，指定测试参数并激活 dev 环境。

```java

@RunWith(SpringRunner.class)
@SpringBootTest(properties = {"name=value-from-test", "gender=male"})
@ActiveProfiles("dev")
public class ConfigurationPrecedenceTest {

    @Value("${name}")
    private String appName;

    @Test
    public void printConfiguredName() {
        System.out.println("最终生效的 name 值为：" + appName);
    }
}
```

执行上述测试方法，控制台将输出：

```
最终生效的 name 值为：value-from-test
```

测试结果表明，来自 `@SpringBootTest` 注解的配置参数拥有较高优先级，覆盖了其他来源的属性值。掌握配置加载的优先级顺序后，当出现配置未按预期生效的情况时，可以有条理地排查各配置源的覆盖情况。