---
title: Spring Boot 配置读取的多重方式与最佳实践
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Boot
---

## 应用配置数据加载方法详解

在Spring Boot应用中，我们经常需要从外部配置源读取参数。本文将详细介绍几种常用的配置加载策略。

### 从主配置文件读取参数

在`application.yml`或`application.properties`中添加以下配置项：

```yaml
# application.yml格式
info:
  address: USA
  company: Spring
  degree: high
```

或

```properties
# application.properties格式
info.address=USA
info.company=Spring
info.degree=high
```

#### 方案一：使用@Value注解注入

通过`@Value`注解可以直接将配置文件中的值注入到类的字段中：

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class CompanyInfoReader {

    @Value("${info.address}")
    private String location;

    @Value("${info.company}")
    private String organization;

    @Value("${info.degree}")
    private String level;

    // 相应的getter和setter方法
    public String getLocation() {
        return location;
    }

    public void setLocation(String location) {
        this.location = location;
    }

    // 其他getter/setter省略...
}
```

#### 方案二：使用@ConfigurationProperties批量绑定

通过`@ConfigurationProperties`注解可以将配置文件中具有相同前缀的参数批量绑定到对象的属性上：

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "info")
public class CompanyInfoBinder {

    private String address;
    private String company;
    private String degree;

    // 相应的getter和setter方法
    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    // 其他getter/setter省略...
}
```

### 加载自定义配置文件

对于非主配置文件，我们可以创建独立的属性文件。例如，在资源目录下创建`config/db-config.properties`：

```properties
db.username=root
db.password=123456
```

#### 方案一：@PropertySource结合@Value

使用`@PropertySource`注解指定要加载的配置文件，然后通过`@Value`注入具体值：

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

@Component
@PropertySource("classpath:config/db-config.properties")
public class DatabaseConfigReader {

    @Value("${db.username}")
    private String userName;

    @Value("${db.password}")
    private String passKey;

    // getter和setter方法
    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getPassKey() {
        return passKey;
    }

    public void setPassKey(String passKey) {
        this.passKey = passKey;
    }
}
```

**注意**：`@PropertySource`注解目前不支持直接加载YAML格式的文件。

#### 方案二：@PropertySource结合@ConfigurationProperties

同样先指定配置文件，然后使用前缀绑定方式批量加载配置：

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "db")
@PropertySource("classpath:config/db-config.properties")
public class DatabaseConfigBinder {

    private String username;
    private String password;

    // getter和setter方法
    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

#### 方案三：通过Environment接口动态获取

所有加载到Spring环境中的配置属性，都可以通过注入`Environment`对象来动态获取：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.env.Environment;
import org.springframework.stereotype.Component;

@Component
public class ConfigAccessor {

    @Autowired
    private Environment env;

    // 根据键名获取配置值
    public String getConfigValue(String key) {
        return env.getProperty(key);
    }

    // 获取数据库用户名示例
    public String getDbUsername() {
        return env.getProperty("db.username");
    }
}
```

### 配置加载方案对比总结

Spring Boot提供了多种灵活的配置读取方式：

1. **@Value注解**：适合单个配置项的注入，简单直接
2. **@ConfigurationProperties**：适合批量绑定具有相同前缀的配置项，支持类型安全
3. **@PropertySource**：用于加载非主配置文件，但仅支持properties格式
4. **Environment接口**：提供动态获取配置的能力，适合运行时不确定配置键的场景

开发者可以根据具体需求选择最合适的配置加载策略。对于简单的少量配置，使用`@Value`即可；对于复杂的多属性配置，推荐使用
`@ConfigurationProperties`进行类型安全的批量绑定。