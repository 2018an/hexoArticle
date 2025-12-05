---
title: 在 Spring Boot 项目中快速集成 Swagger 生成 API 文档
date: 2025-10-30 14:42:34
category: 工具
tags: Swagger
---

以下步骤演示如何在 Spring Boot 应用中集成 Swagger，实现 API 文档的自动生成与在线查看。

集成步骤

1. 引入依赖
   在 pom.xml 中添加以下 Maven 依赖：

xml
<!-- Swagger 核心 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
</dependency>
<!-- Swagger 界面 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
</dependency>
2. 配置参数
在 application.yml 或 application.properties 中添加 Swagger 相关配置：

yaml
swagger:
title: 用户服务 API
description: 提供用户注册、登录、信息管理等接口
version: v1.0
terms-of-service-url: https://www.example.com/terms
base-package: com.example.demo.controller
contact:
name: 技术支持
url: https://www.example.com
email: support@example.com

3. 编写配置类
   创建一个配置类，启用 Swagger 并加载上述配置：

java
@Configuration
@EnableSwagger2
@ConfigurationProperties(prefix = "swagger")
@Data
public class SwaggerConfig {

    private String basePackage;
    private String title;
    private String description;
    private String termsOfServiceUrl;
    private String version;
    private Contact contact;

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage(basePackage))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title(title)
                .description(description)
                .termsOfServiceUrl(termsOfServiceUrl)
                .version(version)
                .contact(contact)
                .build();
    }

}
使用与注解说明
集成后，Swagger 会自动扫描指定包下的控制器类，生成对应接口列表。为进一步提升文档可读性，可使用以下常用注解对接口进行详细描述：

注解 用途说明
@Api 标注在控制器类上，说明该类功能
@ApiOperation 描述具体接口方法
@ApiParam 说明方法参数的名称与含义
@ApiImplicitParam / @ApiImplicitParams 描述接口的请求参数（非绑定到方法参数时使用）
@ApiModel 说明数据模型类（如请求体或返回对象）
@ApiModelProperty 描述模型类中的字段
@ApiResponse / @ApiResponses 声明接口的返回状态与说明
示例代码：

java
@Api(tags = "用户认证模块")
@RestController
@RequestMapping("/auth")
public class AuthController {

    @ApiOperation("用户登录接口")
    @ApiImplicitParams({
        @ApiImplicitParam(name = "username", value = "登录账号", required = true, dataType = "string", paramType = "query"),
        @ApiImplicitParam(name = "password", value = "登录密码", required = true, dataType = "string", paramType = "query")
    })
    @PostMapping("/login")
    public ResponseResult login(String username, String password) {
        // 登录逻辑
    }

}
完成以上配置后，启动项目，访问以下地址即可查看与测试接口：

http://localhost:8080/swagger-ui.html

相关阅读推荐
国外程序员常用的十大学习与资源网站

免费在线工具：流程图与思维导图绘制

提升编码效率的实用工具推荐

深入了解 Git：从入门到实战

RESTful API 设计原则与实践