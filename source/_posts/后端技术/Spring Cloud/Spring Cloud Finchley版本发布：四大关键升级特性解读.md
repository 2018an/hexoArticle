---
title: Spring Cloud Finchley版本发布：四大关键升级特性解读
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Cloud
---

![](img/18-6-20-82197726.jpg)

根据Spring官方博客的最新公告，Spring Cloud Finchley版本已于六月十九日正式发布。在Maven中央仓库中，我们也看到了相应组件的版本更新。

![](img/18-6-20-96626148.jpg)

Finchley正式版的推出似乎经历了较长的开发周期，可谓“厚积薄发”。此次重大更新主要带来了以下四项核心改进。

### 核心更新内容

#### 1. 引入全新的Spring Cloud Gateway模块

Spring Cloud Gateway是采用响应式编程模型的API网关解决方案，基于Spring WebFlux框架与Netty响应式网络库构建，旨在替代原有的Spring
Cloud Netflix Zuul组件。它提供了简洁灵活的动态路由配置能力，并为每个路由节点支持丰富的过滤器链，涵盖URL重写、熔断降级、请求头处理、流量控制及安全认证等多种治理功能。

#### 2. 新增Spring Cloud Function支持

**Spring Cloud Function的核心价值体现在：**

- 倡导以函数式单元构建业务逻辑，提升代码的模块化与复用性；
- 将业务实现与具体运行环境解耦，同一段函数代码可部署为Web服务端点、流式数据处理任务或定时任务等不同形态；
- 提供跨Serverless平台的一致性编程模型，同时支持本地环境或PaaS平台独立运行；
- 在Serverless环境中保留Spring Boot的核心特性，如自动装配、依赖注入、监控指标收集等。

#### 3. 全面适配Spring Boot 2.0.x系列

Finchley版本完全基于Spring Boot 2.0.x技术栈构建，官方明确建议不再将其与Spring Boot 1.5.x及更早版本结合使用，以确保技术栈的兼容性与稳定性。

#### 4. 最低运行时要求提升至JDK 8

正式将Java 8设定为最低版本要求，这符合当前主流Java开发环境的实际情况。

更多详细更新说明与技术细节，请参阅Spring官方博客的完整发布公告：
> https://spring.io/blog/2018/06/19/spring-cloud-finchley-release-is-available

### 历史版本维护周期调整

随着新版本的正式发布，部分历史版本将逐步进入维护终结阶段。Spring官方同步更新了各版本的技术支持时间线：

- **Camden版本**：即日起进入生命周期结束阶段
- **Dalston版本**：技术维护将持续至2018年12月
- **Edgware版本**：其生命周期将与Spring Boot 1.5.x系列同步结束

### 总结

如果对Spring Cloud的版本演进路线尚不清晰，可通过公众号的Spring技术专题获取更系统的学习资料。

**技术社区讨论**：您当前的项目在使用哪个版本的Spring Cloud？对于升级到Finchley版本有何考量？欢迎在评论区分享您的见解与实践经验。