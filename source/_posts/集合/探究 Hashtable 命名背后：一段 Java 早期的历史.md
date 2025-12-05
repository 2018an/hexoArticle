---
title: 探究 Hashtable 命名背后：一段 Java 早期的历史
date: 2025-10-15 11:36:33
category: 后端
tags: 集合
---


在 Java 的命名规范中，类名通常采用驼峰式（CamelCase），即每个单词首字母大写，例如
HashMap、ArrayList、ConcurrentHashMap。但细心的开发者会发现，有一个类的名字显得与众不同：Hashtable，它的 ‘t’ 是小写的。

作为一个基础类，这似乎违背了 Java 基本的命名约定。为何 JDK 的代码中会存在这样一个看似“不规范”的命名呢？

出于好奇，我查阅了相关资料，虽然未找到官方的明确解释，但在 StackOverflow 上看到了一个被广泛认可的讨论。

原帖链接：
https://stackoverflow.com/questions/12506706/why-is-the-t-in-hash-tablehashtable-in-java-not-capitalized

http://qianniu.javastack.cn/18-12-6/56236009.jpg

其中被采纳的最佳答案指出：

Hashtable 诞生于 Java 1.0 版本。而如今我们熟知的、统一的集合类命名规范，是在后来的 Java 2 中，随着 Java 集合框架（Java
Collection Framework）的发布才正式确立的。这顺便也使得 Hashtable 在某种程度上过时了，因此在新代码中并不推荐继续使用。

通过查阅 JDK 源码可以证实，Hashtable 确实是 JDK 1.0 时期就存在的元老级集合类。这便能解释其命名上的“历史遗留问题”。那么，为何不在后续的
JDK 版本中修正这个命名呢？答案很可能是出于向后兼容性的考量。修改一个如此基础且被广泛使用的类名，可能会对大量遗留系统造成破坏。因此，这个“将错就错”的名字便被一直保留至今，即使到了
JDK 11 也未见更改或移除的计划。

此外，有评论提到可能存在一个名为 ConcurrentHashtable 的类。经过核实，无论是 currenthashtable 还是 concurrenthashtable，在
JDK 中均不存在。所有以 Concurrent 开头的并发工具类和接口，都在 java.util.concurrent 包下，如下图所示：

http://qianniu.javastack.cn/18-12-6/49796932.jpg

至此，关于 Hashtable 命名的疑惑便彻底解开了。