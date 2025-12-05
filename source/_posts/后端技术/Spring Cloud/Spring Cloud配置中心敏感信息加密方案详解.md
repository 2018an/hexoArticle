---
title: Spring Cloud配置中心敏感信息加密方案详解
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Cloud
---

## 配置信息的加密处理

从配置中心获取的配置项，默认情况下以明文形式存储和传输。对于数据库连接密码等敏感信息，为了提升安全性，我们需要对这些配置进行加密处理。下面介绍一种在配置中心启用加密功能的方法。

本文采用对称加密方案对配置进行加密。这种方式需要预先设置一个统一的密钥，虽然也可以选择RSA非对称加密，但对称加密在实现上更为简便，且足以满足大多数场景的安全需求。接下来，我们将以对称加密为例进行配置说明。

## 步骤一：安装Java加密扩展包（JCE）

标准JDK自带的JCE（Java Cryptography Extension）组件存在加密强度限制，为了使用无限制长度的密钥，需要替换为官方提供的无限制强度策略文件。

> 下载地址：http://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html

下载完成后，将压缩包内的两个JAR文件（例如 `local_policy.jar` 和 `US_export_policy.jar`）复制到 `JAVA_HOME/jre/lib/security`
目录下，覆盖原有文件。

## 步骤二：配置加密密钥

在配置中心服务端（例如 Spring Cloud Config Server）的配置文件中，添加用于对称加密的密钥。这个密钥将用于后续所有加密和解密操作。

```yaml
encrypt:
  key: 0e010e17-2529-4581-b907-c8edcfd6be09
```

## 步骤三：验证加密端点状态

启动配置中心服务后，可以通过访问其健康检查端点来确认加密功能是否已正常启用。

```
http://192.168.1.237:7100/encrypt/status
```

若服务返回以下JSON响应，则表明加密模块工作正常。

```json
{
  "status": "OK"
}
```

## 步骤四：执行加密与解密操作

配置中心提供了两个HTTP API端点，分别用于对字符串进行加密和解密。

**对明文“develop”进行加密：**

```bash
curl http://192.168.1.237:7100/encrypt -d develop -u config-user:99282424-5939-4b08-a40f-87b2cbc403f6
```

命令执行后，将返回一串经过加密的密文字符串。

**对上述密文进行解密，还原为原始明文：**

```bash
curl http://192.168.1.237:7100/decrypt -d 0fb593294187a31f35dea15e8bafaf77745328dcc20d6d6dd0dfa5ae753d6836 -u config-user:99282424-5939-4b08-a40f-87b2cbc403f6
```

命令中的 `-u` 参数用于指定HTTP Basic认证的用户名和密码。

## 步骤五：在配置文件中使用加密值

将加密后的字符串应用到具体的配置文件中。请注意，加密值需要以 `{cipher}` 作为前缀，并且整个值必须用**单引号**
括起来，否则配置中心在解析时会报错。

```yaml
spring:
  datasource:
    username: '{cipher}0fb593294187a31f35dea15e8bafaf77745328dcc20d6d6dd0dfa5ae753d6836'
```

## 步骤六：管理自动解密功能

默认情况下，配置中心客户端在获取到加密配置后会**自动进行解密**
。如果某些场景下，您希望客户端获取原始的密文，由应用层自行解密，同时仍需保留服务端的加解密端点，可以通过以下配置关闭客户端的自动解密功能。

```yaml
spring.cloud.config.server.encrypt.enabled: false
```

设置此属性后，客户端获取到的将是带有 `{cipher}` 前缀的原始密文，需要您在应用程序中调用相应的解密接口进行处理。而服务端的
`/encrypt` 和 `/decrypt` 端点依然可用。