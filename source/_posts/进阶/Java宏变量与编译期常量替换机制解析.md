---
title: Java宏变量与编译期常量替换机制解析
date: 2025-10-15 11:36:33
category: 后端
tags: 进阶
---

观察以下程序及其输出：

text
public static void main(String[] args) {
String hw = "hello world";
String hello = "hello";
final String finalWorld2 = "hello";
final String finalWorld3 = hello;
final String finalWorld4 = "he" + "llo";

    String hw1 = hello + " world";
    String hw2 = finalWorld2 + " world";
    String hw3 = finalWorld3 + " world";
    String hw4 = finalWorld4 + " world";
    
    System.out.println(hw == hw1); // false
    System.out.println(hw == hw2); // true
    System.out.println(hw == hw3); // false
    System.out.println(hw == hw4); // true

}
为何同样是字符串拼接，有的结果指向同一对象，有的却不同？关键在于宏变量概念。

什么是宏变量？
在Java中，被final修饰且初始值在编译期即可确定的变量，称为宏变量。编译器会将其在代码中的所有引用处直接替换为对应的常量值，这一过程称为宏替换。

例如：

text
final String a = "hello"; // 编译期可确定，是宏变量
final String b = a; // 编译期可确定，是宏变量
final String c = getHello(); // 运行期才能确定，不是宏变量
程序分析
finalWorld2 和 finalWorld4 的值在编译期即可确定为 "hello"，属于宏变量。拼接 " world" 后，编译器直接优化为 "hello world"
，与常量池中的 hw 指向同一对象。

hello 和 finalWorld3 虽然也是final，但其值在编译期无法完全确定（finalWorld3 引用了变量
hello），因此不会触发宏替换，拼接操作会在运行时生成新的字符串对象。

理解宏替换有助于编写更高效且符合预期的字符串逻辑。