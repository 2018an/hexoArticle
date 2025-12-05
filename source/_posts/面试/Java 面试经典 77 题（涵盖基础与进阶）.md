---
title: Java 面试经典 77 题（涵盖基础与进阶）
date: 2025-10-30 14:42:34
category: 程序人生
tags: 面试
---

“金三银四”求职季即将到来，你是否也在准备面试或已经历了几轮“车轮战”？

这里为大家整理了 77 道经典 Java 面试题，涵盖语言基础、集合、并发、JVM、数据库、Web 等方面。如果你的基础还不够扎实，或面试表现不佳，不妨从这些题目入手。

什么是 Java 虚拟机？为什么 Java 被称为“平台无关语言”？

JDK 与 JRE 的区别。

static 关键字的含义，能否覆盖 private 或 static 方法？

静态环境中能否访问非静态变量？

Java 支持哪些数据类型？什么是自动装箱与拆箱？

方法覆盖（Overriding）与重载（Overloading）的区别。

构造函数、构造函数重载、复制构造函数的概念。

Java 是否支持多继承？

接口与抽象类的区别。

值传递与引用传递的区别。

进程与线程的区别。

创建线程的几种方式，你偏好哪种？为什么？

线程的几种状态及其含义。

同步方法与同步代码块的区别。

什么是死锁？

如何避免 N 个线程访问 N 个资源时发生死锁？

集合框架的基本接口有哪些？

为什么集合类没有实现 Cloneable 与 Serializable？

什么是迭代器（Iterator）？

Iterator 与 ListIterator 的区别。

快速失败（fail-fast）与安全失败（fail-safe）的区别。

HashMap 的工作原理。

hashCode() 与 equals() 方法的重要性。

HashMap 与 Hashtable 的区别。

数组与 ArrayList 的区别，何时用数组？

ArrayList 与 LinkedList 的区别。

Comparable 与 Comparator 的作用与区别。

什么是优先级队列（PriorityQueue）？

了解大 O 表示法吗？能举例不同数据结构的时间复杂度吗？

如何选择使用有序数组还是无序数组？

集合类使用的最佳实践有哪些？

Enumeration 与 Iterator 的区别。

HashSet 与 TreeSet 的区别。

System.gc() 与 Runtime.gc() 的作用。

finalize() 方法何时调用？其目的是什么？

对象引用置为 null 后，垃圾回收器会立即释放内存吗？

Java 堆的结构是什么？永久代（PermGen）是什么？

串行收集器与吞吐量收集器的区别。

对象何时可以被垃圾回收？

永久代中会发生垃圾回收吗？

Java 中哪两种异常类型？区别是什么？

Exception 与 Error 的区别。

throw 与 throws 的区别。

异常处理完成后，Exception 对象会怎样？

finally 块与 finalize() 方法的区别。

什么是 JDBC？

JDBC 中驱动（Driver）的作用。

Class.forName() 方法的作用。

PreparedStatement 相比 Statement 的优势。

何时使用 CallableStatement？如何创建？

什么是数据库连接池？

什么是 RMI？

什么是分布式垃圾回收（DGC）？如何工作？

序列化与反序列化的概念。

什么是 Servlet？

Servlet 的体系结构。

GenericServlet 与 HttpServlet 的区别。

Servlet 的生命周期。

doGet() 与 doPost() 的区别。

什么是服务端包含（SSI）？

什么是 Servlet 链（Servlet Chaining）？

如何获取请求 Servlet 的客户端信息？

HTTP 响应的结构。

什么是 Cookie？Session 与 Cookie 的区别。

浏览器与 Servlet 之间使用什么协议通信？

什么是 HTTP 隧道？

sendRedirect() 与 forward() 的区别。

什么是 URL 编码与解码？

JSP 请求的处理过程。

什么是 JSP 指令？有哪些类型？

什么是 JSP 动作（Action）？

隐含对象是什么？有哪些？

面向对象开发的优点。

封装的定义与好处。

多态的定义。

继承的定义。

抽象的定义，抽象与封装的区别。