---
title: Java 12 骚操作， String 类新增的几个“魔法”方法
date: 2025-10-15 11:36:33
category: 后端
tags: 新特性
---

Java 12 带来了不少令人惊喜的语法糖，今天我们就重点聊聊 String 类新增的几个“魔法”方法。它们看似小巧，却能在日常编码中极大提升效率与优雅度。

1. transform：链式转换的利器
   transform 方法允许你对字符串进行一系列连续的转换操作，其核心在于接收一个 Function 并返回转换后的结果。

来看看它的实际威力：

java
private static void demoTransform() {
System.out.println("====== 体验 Java 12 transform ======");
List<String> originalList = List.of("  Java  ", "  Python  ", "  Go  ");
List<String> processedList = new ArrayList<>();

    originalList.forEach(item ->
        processedList.add(
            item.transform(String::strip)      // 移除首尾空格
                .transform(String::toUpperCase) // 转为大写
                .transform(text -> "欢迎学习：" + text) // 添加前缀
        )
    );

    processedList.forEach(System.out::println);

}
运行结果：

```text
====== 体验 Java 12 transform ======
欢迎学习：JAVA
欢迎学习：PYTHON
欢迎学习：GO
```
通过链式调用，原本需要多行完成的清理、格式化与拼接操作，如今一气呵成。

2. indent：自动缩进文本块
   处理多行字符串时，整齐的缩进能让代码或输出更清晰。indent 方法就是为此而生。

java
private static void demoIndent() {
System.out.println("====== 体验 Java 12 indent ======");
String codeBlock = "public class Hello {\n public static void main(String[] args) {\n System.out.println(\"Hi\");\n
}\n}";
// 整体增加一级缩进（4个空格）
String indentedBlock = codeBlock.indent(4);
System.out.println(indentedBlock);
}
该方法会在每行行首添加指定数量的空格（若参数为负则移除空格），并确保末尾有换行符。对于生成格式化的文档、代码或日志输出格外有用。

3. describeConstable：将字符串包装为 Optional
   Java 12 中，String 实现了 Constable 接口，因此多了一个 describeConstable 方法。

java
private static void demoDescribeConstable() {
System.out.println("====== 体验 Java 12 describeConstable ======");
String blogName = "技术栈";
Optional<String> optionalName = blogName.describeConstable();
// 如果字符串不为空，这里就会正常输出
optionalName.ifPresent(System.out::println); // 输出：技术栈
}
虽然看起来只是简单返回 Optional.of(this)，但在一些需要函数式返回 Optional 的 API 或链式调用中，它能提供更好的类型连贯性和表达上的便利。

总结
transform、indent 和 describeConstable 这三个方法，分别从数据转换、格式控制和对象包装三个维度增强了 String 的能力。它们体现了
Java 语言正在朝着更简洁、更函数式的现代化风格演进。掌握这些新“玩具”，能让你的字符串处理代码更加精炼与富有表现力。

https://img/20190613135450.png
https://img/20190613135537.png

