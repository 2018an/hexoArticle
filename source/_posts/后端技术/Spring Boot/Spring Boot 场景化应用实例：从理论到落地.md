---
title: Spring Boot 场景化应用实例：从理论到落地
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Boot
---

## 启用Spring Boot的Web模块

在项目已引入Spring Boot基础依赖的基础上，如果需要开发Web应用程序并利用Spring MVC框架，只需引入对应的启动器依赖即可。Spring
Boot的自动配置机制会处理所有Web功能相关的组件初始化与装配工作。

在Maven项目中，添加以下依赖配置：

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

## 创建应用启动入口

接下来需要定义应用的启动类。通常建议将该类放置在项目的根包路径下，以简化组件扫描范围。示例中为了演示简便，将控制器与启动类合并，但在实际开发中应按照业务模块进行合理的包划分。

启动类必须包含标准的 `main` 方法，并在其中调用Spring Boot的启动入口。

```java
package hello;

import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.stereotype.*;
import org.springframework.web.bind.annotation.*;

@Controller
@SpringBootApplication
public class SampleController {

    @RequestMapping("/")
    @ResponseBody
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(SampleController.class, args);
    }
}
```

## 配置项目打包插件

为了让Maven能够正确打包Spring Boot应用，需要在构建配置中添加官方提供的打包插件。

```
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
			<executions>
				<execution>
					<goals>
						<goal>repackage</goal>
					</goals>
					<configuration>
						<classifier>exec</classifier>
						<fork>true</fork>
						<excludeDevtools>true</excludeDevtools>
					</configuration>
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>
```

## 启动应用的几种途径

Spring Boot项目可以通过以下任意一种方式启动运行：

1. 在IDE中直接执行启动类中的 `main` 方法。
2. 在命令行中使用Maven命令：`mvn spring-boot:run`。
3. 将项目打包为可执行JAR文件后，通过 `java -jar 文件名.jar` 运行。

应用默认使用8080端口接收请求。若要修改端口，可在配置文件中设置 `server.port` 属性，或在启动时通过命令行参数指定。

启动成功后，访问 http://localhost:8080/ 即可看到页面输出：Hello World!

## 快速创建Spring Boot项目

除了手动配置Maven项目外，Spring
Boot官方提供了一个在线项目生成工具。访问 http://start.spring.io/，在界面上选择所需的依赖启动器，即可一键生成项目基础结构，直接导入到开发环境中使用。这种方式尤其适合新手快速搭建项目骨架。