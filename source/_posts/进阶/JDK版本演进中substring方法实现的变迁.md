---
title: JDK版本演进中substring方法实现的变迁
date: 2025-10-15 11:36:33
category: 后端
tags: 进阶
---

String的substring(int beginIndex, int endIndex)方法在JDK 6与JDK 7/8中存在重要差异，主要体现在底层字符数组的共享行为上。

JDK 6的实现
在JDK 6中，String内部通过三个字段表示：char value[], int offset, int
count。substring创建新String时，共享原字符串的value数组，仅调整offset和count。

text
// JDK 6 源码示意
public String substring(int beginIndex, int endIndex) {
return new String(offset + beginIndex, endIndex - beginIndex, value);
}
https://www.programcreek.com/wp-content/uploads/2013/09/string-substring-jdk6-650x389.jpeg

潜在问题：若原字符串很大，而substring截取的部分很小，会导致新字符串仍持有整个大数组的引用，造成内存浪费。当时的一种解决方式是强制创建新数组：

text
x = x.substring(x, y) + "";
JDK 7/8的实现
从JDK 7开始，substring会复制所需范围的字符到新数组中，彻底与原字符串分离。

text
// JDK 7+ 源码示意
public String substring(int beginIndex, int endIndex) {
int subLen = endIndex - beginIndex;
return new String(value, beginIndex, subLen);
}
https://www.programcreek.com/wp-content/uploads/2013/09/string-substring-jdk71-650x389.jpeg

这一改动消除了内存泄漏风险，但增加了数组复制开销。对于现代应用而言，这种开销通常可接受。

了解这一历史差异，有助于在维护老系统或进行深度优化时做出正确判断。