---
title: Java 序列化要点总结
date: 2025-10-14 14:42:34
category: 后端
tags: 基础
---


序列化是将对象状态转换为字节流的过程，用于持久化或网络传输。

如何实现序列化？
类必须实现 java.io.Serializable 接口（标记接口，无方法）。

java
public class User implements Serializable {
private static final long serialVersionUID = 1L;
private String name;
private transient String password; // 不序列化
// getters/setters
}
常用序列化工具
Apache Commons Lang 提供简便工具：

java
import org.apache.commons.lang3.SerializationUtils;

byte[] data = SerializationUtils.serialize(user);
User copy = SerializationUtils.deserialize(data);
关键规则
序列化的类必须实现 Serializable。

transient 字段不参与序列化。

serialVersionUID 用于版本控制，修改需谨慎。

静态字段不会被序列化。

父类若未序列化，其字段不会保存；若已序列化，子类自动支持。

序列化二进制格式仅 Java 可读，跨语言交互建议用 JSON/XML。

最佳实践
为重要类显式声明 serialVersionUID。

敏感信息标记 transient 或加密后序列化。

考虑使用 Externalizable 实现更精细的控制。