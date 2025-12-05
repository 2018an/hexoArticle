---
title: 在 Spring Boot 项目中整合 MongoDB
date: 2025-10-30 14:42:34
category: 数据库
tags: MongoDB
---

引入 Spring Data MongoDB 依赖
在 Spring Boot 项目中，只需添加以下 starter 依赖即可快速集成 MongoDB：

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```
配置 MongoDB 连接信息
Spring Boot 为 MongoDB 提供了丰富的自动配置选项，所有相关配置如下：

properties

# MONGODB 配置 (MongoProperties)

spring.data.mongodb.authentication-database= # 认证数据库名称
spring.data.mongodb.database=test # 默认数据库名
spring.data.mongodb.field-naming-strategy= # 字段命名策略类的全限定名
spring.data.mongodb.grid-fs-database= # GridFS 存储的数据库名
spring.data.mongodb.host=localhost # MongoDB 服务器地址。不可与 uri 同时设置
spring.data.mongodb.password= # 登录密码。不可与 uri 同时设置
spring.data.mongodb.port=27017 # 服务器端口。不可与 uri 同时设置
spring.data.mongodb.reactive-repositories.enabled=true # 启用响应式仓库支持
spring.data.mongodb.repositories.enabled=true # 启用常规仓库支持
spring.data.mongodb.uri=mongodb://localhost/test # 连接 URI。不可与 host/port/credentials 同时设置
spring.data.mongodb.username= # 登录用户名。不可与 uri 同时设置
一个典型的 application.yml 配置示例如下，连接本地 dev 数据库：

yaml
spring:
data:
mongodb:
host: 127.0.0.1
port: 27017
database: dev
在代码中使用 MongoDB
方式一：继承 MongoRepository（推荐用于简单 CRUD）
定义一个 Repository 接口，继承 MongoRepository 即可获得基础的增删改查方法：

java
import org.springframework.data.mongodb.repository.MongoRepository;

public interface IpCodeRepository extends MongoRepository<IpCode, String> {
// 可以在此定义自定义查询方法
}
方式二：注入 MongoTemplate（用于复杂操作）
对于更复杂的查询或聚合操作，可以直接注入 MongoTemplate 来使用：

java
@Autowired
private MongoTemplate mongoTemplate;
总结：第一种方式简洁快速，适合标准操作；第二种方式灵活强大，适合处理复杂的查询逻辑和聚合管道。