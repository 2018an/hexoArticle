---
title: Tomcat GET 请求特殊字符限制与解决方案
date: 2025-10-29 17:30:25
category: 后端
tags: Tomcat
---

![](img/18-2-27-1096232.jpg)

在使用 Tomcat 8.5 及更高版本时，如果 GET 请求中包含未经编码的特殊字符（如中文），服务器可能直接返回 400 错误，请求甚至无法到达应用层。

错误详情
Tomcat 日志中可能出现以下错误信息：

```text
java.lang.IllegalArgumentException: Invalid character found in the request target. The valid characters are defined in
RFC 7230 and RFC 3986
```
对应的 HTTP 响应为 400 Bad Request。

技术背景
此限制源于 Tomcat 对 RFC 7230 和 RFC 3986 规范的安全增强实现。从 Tomcat 7.0.73 版本开始，引入了对请求目标（Request
Target）的严格字符检查。

根据 RFC 3986 规范，URL 中仅允许包含以下字符：

英文字母（A-Z，a-z）

数字（0-9）

特殊字符：- _ . ~

保留字符：! * ' ( ) ; : @ & = + $ , / ? # [ ]

某些字符在 URL 中直接使用可能引起解析歧义，被视为"不安全字符"，包括：

空格（可能被无意添加或删除）

引号及尖括号（< > "）

井号（#，通常表示锚点）

百分号（%，本身是编码字符）

花括号、方括号、反引号等（{ } | \ ^ [ ] ` ~）

解决方案
针对此问题，有以下几种应对策略：

方案一：前端 URL 编码
在发起请求前，对 URL 中的特殊字符进行编码处理：

javascript
// JavaScript示例
let encodedParam = encodeURIComponent(parameter);
方案二：修改请求方法
将 GET 请求改为 POST 请求，参数放在请求体中传输，避开 URL 字符限制。

方案三：调整 Tomcat 配置
如果需要在 URL 中传输 JSON 等包含特殊字符的数据，可修改 catalina.properties 文件：

properties

# 允许花括号和竖线字符

tomcat.util.http.parser.HttpParser.requestTargetAllow=|{}
方案四：降级 Tomcat 版本
回退到 Tomcat 7.0.73 之前的版本（不推荐，存在安全风险）。

方案五：自定义 Tomcat 源码
修改 Tomcat 源码中的字符验证逻辑（仅适用于特殊情况，维护成本高）。

实践建议
主动预防：在可控的前端应用中，始终对 URL 参数进行编码

第三方对接：与合作方协商，要求其遵循 URL 编码规范

特殊需求：如必须在 URL 中传输特定字符，使用方案三配置 Tomcat

架构设计：考虑使用 POST 方法传输复杂数据，或设计 RESTful API 规范

通过合理选择解决方案，既能保证应用兼容性，又能遵循 HTTP 协议规范，确保系统的安全与稳定。