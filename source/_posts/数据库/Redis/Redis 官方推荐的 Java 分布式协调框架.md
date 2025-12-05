---
title: Redis 官方推荐的 Java 分布式协调框架
date: 2025-10-30 14:42:34
category: 数据库
tags: Redis
---

什么是 Redisson？
Redisson 是 Redis 官方推荐的、用于 Java 的高级分布式协调客户端库。

它基于 Netty 这一高性能 NIO 框架构建，充分利用了 Redis 键值数据库的诸多优势，在 Java
标准工具包接口的基础上，提供了一系列具备分布式特性的常用工具类。这使得原本用于协调单机多线程并发的工具，获得了协调分布式多机多线程系统的能力，显著降低了设计和开发大规模分布式系统的复杂度。同时，结合其丰富的分布式服务，进一步简化了分布式环境中各组件间的协作。

Redisson 兼容 Redis 2.6+ 和 JDK 1.6+，采用 Apache License 2.0 授权协议。

官方网站：https://redisson.org/
GitHub 仓库：https://github.com/redisson/redisson

主要适用场景
分布式应用开发

分布式缓存实现

分布式会话管理

分布式任务/服务/延迟执行服务

作为功能丰富的 Redis 客户端

核心特性概览
Redisson 功能强大且全面，主要特性包括：

云服务与集群支持：支持 Amazon ElastiCache、Azure Redis Cache 等云托管服务，以及 Redis
Cluster、哨兵（Sentinel）、主从模式，支持自动发现节点与拓扑更新。

连接与线程安全：提供自行管理的弹性异步连接池，所有操作线程安全，支持异步操作与 LUA 脚本。

丰富的分布式对象：

通用对象桶（Object Bucket）、二进制流、地理空间对象桶、BitSet、原子长整型（AtomicLong）、原子双精度（AtomicDouble）。

发布/订阅（Topic）、布隆过滤器（Bloom Filter）、基数估计算法（HyperLogLog）。

完整的分布式集合：

映射（Map）、多值映射（Multimap）、集（Set）、列表（List）。

有序集（SortedSet）、计分排序集、字典排序集。

队列（Queue）、双端队列（Deque）、阻塞队列、有界阻塞队列、延迟队列、优先队列等。

分布式锁与同步器：

可重入锁（ReentrantLock）、公平锁、联锁（MultiLock）、红锁（RedLock）。

读写锁（ReadWriteLock）、信号量（Semaphore）、可过期信号量、闭锁（CountDownLatch）。

分布式服务：

远程服务（RPC）、实时对象（Live Object）服务、执行服务（Executor Service）。

调度任务服务（Scheduler Service）、映射归纳服务（MapReduce）。

强大的生态集成：支持 Spring 框架、Spring Cache、Hibernate Cache、JCache，提供 Tomcat Session Manager 和 Spring Session 集成。

灵活的序列化：支持 Jackson JSON、Avro、Smile、CBOR、MsgPack、Kryo、FST、LZ4、Snappy 及 JDK 序列化。

与 Jedis 的对比
Jedis 是 Redis 的 Java 客户端实现，其 API 基本覆盖了所有 Redis 命令，但功能相对基础。Redisson
则提供了更高级的抽象，专注于分布式数据结构和服务的实现，使得开发者能将精力更集中于业务逻辑，而非底层的连接和序列化细节。

快速开始
Maven 依赖

```xml
<!-- 适用于 JDK 1.8+ -->
<dependency>
   <groupId>org.redisson</groupId>
   <artifactId>redisson</artifactId>
   <version>3.5.5</version>
</dependency>

<!-- 适用于 JDK 1.6+ -->
<dependency>
   <groupId>org.redisson</groupId>
   <artifactId>redisson</artifactId>
   <version>2.10.5</version>
</dependency>
```
基础使用示例

```java
// 1. 创建配置对象
Config config = new Config();
config.useSingleServer().setAddress("redis://127.0.0.1:6379");

// 2. 创建 Redisson 客户端实例
RedissonClient redisson = Redisson.create(config);

// 3. 获取所需的对象或服务
RMap<String, Object> map = redisson.getMap("myMap");
RLock lock = redisson.getLock("myLock");
RExecutorService executor = redisson.getExecutorService("myExecutorService");
// ... 超过30种不同的分布式对象和服务可供使用
```