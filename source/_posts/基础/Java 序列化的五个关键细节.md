---
title: Java 序列化的五个关键细节
date: 2025-10-14 14:42:34
category: 后端
tags: 基础
---


序列化不仅是将对象转为字节流，更涉及版本兼容、安全控制、性能优化等多个方面。

1. 支持类演变
   通过 serialVersionUID 控制版本兼容性。新增字段、将 static 改为非 static 等修改，反序列化时可自动处理（新增字段为默认值）。

java
public class Person implements Serializable {
private static final long serialVersionUID = 1L;
private String name;
// 后续版本可增加 private int age;
}

2. 可自定义序列化逻辑
   通过实现 writeObject 和 readObject 方法，可在序列化前后对数据进行加密或校验。

java
private void writeObject(ObjectOutputStream out) throws IOException {
// 加密敏感数据
out.defaultWriteObject();
}

private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
in.defaultReadObject();
// 解密数据
}

3. 支持签名与密封
   可使用 SignedObject 和 SealedObject 对序列化数据进行签名和加密，增强安全性。

4. 可替换序列化代理
   通过 writeReplace 和 readResolve 方法，可在序列化流中替换为代理对象，用于优化存储或兼容旧版本。

5. 提供验证机制
   实现 ObjectInputValidation 接口，可在反序列化后验证对象状态。

java
public class Validatable implements Serializable, ObjectInputValidation {
@Override
public void validateObject() throws InvalidObjectException {
// 验证逻辑
}
}