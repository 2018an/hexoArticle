---
title: Java 字符串创建方式的两道典型面试题解析
date: 2025-10-30 14:42:34
category: 程序人生
tags: 面试
---

在 Java 中，创建 String 类型的变量通常有以下两种方式：

java
String str1 = "abcd";
String str2 = new String("abcd");
为什么会有这两种方式？它们在内存中的表现有何不同？下面通过两道常见的面试题来解析。

面试题一：
java
String a = "abcd";
String b = "abcd";
System.out.println(a == b); // true
System.out.println(a.equals(b)); // true
解析：
使用双引号直接赋值的字符串，如果内容相同，JVM 会将其放入字符串常量池中，并让变量指向同一内存地址。因此 a == b 比较引用时返回
true，equals 比较内容也返回 true。

面试题二：
java
String c = new String("abcd");
String d = new String("abcd");
System.out.println(c == d); // false
System.out.println(c.equals(d)); // true
解析：
使用 new 创建的字符串对象会在堆内存中分别分配空间，即使内容相同，也是两个独立的对象。因此 c == d 返回 false，而 equals
比较内容仍返回 true。

下图直观展示了两种方式在内存中的区别：

https://www.programcreek.com/wp-content/uploads/2014/03/constructor-vs-double-quotes-Java-String-New-Page-650x324.png