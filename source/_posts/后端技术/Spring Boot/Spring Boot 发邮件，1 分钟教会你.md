---
title: Spring Boot 发邮件，1 分钟教会你
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Boot
---

Spring Boot 提供了一个发送邮件的简单抽象，使用的是下面这个接口。

> org.springframework.mail.javamail.JavaMailSender

Spring Boot 提供了一个 `starter`，并能自动配置，下面来做个小例子，顺便解析它做了什么工作。


### 1、添加依赖

在 Maven `pom.xml` 配置文件中加入 `spring-boot-starter-mail` 依赖。

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

### 2、添加配置参数

然后在 `application.properties` 文件中加入以下配置。

```
spring.mail.host=smtp.exmail.qq.com
spring.mail.username=admin@javastack.cn
spring.mail.password=123456

# 启动SSL时的配置
spring.mail.smtp.socketFactory.class=javax.net.ssl.SSLSocketFactory
spring.mail.smtp.socketFactory.fallback=false
spring.mail.smtp.socketFactory.port=465
```

### 3、一个简单的发送邮件例子

写一个控制器，写一个简单的发送邮件的小例子，发送成功返回 `true`，发送失败返回 `false`。

```
@Autowired
private JavaMailSender javaMailSender;

@RequestMapping("/sendEmail")
@ResponseBody
public boolean sendEmail() {
	SimpleMailMessage msg = new SimpleMailMessage();
	msg.setFrom("admin@javastack.cn");
	msg.setBcc();
	msg.setTo("admin@javastack.cn");
	msg.setSubject("Java");
	msg.setText("技术分享");
	try {
		javaMailSender.send(msg);
	} catch (MailException ex) {
		System.err.println(ex.getMessage());
		return false;
	}
	return true;
}
```

### 4、自动配置都做了什么？

Spring Boot 发现类路径下有这个 `spring-boot-starter-mail` 包和 `spring.mail.host` 参数就会自动配置 `JavaMailSenderImpl`。

上面那些 `spring.mail.xx` 参数用来装配 `MailProperties` 这个类。

> org.springframework.boot.autoconfigure.mail.MailProperties

自动配置类：

> org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration

![](img/18-8-27-70236436.jpg)

> org.springframework.boot.autoconfigure.mail.MailSenderPropertiesConfiguration

![](img/18-8-27-56485452.jpg)

其实就是用了上面装配的参数注册了一个 `JavaMailSenderImpl` 实例而已，然后你就可以注入使用了。
