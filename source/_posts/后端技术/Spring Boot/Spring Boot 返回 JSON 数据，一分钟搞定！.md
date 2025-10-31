---
title: Spring Boot 返回 JSON 数据，一分钟搞定！
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Boot
---

在 WEB 项目中返回 JSON 数据是常见的交互形式，在 Spring Boot 中这一切都变得十分简单。So easy!!!


## 如何返回 JSON 数据？

在 Spring Boot 中返回 JSON 数据很简单，如下几步。

### 加入依赖

```
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.0.4.RELEASE</version>
</parent>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

除了 Spring Boot 必须自带的 parent 依赖外，仅仅只需要加入这个 `spring-boot-starter-web` 包即可，它会自动包含所有 JSON 处理的包，如下图所示。

![](http://qianniu.javastack.cn/18-8-16/4664411.jpg)

这个插件感谢知识星球球友的分享，简单不错，点击文章底部的阅读原文，可以加入一起学习。


### 返回 JSON 数据格式定义

##### 1）定义返回方式

在 Controller 类上面用 `@RestController` 定义或者在方法上面用 `@ResponseBody` 定义，表明是在 Body 区域输出数据。

下面是使用示例：

```
@RestController
public class JsonTest {

	@GetMapping(value = "/user/{userId}")
	public User getUserInfo(@PathVariable("userId") String userId) {
		User user = new User("Java", 18);
		user.setId(Long.valueOf(userId));
		return user;
	}

}
```

##### 2）自定义输出格式

上面的方法直接返回对象，对象会自动转换为 JSON 格式，不过是默认的标签，可以通过以下标签进行自定义 JSON 格式。


```
public class User {

	@JsonProperty("user-name")
	private String userName;

	private Long id;

	private Integer age;

	@JsonIgnore
	private String address;

	@JsonInclude(JsonInclude.Include.NON_NULL)
	private String memo;
	
	// get set 略
	
}
```

程序输出：

```
{"id":1,"age":18,"user-name":"Java"}
```

上面演示了几个常用的注解。

**@JsonProperty：** 可用来自定义属性标签名称；

**@JsonIgnore：** 可用来忽略不想输出某个属性的标签；

**@JsonInclude：** 可用来动态包含属性的标签，如可以不包含为 null 值的属性；

更多注解可以查看这个包：

![](http://qianniu.javastack.cn/18-8-16/13417927.jpg)

### 如何手动完成对象 和 Json 的互转？

`jackson-databind` 包里面有一个 `com.fasterxml.jackson.databind.ObjectMapper` 类可以完成对象和 Json 数据的互转，下面是一个简单的合作示例。

```
ObjectMapper objectMapper = new ObjectMapper();

String userJsonStr = objectMapper.writeValueAsString(user);

User jsonUser = objectMapper.readValue(userJsonStr, User.class);
```

更多相关的使用及原理可以查看这个包。

好了，Spring Boot 返回 JSON 格式数据就是这么简单，有什么不懂的可以点击阅读原文加入星球和大家一起学习讨论。

《Spring Boot 返回 XML 数据》栈长正在拼命写作中，过两天分享。
