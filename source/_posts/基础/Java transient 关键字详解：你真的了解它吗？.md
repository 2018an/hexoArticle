---
title: Java transient 关键字详解：你真的了解它吗？
date: 2025-10-14 14:42:34
category: 后端
tags: 基础
---


序列化是什么？
在实际开发中，我们常需要将对象的状态保存到文件或通过网络传输到另一台机器。Java
序列化机制正是将对象转换为字节序列的过程，以便存储或传输，并在需要时重新构造为原始对象。

transient 关键字的作用
用 transient 修饰的成员变量，在对象序列化时会被忽略，反序列化后其值为默认值（如 null、0 等）。

示例1：基本使用
java
class User implements Serializable {
private String name;
private transient String password;

    // 构造器、getter、setter 省略

}

// 测试
User user = new User("Alice", "secret");
// 序列化后，password 字段不会被保存
// 反序列化后，user.getPassword() 返回 null
静态变量能被序列化吗？
不能。静态变量属于类而非对象，序列化机制不会保存静态字段的值。例如：

java
class Config implements Serializable {
public static String version = "v1.0";
private transient String token;
}
即使 version 在序列化后发生变化，反序列化时仍读取当前 JVM 中的静态变量值。

使用 Externalizable 自定义序列化
如果实现 Externalizable 接口而非 Serializable，则可以完全控制序列化过程，此时 transient 关键字无效，因为序列化行为由
writeExternal 和 readExternal 方法决定。

java
class CustomUser implements Externalizable {
private String name;
private transient String id; // 可通过 writeExternal 手动序列化

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeObject(id); // 明确序列化
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        id = (String) in.readObject();
    }

}
使用建议
transient 只对实现 Serializable 的类有效。

常用于存储敏感信息或临时计算字段。

静态变量无论是否用 transient 修饰，都不会被序列化。

如需精细控制序列化行为，请使用 Externalizable。