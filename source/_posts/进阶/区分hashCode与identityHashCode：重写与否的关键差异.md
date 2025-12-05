---
title: 区分hashCode与identityHashCode：重写与否的关键差异
date: 2025-10-15 11:36:33
category: 后端
tags: 进阶
---

identityHashCode 概述
System.identityHashCode(Object x) 是一个本地方法，其作用是返回对象的原始哈希码，无论该对象的类是否重写了hashCode()方法。

对比示例
text
public static void main(String[] args) {
String str1 = new String("abc");
String str2 = new String("abc");
System.out.println("str1 hashCode: " + str1.hashCode()); // 96354
System.out.println("str2 hashCode: " + str2.hashCode()); // 96354
System.out.println("str1 identityHashCode: " + System.identityHashCode(str1)); // 1173230247
System.out.println("str2 identityHashCode: " + System.identityHashCode(str2)); // 856419764

    User user = new User("test", 1);
    System.out.println("user hashCode: " + user.hashCode());           // 621009875
    System.out.println("user identityHashCode: " + System.identityHashCode(user)); // 621009875

}
结果分析
str1与str2的hashCode相同，因为String类重写了hashCode()，其计算基于字符串内容。

str1与str2的identityHashCode不同，因为该方法返回的是基于对象内存地址的原始哈希值，与内容无关。

User类未重写hashCode()，因此其hashCode与identityHashCode返回值一致。

结论
hashCode() 可被重写，常用于哈希集合中确定对象存储位置。

identityHashCode() 始终返回JVM赋予对象的原始哈希值，与对象内容无关，适用于需要区分对象实例的场景。

