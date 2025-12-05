---
title: 字符串拼接：+ 运算符与StringBuilder的适用边界
date: 2025-10-15 11:36:33
category: 后端
tags: 进阶
---

在Java开发中，我们常被告知字符串拼接应使用StringBuilder或StringBuffer，而非 + 运算符。然而，这一准则并非绝对，需视场景而定。

不适用 + 的场景
当字符串拼接分散在多个表达式时，每次 + 操作都会隐式创建新的StringBuilder对象，效率低下。

text
private void test1() {
String www = "www.";
String str = www;
str += "javastack.";
str += "com";
}
对应字节码中会出现两次 NEW java/lang/StringBuilder，在循环中频繁拼接时将严重影响性能。

适用 + 的场景
当所有拼接在同一表达式内完成，且均为字面量时，编译器会进行优化，直接合并为一个完整字符串。

text
private static void test2() {
String str = "www." + "javastack." + "com";
}
字节码中仅有一条 LDC "www.javastack.com" 指令，未创建StringBuilder。此优化同样适用于跨行的单表达式拼接：

text
String sql = "select name, sex, age, address"

+ " from t_user"
+ " where age > 18";
  核心原则
  循环或多表达式拼接：避免使用 +，应显式使用StringBuilder。

单表达式字面量拼接：可使用 +，编译器会优化。

理解这一区别，可在面试或实际编码中做出更合适的选择。