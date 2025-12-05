---
title: 一个功能全面的轻量级 Java 工具库
date: 2025-10-30 14:42:34
category: 工具
tags: 工具类
---

https://img/18-2-27-74126275.jpg

认识 Jodd
Jodd 是一个面向 Java 开发者的综合性迷你框架与工具集合，其完整包体积不足 1.7MB，却集成了众多实用模块。它的设计初衷是简化常见开发任务，让编码更高效、更有趣，堪称
Java 领域的“多功能工具箱”。

如果你在项目中考虑自行实现某些通用功能，不妨先查看 Jodd 是否已提供成熟方案。

核心功能概览
Jodd 能够处理以下典型场景：

Java Bean 的操作与注入

从多种数据源加载和装配 Bean

简化 JDBC 操作与 SQL 处理

日期时间的便捷处理

字符串的格式化与增强操作

本地文件的搜索与管理

Servlet 请求的辅助处理

甚至包含一个基于 JSP 的轻量级 MVC 框架

模块化组成
Jodd 由多个独立模块构成，开发者可按需引入。

核心工具集 (Jodd Core)

包含一系列高性能基础工具：

TypeConverter：强大的类型转换器

BeanUtil：支持嵌套属性与集合操作的 Bean 工具

JDateTime：增强型日期时间类

各类高性能的 IO 工具（Buffer、Writer 等）

Wildcard：通配符匹配工具

ServletUtil：Servlet 与 JSP 扩展工具

FindFile / ClassFinder：支持通配符与正则的文件/类搜索器

Cache：简洁的缓存实现（LRU、FIFO）

StringUtil：功能丰富的字符串处理工具

轻量 MVC 框架 (Madvoc)

一个注重约定优于配置的快速 Web 开发框架：

自动扫描 Action 与 Result

支持嵌套属性与集合的自动参数注入

REST 风格 URL 路由

可通过配置文件设置拦截器

高度可扩展的开放 API

HTTP 客户端 (Jodd HTTP)

基于 Socket 的轻量级 HTTP 客户端：

支持 Cookie、文件上传、GZIP

可自定义请求头

支持 Basic 认证

增强型配置管理 (Props)

扩展了 Java Properties 格式：

原生支持 UTF-8 编码

支持变量插值与多行值

支持区段（类似 INI 文件）

邮件处理 (Jodd Email)

基于 javax.mail 的易用邮件客户端：

支持 SSL 与附件

支持 POP3/IMAP 接收

可解析 EML 文件

轻量级 IOC 容器 (Petite)

简化数据库操作 (Db & DbOom)

HTML/XML 解析 (Lagarto & Jerry)

Lagarto：高性能解析器

Jerry：类似 jQuery 的 HTML 操作 API

字段验证框架 (VTor)

基于注解的声明式验证工具，支持多套验证规则，易于扩展。

代理生成器 (Proxetta)

用于快速生成动态代理的高性能工具。

总结
Jodd 在极小的体积内，整合了类似 Apache Commons 多个组件的核心功能，并提供了自研的 MVC、IOC、ORM 等解决方案。其 HTTP
客户端调用流畅，HTML 操作方式类似 jQuery，日期处理可比肩 Joda-Time，堪称“麻雀虽小，五脏俱全”。

简而言之：Jodd = 工具集 + IOC + MVC + 数据库支持 + AOP + 事务 + JSON + HTML，全部封装在不足 1.7MB 的包中。

了解更多详情与使用示例，可访问其官方网站：https://jodd.org/