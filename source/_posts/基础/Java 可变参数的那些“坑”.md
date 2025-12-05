---
title: Java 可变参数的那些“坑”
date: 2025-10-14 14:42:34
category: 后端
tags: 基础
---


可变参数（Varargs）允许方法接受数量不定的参数，语法为 Type... args。虽然灵活，但使用不当容易引发问题。

示例分析
java
public static void main(String[] args) {
printFormatted("姓名=%s，备注=%s", "张三", "优秀");
// 输出：姓名=张三，备注=优秀
}

private static void printFormatted(String format, Object... args) {
String result = String.format(format, args);
System.out.println(result);
}
上述代码运行正常。但若调整参数传递方式：

java
private static void printFormatted(String format, Object... args) {
// 错误：将 args 数组作为单个参数传递
String result = String.format(format, args, "额外信息");
System.out.println(result);
}
此时输出可能变成：
姓名=[Ljava.lang.Object;@1b6d3586，备注=额外信息
因为 args 被当作一个数组对象传入，而非多个独立参数。

使用规范
根据《阿里巴巴Java开发手册》建议：

可变参数应放在参数列表末尾。

相同类型、相同业务含义的参数才使用可变参数。

避免使用 Object...，推荐明确类型。

尽量少用可变参数，以免在重载、重构时引起歧义。
