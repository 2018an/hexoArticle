---
title: Spring Boot 与 Thymeleaf 视图层整合实战指南
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Boot
---

## Thymeleaf模板引擎简介

Thymeleaf是一个现代化的服务端Java模板引擎，专门用于处理XML、XHTML和HTML5文档。与传统的Velocity、FreeMarker等模板引擎类似，它可以无缝集成到Spring
MVC等Web框架中，作为视图层渲染工具。

在Spring Boot生态中，Thymeleaf已成为首选的模板解决方案。值得注意的是，新版本的Spring Boot已不再对Velocity提供官方支持。

> 官方网站：http://www.thymeleaf.org/

## 项目依赖配置

要在Spring Boot项目中启用Thymeleaf，只需在构建配置中添加对应的启动器依赖即可。

Maven项目配置示例：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

该启动器已自动包含Web模块依赖，无需单独引入`spring-boot-starter-web`。

## 自动配置详解

Spring Boot为Thymeleaf提供了一套完整的自动配置方案，主要涉及以下两个核心类：

自动配置主类：
> org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration

配置属性类：
> org.springframework.boot.autoconfigure.thymeleaf.ThymeleafProperties

通过分析源码，我们可以了解默认配置参数：

```
private static final Charset DEFAULT_ENCODING = Charset.forName("UTF-8");
private static final MimeType DEFAULT_CONTENT_TYPE = MimeType.valueOf("text/html");
public static final String DEFAULT_PREFIX = "classpath:/templates/";
public static final String DEFAULT_SUFFIX = ".html";
```

- 默认字符编码：UTF-8
- 默认内容类型：text/html
- 默认模板查找路径：classpath:/templates/
- 默认模板文件扩展名：.html

所有配置参数均可通过`application.properties`或`application.yml`文件中的`spring.thymeleaf.*`前缀进行自定义调整。

## 快速上手实践

基于上述配置原理，我们可以按照以下步骤快速创建Thymeleaf模板：

**第一步：创建模板目录**
在项目的`resources`资源目录下新建`templates`文件夹。

**第二步：添加模板文件**
在`templates`目录中创建HTML模板文件，例如`index.html`。

**第三步：编写模板内容**

1. 在HTML文档开头声明Thymeleaf命名空间：

```html

<html xmlns:th="http://www.thymeleaf.org">
```

2. 使用`th:`前缀的属性进行动态内容绑定：

```html

<div th:text="${message}">默认文本内容</div>
```

3. 引用静态资源文件（CSS、JS等）：

```html

<link rel="stylesheet" th:href="@{/css/style.css}"/>
<script th:src="@{/js/app.js}"></script>
```

4. 访问控制器传递的数据：

- 模型属性：`${attributeName}`
- 会话属性：`${session.attributeName}`
- 请求参数：`${param.parameterName}`

更多数据访问表达式和高级用法，可参考[官方文档](http://www.thymeleaf.org/doc/articles/springmvcaccessdata.html)获得详细说明。