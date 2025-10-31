---
title: Spring Boot 支持 HTTPS 如此简单，So easy
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Boot
---

这里讲的是 Spring Boot 内嵌式 Server 打 jar 包运行的方式，打 WAR 包部署的就不存在要 Spring Boot 支持 HTTPS 了，需要去外部对应的 Server 配置。



## 支持 HTTPS

Spring Boot 配置 SSL 很简单，只需要通过一系列的 `server.ssl.*` 参数即可完成配置，如下所示。

application.properties 配置文件参考配置：

```
server.port=8443
server.ssl.protocol=TLS
server.ssl.key-store=classpath:javastack.keystore
server.ssl.key-store-password=javastack
server.ssl.key-store-type=JKS
```

这边只是提供了一个 SSL 单向验证的演示，更多 SSL 参数配置如下。

```
server.ssl.ciphers= # Supported SSL ciphers.
server.ssl.client-auth= # Whether client authentication is wanted ("want") or needed ("need"). Requires a trust store.
server.ssl.enabled= # Enable SSL support.
server.ssl.enabled-protocols= # Enabled SSL protocols.
server.ssl.key-alias= # Alias that identifies the key in the key store.
server.ssl.key-password= # Password used to access the key in the key store.
server.ssl.key-store= # Path to the key store that holds the SSL certificate (typically a jks file).
server.ssl.key-store-password= # Password used to access the key store.
server.ssl.key-store-provider= # Provider for the key store.
server.ssl.key-store-type= # Type of the key store.
server.ssl.protocol=TLS # SSL protocol to use.
server.ssl.trust-store= # Trust store that holds SSL certificates.
server.ssl.trust-store-password= # Password used to access the trust store.
server.ssl.trust-store-provider= # Provider for the trust store.
server.ssl.trust-store-type= # Type of the trust store.
```

> 参数对应的类：org.springframework.boot.web.server.Ssl

上面的例子配置后就能开启 HTTPS 了，默认的 HTTP 协议就不再支持了，Spring Boot 不支持以配置文件配置的方式同时支持 HTTP 和 HTTPS。

## 如何同时支持?

如果你需要同时支持 HTTP 和 HTTPS 这两个协议，就需要把另外一个协议用程序化的方式来配置。因为通过程序的方式配置 HTTP 协议更加简单一点，所以，Spring Boot 推荐的做法是把 HTTPS 配置在配置文件，HTTP 通过程序来配置。

来，下面示例就是通过程序的方式来额外支持 HTTP 协议。

```
@SpringBootApplication
public class JavastackApplication {

	@Bean
	public ServletWebServerFactory servletContainer() {
		TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
		tomcat.addAdditionalTomcatConnectors(createStandardConnector());
		return tomcat;
	}

	private Connector createStandardConnector() {
		Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
		connector.setPort(8080);
		return connector;
	}

	public static void main(String[] args) {
		SpringApplication.run(JavastackApplication.class, args);
	}

}
```

启动 Spring Boot 之后就会看到下面的同时支持两个协议日志。

```
Tomcat started on port(s): 8443 (https) 8080 (http) with context path '/'
```

Spring Boot 支持 HTTPS 如此简单，开发现在把运维的事都做了……

在公众号后台回复：boot，还能获取栈长整理的往期 Spring Boot 教程，都是实战干货，以下仅为部分预览。

- Spring Boot 读取配置的几种方式
- Spring Boot 如何做参数校验？
- Spring Boot 最核心的 25 个注解！
- Spring Boot 2.x 启动全过程源码分析
- Spring Boot 2.x 新特性总结及迁移指南
