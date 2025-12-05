---
title: Spring Boot 项目打包形态转换：灵活切换与服务器部署适配
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Boot
---

![](http://qianniu.javastack.cn/18-2-27/88945925.jpg)

Spring Boot 应用默认采用可执行 JAR 包的形式进行部署，这也是官方推荐的方式。这种封装形式简单独立，便于分发和运行。然而，在需要频繁发布补丁或增量更新的场景下，庞大的
JAR 文件在上传和替换时可能显得不够灵活。此时，传统的 WAR 包部署方式可能更具优势，它允许你仅更新变动的部分，从而简化补丁发布流程。因此，根据项目实际运维需求，将应用转换为
WAR 包部署是一个可行的选择。

#### 实现 WAR 包部署的配置步骤

以下将以 Maven 项目为例，说明关键的配置调整。如果你使用 Gradle，请参考类似的配置项进行修改。

**1、调整应用启动类**
需要让主启动类继承 `SpringBootServletInitializer` 基类，并重写其 `configure` 方法，以确保在外部 Servlet 容器（如
Tomcat）中也能正常启动 Spring Boot 上下文。

参考以下调整后的启动类代码：

```java

@SpringBootApplication
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        // 指定应用启动的配置源
        return builder.sources(Application.class);
    }

    public static void main(String[] args) {
        // 保留此main方法，仍支持以独立JAR形式运行
        SpringApplication.run(Application.class, args);
    }
}
```

**2、修改项目打包格式**
在项目的 `pom.xml` 文件中，显式地将打包类型指定为 `war`。

```
<packaging>war</packaging>
```

若不指定，Maven 默认会生成 JAR 包。

**3、排除内嵌的 Tomcat 容器**
为了避免与外部部署的 Tomcat 服务器产生库冲突，需要将 Spring Boot 内置的 Tomcat 依赖作用域标记为 `provided`
。这表示该依赖在编译和测试时需要，但不会打包到最终的 WAR 文件中。

```
<dependencies>
    <!-- 其他依赖 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <scope>provided</scope>
    </dependency>
    <!-- 其他依赖 -->
</dependencies>
```

**4、确保 WAR 插件配置正确**
如果你是通过继承 `spring-boot-starter-parent` 来使用 Spring Boot，则相关配置已预设完毕，通常无需额外添加。若你采用的是
`spring-boot-dependencies` 依赖管理方式，则需在 `pom.xml` 中显式配置 `maven-war-plugin` 插件：

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <configuration>
        <!-- 跳过web.xml检查，避免构建失败 -->
        <failOnMissingWebXml>false</failOnMissingWebXml>
    </configuration>
</plugin>
```

#### 打包操作

生成 WAR 包的命令与打包 JAR 包完全相同，执行 Maven 打包指令即可：

```
mvn clean package
```

打包完成后，可在 `target` 目录下找到生成的 `.war` 文件。

在 IntelliJ IDEA 等集成开发环境中，也可以通过可视化构建选项完成打包，如下图所示：
![](http://qianniu.javastack.cn/18-2-8/28070341.jpg)

一个明显的体验是，构建 WAR 包的过程通常比构建可执行 JAR 包要耗时更长。

#### 转换部署方式后的注意事项

1. **容器配置的转移**：在 `application.properties` 或 `application.yml` 中设置的 `server.*`
   等服务器相关属性（如端口、上下文路径）将不再生效，这些配置需转移至外部 Tomcat 容器的配置文件中进行管理。
2. **版本兼容性**：当升级 Spring Boot 版本时，需关注其与外部 Tomcat 版本的兼容性，这可能需要进行验证和同步升级。
3. **构建效率**：如前所述，打包为 WAR 格式的构建速度相对较慢，在持续集成流程中需考虑此时间成本。

目前来看，上述是主要的技术考量点。你是否在实践中遇到过其他问题或有不同的见解？欢迎在评论区分享你的经验。