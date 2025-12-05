---
title: Java 11 新特性解读：单命令完成编译与运行
date: 2025-10-30 14:42:34
category: 程序人生
tags: 行业动态
---

http://cc.cocimg.com/api/uploads//image/20171030/1509368302170729.jpg

Java 11 正式版即将发布，原计划于9月推出，目前仅剩不到三个月时间。此次更新将带来多项新功能，本文重点介绍其中一项名为 JEP 330
的特性。

简化流程：一键编译并执行源代码
回顾传统做法：

text
// 编译
javac Javastack.java

// 运行
java Javastack
通常我们需要先编译再运行，分为两个独立步骤。而在 Java 11 中，仅需一条指令即可完成：

text
java Javastack.java
尽管如此，这一改进在实际开发中的意义可能有限。大多数开发者依赖 IDE
完成编译与运行，真正使用命令行操作的场景并不多见。当然，如果你习惯使用文本编辑器编写代码，这个功能或许能带来一些便利。

那么，这是否意味着 javac 将逐渐被淘汰？并非如此。在某些场景下，我们仍需要独立的编译命令，而非直接执行源码。

支持 Shebang #! 符号执行 Java 程序
Shebang #! 是什么？它同样是 JEP 330 涉及的一项技术，允许在 UNIX 系统脚本中直接运行 Java 程序，示例如下：

text
#!/path/to/java --source version
JEP 330 特性小结
Oracle 提出的 JEP 330 主要面向小型 Java 应用的快速编译与执行，并非旨在将 Java 改造成通用脚本语言。该特性在评审阶段曾引发争议，但最终达成一致，确定将被纳入
Java 11 版本。

参考来源：https://securityonline.info/jdk-11-will-introduce-shebang-symbol/