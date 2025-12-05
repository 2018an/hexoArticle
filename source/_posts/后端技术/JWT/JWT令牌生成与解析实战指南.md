---
title: JWT令牌生成与解析实战指南
date: 2025-10-29 17:30:25
category: 后端
tags: JWT
---

前文介绍了JWT的基本概念、应用场景、技术优势及使用要点，本文将通过具体实例演示JWT的实际应用。

根据JWT官网的类库支持情况，jjwt是Java环境中算法支持最全面的工具包，推荐使用。

> 项目地址：https://github.com/jwtk/jjwt

下面演示如何使用jjwt实现JWT令牌的生成与解析，重点展示SHA512算法的应用。

**1、添加jjwt依赖**

```xml

<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.0</version>
</dependency>
```

注意：JJWT需要Jackson 2.x版本支持，版本过低可能导致异常。

2、创建测试类JWTTest

3、生成签名密钥

采用SHA512算法需要预先定义密钥：

```java
Key KEY = new SecretKeySpec("javastack".getBytes(),
        SignatureAlgorithm.HS512.getJcaName());
```

此处生成固定密钥："javastack"

4、生成JWT令牌

核心实现代码：

```java
Map<String, Object> stringObjectMap = new HashMap<>();
stringObjectMap.

put("type","1");

String payload = "{\"user_id\":\"1341137\", \"expire_time\":\"2018-01-01 0:00:00\"}";
String compactJws = Jwts.builder().setHeader(stringObjectMap)
        .setPayload(payload).signWith(SignatureAlgorithm.HS512, KEY).compact();

System.out.

println("jwt key:"+new String(KEY.getEncoded()));
        System.out.

println("jwt payload:"+payload);
System.out.

println("jwt encoded:"+compactJws);
```

注意：头部参数可选设置，声明(claims)与载荷(payload)不可同时设置。

运行输出：

```text
jwt key:javastack
jwt payload:{"user_id":"1341137", "expire_time":"2018-01-01 0:00:00"}
jwt encoded:eyJ0eXBlIjoiMSIsImFsZyI6IkhTNTEyIn0.eyJ1c2VyX2lkIjoiMTM0MTEzNyIsICJleHBpcmVfdGltZSI6IjIwMTgtMDEtMDEgMDowMDowMCJ9.cnyXRnwczgNcNYqV6TUY2MaMfk6vujsZltC8Q51l40dwYJg516oZcV4VDKOypPT8fD7AE63PIhfdm2ALVrfv5A
```

5、解析JWT令牌内容

核心实现代码：

```java
Jws<Claims> claimsJws = Jwts.parser().setSigningKey(KEY).parseClaimsJws(compactJws);
JwsHeader header = claimsJws.getHeader();
Claims body = claimsJws.getBody();

System.out.

println("jwt header:"+header);
System.out.

println("jwt body:"+body);
System.out.

println("jwt body user-id:"+body.get("user_id", String .class));
```

运行输出：

```text
jwt header:{type=1, alg=HS512}
jwt body:{user_id=1341137, expire_time=2018-01-01 0:00:00}
jwt body user-id:1341137
```

将生成的密文通过JWT官网调试工具进行验证：

![](img/18-1-4-36539517.jpg)

验证成功。其他算法的应用逻辑与此类似，通过JWT可实现跨服务数据的安全传递与验证。