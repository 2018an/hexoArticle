---
title: Java Base64 编码方案对比
date: 2025-10-14 14:42:34
category: 后端
tags: 基础
---


Base64 是一种将二进制数据编码为 ASCII 字符串的方法，常用于数据传输或存储。Java 中实现 Base64 有多种方式。

1. 早期 JDK（不推荐）
   java
   import sun.misc.BASE64Encoder;
   import sun.misc.BASE64Decoder;

BASE64Encoder encoder = new BASE64Encoder();
BASE64Decoder decoder = new BASE64Decoder();
String encoded = encoder.encode(data);
byte[] decoded = decoder.decodeBuffer(encoded);
缺点：位于 sun.misc 包，非标准 API，性能较差，未来可能被移除。

2. Apache Commons Codec
   java
   import org.apache.commons.codec.binary.Base64;

Base64 base64 = new Base64();
String encoded = base64.encodeToString(data);
byte[] decoded = base64.decode(encoded);
优点：稳定可靠，性能较好。缺点：需引入外部依赖。

3. Java 8+ 标准 API（推荐）
   java
   import java.util.Base64;

Base64.Encoder encoder = Base64.getEncoder();
Base64.Decoder decoder = Base64.getDecoder();
String encoded = encoder.encodeToString(data);
byte[] decoded = decoder.decode(encoded);
优点：JDK 内置，性能最优（比 sun.misc 快 11 倍以上），无需额外依赖。

总结
新项目一律使用 java.util.Base64。

兼容旧系统时可考虑 Commons Codec。

避免使用 sun.misc 相关类。