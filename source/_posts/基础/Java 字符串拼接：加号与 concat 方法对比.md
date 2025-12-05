---
title: Java 字符串拼接：加号与 concat 方法对比
date: 2025-10-14 14:42:34
category: 后端
tags: 基础
---


在 Java 中，字符串拼接是日常开发中的常见操作。通常我们使用 + 运算符，但也可以使用 concat
方法。虽然二者最终都能实现字符串拼接，但它们在用法、特性及底层实现上有显著差异。

示例对比
我们先来看一段示例代码：

java
public static void main(String[] args) {
// 示例1
String s1 = "Hello";
System.out.println(s1 + 123); // Hello123
System.out.println(123 + s1); // 123Hello

    String s2 = "Java";
    s2 = s2.concat("World").concat("!");
    System.out.println(s2);              // JavaWorld!

    // 示例2
    String s3 = "Test";
    System.out.println(s3 + null);       // Testnull
    System.out.println(null + s3);       // nullTest

    String s4 = null;
    // System.out.println(s4.concat("X")); // NullPointerException
    // System.out.println("X".concat(s4)); // NullPointerException

}
concat 方法源码解析
java
public String concat(String str) {
int otherLen = str.length();
if (otherLen == 0) {
return this;
}
int len = value.length;
char buf[] = Arrays.copyOf(value, len + otherLen);
str.getChars(buf, len);
return new String(buf, true);
}
该方法创建了一个新的字符数组，并将原字符串和目标字符串的内容复制进去，最终返回一个新的字符串对象。

主要区别总结
参数类型灵活性

+ 可以拼接字符串与数字等基本类型，甚至允许 null；而 concat 仅接受字符串参数，且对 null 直接抛出 NullPointerException。

空值处理

+ 在遇到 null 时会将其转为 "null" 字符串处理；concat 则严格校验参数，禁止 null。

性能与编译行为
查看字节码可以发现，+ 在编译时被转换为 StringBuilder 的 append 操作。因此，多次 + 拼接会生成多个 StringBuilder 对象；而
concat 在拼接空字符串时略快，但在多字符串拼接场景下，仍建议使用 StringBuilder。

适用场景
简单拼接或含非字符串类型时可用 +；确定参数为字符串且非空时可用 concat；循环或复杂拼接应使用 StringBuilder。