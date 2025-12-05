---
title: 75 道 BATJ 中高级 Java 面试题目集锦
date: 2025-10-30 14:42:34
category: 程序人生
tags: 面试
---

整理了 BATJ（百度、阿里、腾讯、京东）等公司在 Java 中高级面试中常见的 75 道题目，供大家查漏补缺。若能掌握这些题目，能在面试中拉开与多数竞争者的差距。

75 道 BATJ 中高级 Java 面试题
hashCode 相等，两个对象一定相等吗？equals 呢？反之呢？

介绍一下 Java 集合框架的组成。

HashMap 与 Hashtable 底层实现的区别？Hashtable 与 ConcurrentHashMap 呢？

HashMap 与 TreeMap 的区别？底层数据结构是什么？

线程池用过吗？有哪些参数？底层如何实现？

synchronized 与 Lock 的区别？synchronized 何时是对象锁？何时是类锁？

ThreadLocal 是什么？底层如何实现？写一个示例。

volatile 的工作原理。

CAS 机制是什么？如何实现？

用至少四种方式写一个单例模式。

（以上为热身题，你能答对几道？）

介绍 JVM 内存模型？用过哪些垃圾回收器？

线上频繁 Full GC 如何处理？CPU 使用率过高怎么办？

如何定位线上问题？解决思路是什么？

了解字节码吗？Integer x = 5 与 int y = 5 比较时经历了哪些步骤？

类加载机制是什么？有哪些类加载器？分别加载哪些文件？

手写一个类加载示例。

了解 OSGi 吗？它是如何实现的？

你做过哪些 JVM 优化？使用了什么方法？达到了什么效果？

Class.forName("java.lang.String") 与 String.class.getClassLoader().loadClass("java.lang.String") 的区别？

Tomcat 的运行机制与框架结构。

Tomcat 的线程模型分析。

Tomcat 系统参数认识与调优。

MySQL 底层 B+Tree 机制。

SQL 执行计划详解。

索引优化详解。

SQL 语句如何优化？

Spring 的 AOP 与 IOC 机制，底层如何实现？

CGLib 与 JDK 动态代理的区别？手写一个 JDK 动态代理。

使用 MySQL 索引的原则？索引的数据结构？B+Tree 与 B-Tree 的区别？

MySQL 存储引擎有哪些？区别是什么？

高并发系统在数据库层面如何设计？数据库锁有哪些类型？如何实现？

数据库事务有哪些？

如何设计可动态扩缩容的分库分表方案？

用过哪些分库分表中间件？优缺点？底层实现原理？

从未分库分表系统迁移到分库分表系统，如何设计平滑切换方案？TCC 事务如何处理网络问题？

分布式事务了解吗？你们是如何解决的？

为什么要分库分表？

RPC 通信原理与分布式通信原理。

分布式寻址算法有哪些？一致性哈希了解吗？手写 Java 实现。如果按 userId 取模分片，如何查询连续时间段的数据？

分库分表后主键如何生成？有哪些方案？

Redis 与 Memcached 的区别？为什么单线程的 Redis 效率更高？

Redis 的数据类型及适用场景。

Redis 主从复制与集群模式的实现原理？Key 如何寻址？

如何用 Redis 或 ZooKeeper 实现分布式锁？哪种效率更高？

Redis 持久化机制？优缺点？底层实现？

Redis 过期策略有哪些？手写一个 LRU 的 Java 实现。

Dubbo 的实现过程？注册中心挂了还能通信吗？

Dubbo 支持的序列化协议？Hessian 的数据结构？PB 为什么效率高？

Netty 能做什么？NIO、BIO、AIO 的区别？

Dubbo 的负载均衡与高可用策略有哪些？动态代理策略呢？

为什么需要系统拆分？不用 Dubbo 可以吗？Dubbo 与 Thrift 的区别？

为什么使用消息队列？优缺点？

如何保证消息队列高可用？如何避免重复消费？

Kafka、ActiveMQ、RabbitMQ、RocketMQ 的优缺点？

如果让你设计一个消息队列，你会如何架构？

TCP/IP 四层模型是什么？

HTTP 的工作流程？HTTP/1.0、1.1、2.0 的区别？

TCP 三次握手与四次挥手流程图？为什么不是两次或五次？

HTTPS 的工作流程？如何防止被抓包？

源码中常用的设计思想与设计模式。

如何选择日志框架（log4j、log4j2、slf4j、jcl…）？

Spring AOP 原理，与 AspectJ 的关系，源码问题。

Dubbo 底层通信原理。

RPC 通信与分布式通信原理。

如何使用 Spring Cloud 构建微服务项目？

如何正确使用 Docker？

SpringMVC 底层原理及源码分析。

MyBatis 底层实现原理及源码分析。

MySQL 索引原理及实现方式。

索引底层算法、正确使用与优化。

Spring Boot 如何快速构建系统？

ZooKeeper 的原理与用途？Paxos 算法了解吗？

设计一个消息队列的架构思路。

分布式事务的解决方案。

你做过哪些 JVM 优化？方法与效果？

说真的，这些题目你能答出多少？

建议每天花时间学习几道，日积月累，在面试时自然能够从容应对。