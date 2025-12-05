---
title: Spring Boot 实现 XML 数据响应与格式处理指南
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Boot
---

## Spring Boot中实现XML数据响应输出

在基于Spring Boot构建的Web应用中，除了常见的JSON格式外，有时也需要支持XML数据格式的响应。本文将在已有Spring
Boot项目基础上，演示如何快速实现XML数据的返回。

## 实现XML数据返回的关键步骤

### 添加XML处理依赖

在项目配置文件中引入Jackson的XML数据格式支持模块：

```xml

<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

此依赖无需指定版本号，因为Spring Web MVC中已经预定义了其版本信息。查看`spring-webmvc`的依赖管理可见：

```xml

<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.9.5</version>
    <scope>compile</scope>
    <optional>true</optional>
    <!-- 排除特定依赖 -->
    <exclusions>
        <exclusion>
            <artifactId>jcl-over-slf4j</artifactId>
            <groupId>org.slf4j</groupId>
        </exclusion>
    </exclusions>
</dependency>
```

注意到`<optional>true</optional>`标签，这意味着该依赖不会自动传递到项目中，需要显式声明引入。

### 配置XML响应格式

#### 1. 声明响应方式

在控制器类或方法上使用`@RestController`或`@ResponseBody`注解，表明响应内容将直接写入HTTP响应体中。

#### 2. 指定响应类型

默认情况下，响应的`Content-Type`头可能为`application/xhtml+xml;charset=UTF-8`。为确保明确返回XML格式，可以在请求映射中指定媒体类型：

```java
@RequestMapping(value = "/api/data",
        produces = MediaType.APPLICATION_XML_VALUE)
```

这会强制设置响应内容类型为`application/xml;charset=UTF-8`。

#### 3. 定制XML输出结构

通过Jackson提供的XML注解，可以精细控制生成的XML文档结构：

```java

@JacksonXmlRootElement(localName = "api_response")
public class UserProfileVO {

    @JacksonXmlProperty(localName = "user_name")
    private String userName;

    @JacksonXmlElementWrapper(useWrapping = false)
    @JacksonXmlProperty(localName = "order_record")
    private List<OrderVO> orderList;

    // 构造函数、getter和setter方法
}
```

**核心注解说明**：

- `@JacksonXmlRootElement`：标注于类上，定义XML文档的根元素名称
- `@JacksonXmlProperty`：标注于属性上，定义XML元素或属性的名称
- `@JacksonXmlElementWrapper`：控制集合类型的包装方式，可禁用默认的包装层

完整的XML处理注解集可在相应包中查看：
![](http://qianniu.javastack.cn/18-8-16/89032800.jpg)

### 手动对象与XML转换

`jackson-dataformat-xml`模块提供了`XmlMapper`类，继承自通用的`ObjectMapper`，专门处理XML序列化与反序列化：

```java
import com.fasterxml.jackson.dataformat.xml.XmlMapper;

// 创建XML映射器实例
XmlMapper xmlMapper = new XmlMapper();

        // 对象转为XML字符串
        String xmlOutput = xmlMapper.writeValueAsString(userProfile);

        // XML字符串转为对象
        UserProfileVO userProfile = xmlMapper.readValue(xmlInput, UserProfileVO.class);

        // 格式化输出（美化格式）
        String formattedXml = xmlMapper.writerWithDefaultPrettyPrinter()
                .writeValueAsString(userProfile);
```

**常用转换方法**：

- `XmlMapper.readValue()`：将XML源数据反序列化为Java对象
- `XmlMapper.writeValue()`：将Java对象序列化为XML数据流
- `ObjectMapper.writeValueAsString()`：将对象序列化为XML字符串

## 实际应用示例

完整控制器实现参考：

```java

@RestController
@RequestMapping("/api")
public class DataController {

    @GetMapping(value = "/user/{id}",
            produces = MediaType.APPLICATION_XML_VALUE)
    public UserProfileVO getUserProfile(@PathVariable Long id) {
        UserProfileVO profile = new UserProfileVO();
        profile.setUserName("SpringDeveloper");
        // 设置订单列表等数据
        return profile; // 自动转换为XML格式
    }

    @PostMapping(value = "/user",
            consumes = MediaType.APPLICATION_XML_VALUE,
            produces = MediaType.APPLICATION_XML_VALUE)
    public UserProfileVO createUser(@RequestBody UserProfileVO profile) {
        // 处理XML请求体，返回XML响应
        return profileService.save(profile);
    }
}
```

通过以上配置，Spring Boot应用即可同时支持JSON和XML两种数据格式的请求与响应，满足不同客户端或集成场景的需求。